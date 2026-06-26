# 04 — Database Design

This document explains the **data model**: what we store, how the pieces relate,
and the **best practices** we follow. The same design is expressed as runnable
schema in [`app/prisma/schema.prisma`](../app/prisma/schema.prisma). This doc is
the human-readable explanation; the schema file is the source of truth.

## Design goals

1. **Every fact is traceable.** A fact must always point back to where in the
   source text it came from. This is the backbone of "the AI never invents".
2. **Nothing is destroyed silently.** When facts conflict, we keep both and flag
   it. We use soft-deletes so a writer can recover from mistakes.
3. **Cross-referencing is first-class.** Links between entities are real rows we
   can query, not guesses made at display time.
4. **The writer owns their data.** The schema maps cleanly onto files we can
   write to the writer's cloud storage (doc 06).
5. **Privacy and security by structure.** Sensitive values (like OAuth tokens)
   are isolated and encrypted; everything ties back to an owning account.

## Best practices we follow (and why)

- **UUID primary keys** (not sequential integers). They don't leak how many
  records exist and are safe to expose in URLs.
- **`createdAt` / `updatedAt` timestamps on every table.** Cheap, and
  invaluable for debugging, sorting, and auditing.
- **Soft deletes** via a `deletedAt` timestamp instead of physically deleting
  rows, so writers can undo and we never lose provenance.
- **Foreign keys with explicit relations.** The database itself enforces that,
  e.g., a fact cannot exist without an entity. Data can't drift into nonsense.
- **Every user-owned row carries an `ownerId`.** This is the foundation of access
  control: a query for one writer's data can never accidentally return
  another's. (Enforced in app logic, and a candidate for Postgres Row-Level
  Security later — see doc 05.)
- **Enums for fixed sets** (entity types, job status, contradiction status) so
  invalid values can't be stored.
- **Indexes on the columns we filter by most** (owner, project, entity type) for
  speed as data grows.
- **Secrets never stored in plain text.** OAuth tokens are encrypted at rest in a
  dedicated table (doc 05).
- **Migrations, not hand-edits.** Every structural change is a reviewable,
  reversible migration managed by Prisma.

## The core tables (conceptual)

Below is the conceptual model. Field lists are representative, not exhaustive;
the authoritative list is in `schema.prisma`.

### `User`
A person with an account.
- `id`, `email`, `displayName`, `createdAt`, `updatedAt`, `deletedAt`
- Authentication details (managed largely by Auth.js).

### `StorageConnection`
A link to a writer's cloud storage (e.g. their Google Drive).
- `id`, `userId` → User
- `provider` (enum: `GOOGLE_DRIVE`, future: `DROPBOX`, `ONEDRIVE`)
- `encryptedAccessToken`, `encryptedRefreshToken`, `scopes`, `expiresAt`
- `rootFolderId` (where this app's files live in the writer's storage)
- Tokens are **encrypted at rest**. See doc 05.

### `Project`
One body of work — a novel, a campaign, a game's design bible.
- `id`, `userId` → User
- `title`, `kind` (enum: `NOVEL`, `SHORT_STORY`, `GAME_DESIGN`, `TTRPG_CAMPAIGN`,
  `OTHER`)
- `description`, timestamps, `deletedAt`

A user has many projects; a project belongs to one user. Everything below hangs
off a project, so different works keep separate knowledge bases.

### `SourceDocument`
A unit of imported text (a chapter, a session log, a pasted passage).
- `id`, `projectId` → Project
- `title`, `orderIndex` (so chapters keep their sequence)
- `cloudFileId` (where the original lives in the writer's storage)
- `contentHash` (to detect unchanged text and skip re-processing)
- timestamps, `deletedAt`

### `TextSpan`
A precise location **within** a source document — this is what makes facts
traceable and lets us "jump the writer to the exact spot".
- `id`, `sourceDocumentId` → SourceDocument
- `startOffset`, `endOffset` (character positions in the document)
- `excerpt` (the quoted text, for display)
- A `TextSpan` is referenced by facts and contradictions as their evidence.

### `Entity`
A tracked "thing": a character, place, item, creature, etc.
- `id`, `projectId` → Project
- `type` (enum: `CHARACTER`, `LOCATION`, `ITEM`, `CREATURE`, `ANIMAL`,
  `BUILDING`, `ENVIRONMENT`, `FACTION`, `EVENT`, `OTHER` — see doc 08)
- `canonicalName` (the primary name we display)
- `summary` (a short compiled overview — built only from sourced facts)
- timestamps, `deletedAt`

### `EntityAlias`
The other names an entity is known by, so "Will", "William", and "the captain"
all resolve to one entity.
- `id`, `entityId` → Entity
- `alias`, `confidence` (how sure we are this alias maps here)

### `Fact`
A single extracted statement about an entity — the atomic unit of the knowledge
base.
- `id`, `entityId` → Entity
- `attribute` (e.g. `eyeColour`, `occupation`, `speechStyle`) — see doc 08 for
  how attributes are organised per entity type.
- `value` (e.g. `"blue"`, `"blacksmith"`)
- `textSpanId` → TextSpan (**required** — every fact cites its source)
- `extractionJobId` → ExtractionJob (which run produced it)
- `status` (enum: `ACTIVE`, `SUPERSEDED`, `DISPUTED`)
- `confidence`, timestamps, `deletedAt`

When new text restates the same fact, we don't duplicate blindly; when it
*conflicts*, we create a `Contradiction` (below) rather than overwriting.

### `EntityLink`
A cross-reference between two entities (the hyperlink graph).
- `id`, `fromEntityId` → Entity, `toEntityId` → Entity
- `relationship` (e.g. `OWNS`, `LOCATED_IN`, `MEMBER_OF`, `RELATED_TO`,
  `MENTIONS`)
- `textSpanId` → TextSpan (where the relationship was observed)
- timestamps

This table is what powers "follow the hyperlink to read what you wrote before".

### `Contradiction`
A flagged conflict for the writer to resolve.
- `id`, `entityId` → Entity, `attribute`
- `factAId` → Fact, `factBId` → Fact (the two conflicting facts)
- `status` (enum: `OPEN`, `RESOLVED`, `DISMISSED`)
- `resolutionNote`, `resolvedByUserId`, `resolvedAt`
- timestamps

Each conflicting fact carries its own `TextSpan`, which is how we send the writer
to the exact spot in each source. See doc 09.

### `ExtractionJob`
One background run that turned source text into knowledge.
- `id`, `projectId` → Project, `sourceDocumentId` → SourceDocument
- `provider` (which AI service), `model`
- `status` (enum: `QUEUED`, `RUNNING`, `SUCCEEDED`, `FAILED`)
- `error`, `startedAt`, `finishedAt`, `tokenUsage`/cost fields
- timestamps

Keeping a record of every job gives us provenance ("which run created this
fact?"), debuggability, and a basis for usage/cost tracking.

### `AuditLog` (security & accountability)
An append-only record of significant actions (sign-in, storage connect/disconnect,
contradiction resolved, project deleted).
- `id`, `userId`, `action`, `metadata`, `ipAddress`, `createdAt`

## How the tables relate (text diagram)

```
User ─┬─< StorageConnection
      └─< Project ─┬─< SourceDocument ─< TextSpan
                   ├─< Entity ─┬─< EntityAlias
                   │           ├─< Fact >── TextSpan
                   │           └─< EntityLink >── Entity (the other end)
                   ├─< Contradiction >── Fact (x2), Entity
                   └─< ExtractionJob

(─< means "has many";  >── means "references")
```

## How this maps to the writer's cloud storage

The relational database is the **fast, queryable index**. The **permanent
memory** is the writer's cloud storage, where we also write a human-readable,
serialised copy of the knowledge base (e.g. one file per entity). If the app's
database were ever lost, the writer's own copy in their Drive remains. The exact
file format and sync rules are in doc 06.

## What we deliberately do NOT store

- We do not store AI provider API keys per row; there is a single server-side
  configuration for the app's own provider access, plus the writer's choice of
  provider/model (doc 07).
- We avoid storing more of the manuscript than needed in the app DB. The
  canonical copy stays in the writer's storage; the app keeps source text and
  spans needed for traceability and re-processing (revisit retention in doc 00).
