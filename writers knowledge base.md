# Writers Knowledge Base — Master Build Reference

> **What this file is.** This is the single, authoritative reference for building
> the website. It consolidates every decision, data structure, screen, security
> rule, and workflow needed to write the code. When building, follow THIS
> document; the numbered files in `docs/` are the longer-form reasoning behind
> each decision, and `app/prisma/schema.prisma` is the runnable form of the data
> model. If this document and another file ever disagree, **this document wins**
> and the other should be updated to match.
>
> **How it's organised (so it's easy to navigate while coding):** a table of
> contents, then self-contained sections. Each buildable feature has: purpose →
> behaviour → data it touches → API it needs → acceptance criteria.

**Working title:** AI Writing Assistant (no official public name yet — see §16).
**Status:** internal development.
**Last updated:** 2026-06-26.

---

## Table of contents

1. [Product in one page](#1-product-in-one-page)
2. [Core concepts & vocabulary](#2-core-concepts--vocabulary)
3. [Technology stack (locked)](#3-technology-stack-locked)
4. [Decisions already made](#4-decisions-already-made)
5. [The screens (page-by-page build spec)](#5-the-screens-page-by-page-build-spec)
6. [Data model (complete)](#6-data-model-complete)
7. [Security architecture (the backbone)](#7-security-architecture-the-backbone)
8. [Backups & disaster recovery](#8-backups--disaster-recovery)
9. [Export & sync (user owns their data)](#9-export--sync-user-owns-their-data)
10. [AI access plan (the core)](#10-ai-access-plan-the-core)
11. [API surface](#11-api-surface)
12. [Background jobs](#12-background-jobs)
13. [Coding standards](#13-coding-standards)
14. [Configuration & environment](#14-configuration--environment)
15. [Build order (phases → features)](#15-build-order-phases--features)
16. [Still open](#16-still-open)

---

## 1. Product in one page

A website where a writer signs in, creates **projects** (a novel, a game design
bible, a TTRPG campaign…), organises their writing into **categories**, and
writes **pages** inside those categories. A connected **AI** reads the pages and
automatically builds a cross-referenced, hyperlinked **knowledge base** of
**assist files** — one per character, place, item, creature, faction, etc. —
compiling every detail the text supports, flagging contradictions for the writer
to resolve, and keeping everything linked so the writer never has to trawl back
through chapters.

**The one rule that defines the product:**

> **The AI compiles and cross-references. It never writes or edits the writer's
> pages.** It only *reads* the writer's content and *creates new* assist files.
> Every fact it records traces back to an exact spot in a page. The AI is a
> librarian, never a co-author.

**Who it's for:** solo fiction writers and worldbuilders who need consistency
across long or multi-part work (novelists, series writers, game designers, TTRPG
game masters, hobbyist worldbuilders).

---

## 2. Core concepts & vocabulary

| Term | Meaning | Backed by table |
|------|---------|-----------------|
| **User / writer** | A person with an account. | `User` |
| **Project** | One body of work. Everything else lives under a project. | `Project` |
| **Category** | A named bucket that organises pages within a project (e.g. "Chapters", "Lore", "Session Notes"). May nest. | `Category` |
| **Page** | A document the writer writes in. **This is the source content.** Lives in a category. | `Page`, `PageVersion` |
| **Text span** | An exact character range within a page; makes facts traceable. | `TextSpan` |
| **Assist file / Entity** | An AI-generated reference file about one thing (a character, place, item…). The AI *creates* these; the writer never has to. | `Entity`, `AssistFile` |
| **Fact** | A single sourced statement about an entity (attribute + value + where it came from + when in the story it's true). | `Fact` |
| **Link / cross-reference** | A typed relationship between two entities → becomes a hyperlink. | `EntityLink` |
| **Contradiction** | Two facts that can't both be true at the same story-time; flagged for the writer. | `Contradiction` |
| **Provider** | An AI service (Anthropic Claude default; pluggable). | `ProviderCredential` |
| **Storage connection** | The writer's own cloud storage for export/sync (Google Drive first). | `StorageConnection` |

**Key relationship:** `Project → Category → Page → (AI reads) → Entity/AssistFile
+ Fact + EntityLink + Contradiction`. Pages are written by the human; entities
are created by the AI from those pages.

---

## 3. Technology stack (locked)

| Layer | Choice | Notes |
|-------|--------|-------|
| Language | **TypeScript** | One language, frontend + backend. |
| Framework | **Next.js (React, App Router)** | Pages + API in one project. |
| Styling | **Tailwind CSS** | |
| Database | **PostgreSQL** | Relational; fits cross-referenced data. |
| DB access | **Prisma (ORM)** | Schema in `app/prisma/schema.prisma`; migrations only. |
| Auth | **Auth.js (NextAuth)** | Google OAuth + optional email/password (§7). |
| Jobs | **BullMQ + Redis** | Background AI extraction. |
| AI (default) | **Anthropic Claude (latest model)** | Pluggable provider layer. Writers bring their own key. |
| Cloud storage | **Google Drive API** (app-folder scope) | First export/sync target. |
| Hosting | **Managed cloud** | Platform TBD; build cloud-agnostic. |
| Secrets/keys | **Cloud KMS + env vars** | No secrets in the repo. |

Rich text editing for pages: a well-supported editor (e.g. **TipTap**/ProseMirror
or Lexical). Store page content as portable structured text (Markdown or
ProseMirror JSON) — see §6 `Page.content`.

---

## 4. Decisions already made

From `docs/00-open-questions.md` (kept here for building convenience):

- **AI cost:** writers **bring their own API key** (encrypted at rest).
- **Drive scope:** **app-folder only** (app can't see the rest of the writer's Drive).
- **First build:** core loop first (auth → projects → categories → pages →
  extraction → browse assist files); Drive sync comes after.
- **Hosting:** managed cloud.
- **Change over time:** **track a timeline** — facts carry "true as of page/chapter
  N"; legitimate change becomes `HISTORICAL`, not a contradiction.
- **Entity merging:** **ask the writer when unsure**; auto-merge only when confident.
- **Contradiction sensitivity:** **per-project, adjustable on the fly** via a slider.
- **Entity priority order:** characters → places → environments → factions →
  equipment → items (others still tracked).
- **Comment standard:** heavily commented, plain-English code (§13).

---

## 5. The screens (page-by-page build spec)

Each screen lists: **purpose**, **key elements**, **behaviour**, and **done when**.

### 5.1 Front page (marketing / welcome) — public

- **Purpose:** explain what the site is and its abilities; convert a visitor into
  a sign-up.
- **Key elements:**
  - Hero: one-line pitch ("Your story bible, written for you.") + sub-line + a
    primary "Get started / Sign in" button.
  - "What it does" section, describing the abilities in plain language:
    - Write your novel/campaign in organised pages and categories.
    - The AI reads your pages and builds a reference file for every character,
      place, item, creature, faction, and event — automatically.
    - Everything is cross-referenced and hyperlinked, so you never lose a detail.
    - It flags contradictions (e.g. eye colour changing) and takes you to the
      exact spot to fix them.
    - It tracks how things change over time (a character moving, dying, etc.).
    - **It never writes or edits your story — it only organises what you wrote.**
    - Your work is yours: strong security, regular backups, and export/sync to
      your own cloud or machine.
  - "How it works" 3-step strip: Write → AI compiles → Browse & resolve.
  - Trust/security callout (see §7 highlights: encryption, backups, you own your data).
  - Footer: privacy policy, terms, contact. (Privacy policy required before any
    non-internal launch — §16.)
- **Behaviour:** fully static/public; no user data. Fast, accessible, responsive.
- **Done when:** a visitor can understand the product and reach sign-in in one click.

### 5.2 Sign in / Sign up — public

- **Purpose:** authenticate the writer securely.
- **Key elements:** "Continue with Google" (primary). Optional email + password
  (if enabled) with: password strength meter, and **2FA/MFA** enrolment prompt
  (§7). "Forgot password" flow (email-based, if email/password enabled).
- **Behaviour:** delegates to Auth.js. On first sign-in, create `User`, then send
  to onboarding (§5.3). Sessions are secure, http-only cookies (§7).
- **Done when:** a new user can create an account and land in their dashboard; an
  existing user can sign back in; brute-force and enumeration are mitigated (§7).

### 5.3 Onboarding (first run) — authenticated

- **Purpose:** get the writer to a usable state.
- **Steps:** (1) welcome + the one rule; (2) **add your AI key** — guided,
  friendly flow to paste an Anthropic/OpenAI key, with a "where do I get this?"
  helper and a live "test key" check (§10, §16 Q16); (3) create your first project.
- **Behaviour:** the AI key is validated then stored encrypted (`ProviderCredential`).
  Skippable steps can be resumed later from Settings.
- **Done when:** the writer has a project and (ideally) a working AI key.

### 5.4 Dashboard / Projects — authenticated

- **Purpose:** the home base; list and create projects.
- **Key elements:** project cards (title, kind, last activity, open-contradiction
  count); "New project" (title + kind: novel / short story / game design / TTRPG
  campaign / other); account menu.
- **Behaviour:** every query scoped to the signed-in user (§7). Soft-deleted
  projects hidden.
- **Done when:** the writer can create, open, rename, and (soft) delete projects.

### 5.5 Project workspace — authenticated

The main working area for one project. Three-region layout:

- **Left — navigator:**
  - Two switchable trees:
    - **Content:** categories → pages (the writer's writing).
    - **Knowledge base:** entity types in priority order (Characters, Places,
      Environments, Factions, Equipment, Items, then others) → assist files.
  - Actions: new category, new page, search across pages and entities.
- **Centre — main view:** whatever is open (a page editor, an entity/assist file,
  the review queue, or the source viewer).
- **Top bar:** project title, "Add/Write" action, a **review badge** (open
  contradiction count), and quick access to project settings (incl. the
  sensitivity slider).
- **Done when:** the writer can navigate content and knowledge base, and open any
  item into the centre view.

### 5.6 Category management — authenticated

- **Purpose:** organise pages.
- **Key elements:** create/rename/delete categories; optional **sub-categories**
  (nesting); drag to reorder; move pages between categories.
- **Behaviour:** deleting a non-empty category asks what to do with its pages
  (move to another category or soft-delete). Order preserved via `orderIndex`.
- **Done when:** the writer can fully organise pages into a category structure.

### 5.7 Page editor — authenticated (the writing surface)

- **Purpose:** where the writer actually writes. **This is source content.**
- **Key elements:** rich-text editor (headings, bold/italic, lists, links),
  title, category selector, autosave indicator, word count, version history
  access, and a "Compile with AI" / "Re-scan" action for this page (or auto on save).
- **Behaviour:**
  - **Autosave** with debounce; each meaningful save creates/updates a
    `PageVersion` (history, and a safety net).
  - Page content is stored **encrypted at rest** (§7).
  - Saving (or an explicit "Compile") **enqueues an extraction job** (§10, §12).
  - The AI **never** modifies the page text. Any AI output appears as separate
    assist files, never inline edits.
- **Done when:** the writer can write, autosave, see version history, and trigger
  AI compilation; content is encrypted; extraction runs in the background.

### 5.8 Assist file / Entity page — authenticated

- **Purpose:** the AI-built reference page for one entity.
- **Key elements:**
  - Name + aliases + type badge; short compiled summary (from sourced facts only).
  - Facts grouped by category (Appearance, Personality, Speech, Relationships…),
    **each showing its source page + quote**, click to jump to the exact span.
  - **Timeline/history** for time-varying details (current value + how it changed).
  - Related entities as **hyperlinks** (the cross-reference graph).
  - Disputed facts marked, linking into the review queue.
  - "Appears in" list of pages.
  - Clear "AI-generated" marking; a control to correct/split/merge an entity, and
    to confirm uncertain alias/merge suggestions.
- **Behaviour:** read-first; writer edits are limited to organising/correcting the
  knowledge base (not the story). Regenerated as pages change.
- **Done when:** entities render with sourced, timeline-aware facts and working
  hyperlinks; jump-to-source works.

### 5.9 Source viewer — authenticated

- **Purpose:** read a page with the ability to land on an exact span.
- **Behaviour:** deep-links to `startOffset`/`endOffset` highlight the passage —
  powers "Go to this spot" from facts and contradictions.
- **Done when:** any span link opens the right page scrolled to and highlighting it.

### 5.10 Review queue (contradictions) — authenticated

- **Purpose:** resolve conflicts.
- **Key elements:** list of `OPEN` contradictions; each shows both versions with
  sources + "Go to this spot"; resolution choices (keep A / keep B / both valid
  over time / edit the source / dismiss); a history view; the **sensitivity
  slider** (Relaxed ↔ Strict, per project, live).
- **Behaviour:** resolutions update fact statuses and record who/when/why; resolved
  items aren't re-flagged unless the source text changes. Timeline-aware (only
  same-time conflicts appear).
- **Done when:** the writer can work through and resolve conflicts, adjust
  sensitivity live, and see history.

### 5.11 Export & sync — authenticated

- **Purpose:** the writer owns their data (§9).
- **Key elements:** "Download everything" (zip); "Connect Google Drive" for
  ongoing sync (app-folder scope); sync status; disconnect.
- **Done when:** the writer can download a complete, readable copy and/or keep a
  synced copy in their own Drive.

### 5.12 Settings — authenticated

- **Purpose:** account, AI key, storage, privacy, security.
- **Key elements:** AI provider/model + key management (validate/replace/remove);
  storage connections; **security** (2FA/MFA, active sessions, sign-out
  everywhere); account (display name, email); **data** (export, delete account);
  privacy info (what's sent to which AI provider).
- **Done when:** the writer can manage all of the above safely.

---

## 6. Data model (complete)

This supersedes the earlier draft in `app/prisma/schema.prisma` by adding
**Category**, **Page/PageVersion**, **AssistFile**, and **ApiToken**. The Prisma
file must be brought into line with this section (tracked in §15). Conventions:
UUID primary keys, `createdAt`/`updatedAt` on every table, **soft delete**
(`deletedAt`) on user-facing rows, every user-owned row traceable to an owner,
enums for fixed sets, indexes on hot columns, secrets/content encrypted at rest.

### 6.1 Enums

- `ProjectKind`: NOVEL, SHORT_STORY, GAME_DESIGN, TTRPG_CAMPAIGN, OTHER
- `EntityType`: CHARACTER, LOCATION, BUILDING, ENVIRONMENT, ITEM, EQUIPMENT,
  CREATURE, ANIMAL, FACTION, EVENT, OTHER
- `FactStatus`: ACTIVE, HISTORICAL, SUPERSEDED, DISPUTED
- `ContradictionStatus`: OPEN, RESOLVED, DISMISSED
- `JobStatus`: QUEUED, RUNNING, SUCCEEDED, FAILED
- `LinkType`: OWNS, LOCATED_IN, MEMBER_OF, RELATED_TO, ALLY_OF, ENEMY_OF,
  PARTICIPATED_IN, MENTIONS
- `StorageProvider`: GOOGLE_DRIVE, DROPBOX, ONEDRIVE
- `AiProvider`: ANTHROPIC, OPENAI
- `AssistFileType`: ENTITY, INDEX, SUMMARY
- `ApiTokenScope`: READ_CONTENT, READ_KNOWLEDGE, WRITE_ASSIST

### 6.2 Tables

**User** — a person. `id`, `email` (unique), `displayName?`, `emailVerifiedAt?`,
`mfaEnabled` (bool), timestamps, `deletedAt`.
Relations: authCredentials, sessions, providerCredentials, storageConnections,
projects, apiTokens, auditLogs.

**AuthCredential** *(only if email/password enabled)* — `id`, `userId`,
`passwordHash` (**Argon2id**), `passwordUpdatedAt`, timestamps. Never store raw
passwords. MFA secrets stored encrypted (separate column/table).

**Session** — managed by Auth.js (secure, http-only cookies). Track device/IP for
"active sessions" + "sign out everywhere".

**ProviderCredential** — writer's own AI key. `id`, `userId`, `provider`
(`AiProvider`), `encryptedApiKey`, `preferredModel?`, `lastValidatedAt?`,
timestamps, `deletedAt`. Unique `(userId, provider)`. **Encrypted at rest** (§7).

**StorageConnection** — writer's cloud storage. `id`, `userId`, `provider`
(`StorageProvider`), `encryptedAccessToken`, `encryptedRefreshToken?`, `scopes`,
`expiresAt?`, `rootFolderId?`, timestamps, `deletedAt`. **Encrypted at rest.**

**Project** — `id`, `userId`, `title`, `kind` (`ProjectKind`), `description?`,
`contradictionSensitivity` (Int 0–100, default 50), timestamps, `deletedAt`.
Index `(userId)`.

**Category** — organises pages. `id`, `projectId`, `parentCategoryId?` (nesting),
`name`, `orderIndex` (Int), timestamps, `deletedAt`. Index `(projectId,
parentCategoryId)`.

**Page** — the writer's writing (source content). `id`, `projectId`, `categoryId?`,
`title`, `orderIndex` (Int), `content` (**encrypted** structured text — Markdown
or ProseMirror JSON), `contentHash` (to skip re-processing unchanged pages),
`wordCount?`, timestamps, `deletedAt`. Indexes `(projectId)`, `(categoryId)`.

**PageVersion** — history/safety net. `id`, `pageId`, `content` (encrypted),
`contentHash`, `createdAt`, `createdByUserId`. Index `(pageId, createdAt)`.

**TextSpan** — exact location in a page. `id`, `pageId`, `startOffset`,
`endOffset`, `excerpt`, `createdAt`. Index `(pageId)`. Cited by facts + links.

**Entity** — an AI-built "thing". `id`, `projectId`, `type` (`EntityType`),
`canonicalName`, `summary?`, `priorityRank?` (from type order), timestamps,
`deletedAt`. Index `(projectId, type)`.

**EntityAlias** — `id`, `entityId`, `alias`, `confidence?`, `confirmedByUser`
(bool), `createdAt`. Index `(entityId)`.

**Fact** — atomic sourced statement. `id`, `entityId`, `attribute`, `value`,
`textSpanId` (**required** — traceability), `extractionJobId?`, `status`
(`FactStatus`, default ACTIVE), `confidence?`, `timelineOrder?` (story-position;
defaults from the page's order — enables "true as of chapter N"), timestamps,
`deletedAt`. Index `(entityId, attribute)`.

**EntityLink** — the hyperlink graph. `id`, `fromEntityId`, `toEntityId`,
`relationship` (`LinkType`), `textSpanId?`, `createdAt`. Indexes on both entity
ids.

**Contradiction** — flagged conflict. `id`, `projectId`, `entityId`, `attribute`,
`factAId`, `factBId`, `status` (`ContradictionStatus`, default OPEN),
`resolutionNote?`, `resolvedByUserId?`, `resolvedAt?`, timestamps. Indexes
`(projectId, status)`, `(entityId)`.

**AssistFile** — a **new file the AI creates** to assist the writer (the
exportable/syncable reference document). `id`, `projectId`, `type`
(`AssistFileType`), `entityId?` (for ENTITY type), `title`, `renderedContent`
(Markdown, generated from facts/links — safe to export), `cloudFileId?` (where
it's synced in the writer's storage), `sourceHash` (what it was generated from),
timestamps, `deletedAt`. Index `(projectId, type)`. **These are always AI-created
and clearly marked; they never overwrite pages.**

**ExtractionJob** — one background run. `id`, `projectId`, `pageId?`, `provider`,
`model`, `status` (`JobStatus`), `error?`, `tokenUsage?`, `startedAt?`,
`finishedAt?`, timestamps. Relation: facts produced. Index `(projectId, status)`.

**ApiToken** — for programmatic AI access (§10.4). `id`, `userId`, `name`,
`tokenHash` (store only a hash), `scopes` (`ApiTokenScope[]`), `projectId?` (can
scope to one project), `lastUsedAt?`, `expiresAt?`, `revokedAt?`, timestamps.
Index `(userId)`.

**AuditLog** — append-only. `id`, `userId?`, `action`, `metadata?` (small JSON, no
secrets/content), `ipAddress?`, `createdAt`. Index `(userId)`.

### 6.3 Relationship diagram (text)

```
User ─┬─ AuthCredential / Session / ProviderCredential / StorageConnection / ApiToken / AuditLog
      └─< Project ─┬─< Category (self-nesting) ─< Page ─┬─< PageVersion
                   │                                    └─< TextSpan ─< Fact / EntityLink
                   ├─< Entity ─┬─< EntityAlias
                   │           ├─< Fact >── TextSpan
                   │           └─< EntityLink >── Entity
                   ├─< AssistFile (>── Entity, optional)
                   ├─< Contradiction >── Fact ×2, Entity
                   └─< ExtractionJob ─< Fact
```

---

## 7. Security architecture (the backbone)

Security is a first-class feature. A writer's unpublished work is highly
sensitive. We aim **beyond baseline industry standard** using **defence in
depth** — multiple independent layers, so no single failure is catastrophic.

### 7.1 Principles

- **Least privilege** everywhere (DB roles, OAuth scopes, API tokens, service
  accounts).
- **Encrypt in transit and at rest**, including **field-level encryption** of the
  most sensitive data (page content, AI keys, storage tokens).
- **Assume breach:** design so that a leak of one store (e.g. the database) does
  not by itself expose secrets or readable content.
- **The writer is in control:** they can see sessions, revoke access, export, and
  delete.

### 7.2 Authentication

- Primary: **Google OAuth** via Auth.js (no password to store).
- Optional email/password: hashed with **Argon2id** (memory-hard), never
  reversible; strong-password policy + breached-password check.
- **MFA/2FA** (TOTP authenticator apps; optional WebAuthn/passkeys later) — offer
  to all accounts, strongly encourage.
- **Session security:** secure, http-only, `SameSite` cookies; short-lived with
  rotation; "active sessions" list and "sign out everywhere".
- **Anti-abuse:** rate limiting + progressive backoff on sign-in; generic errors
  to prevent account enumeration; lockout/alert on repeated failures; CAPTCHA
  only if abuse appears.

### 7.3 Authorisation

- Every user-owned row is scoped to its owner in application code, **and** backed
  by **PostgreSQL Row-Level Security (RLS)** as an independent second layer, so a
  bug in app code still cannot cross accounts.
- API tokens (§10.4) carry explicit, minimal **scopes** and can be project-scoped
  and revoked.

### 7.4 Encryption

- **In transit:** TLS 1.2+ (prefer 1.3) everywhere; HSTS.
- **At rest (whole database):** provider-level encrypted storage/volumes.
- **Field-level (envelope) encryption** for the crown jewels, on top of DB
  encryption:
  - **Page content** and **PageVersion content** (the writer's actual words).
  - **AI provider keys** (`ProviderCredential.encryptedApiKey`).
  - **Storage tokens** (`StorageConnection.encrypted*`).
  - **MFA secrets.**
  - Scheme: a per-user (or per-record) **data key** encrypts the field; the data
    key is itself wrapped by a **master key held in a cloud KMS** (never in the DB
    or code). Algorithm: **AES-256-GCM**. Decryption happens only server-side, in
    memory, at the moment of use.
- **Key management:** master keys in KMS with rotation; wrapped data keys stored
  alongside records; `TOKEN_ENCRYPTION_KEY`/KMS references via env only.

### 7.5 Application hardening

- **Validate all input** at every boundary (e.g. Zod schemas) — never trust the
  browser or any external caller.
- **Output encoding**/safe rendering to prevent XSS; never render user text as raw
  HTML unsafely; strict **Content-Security-Policy**; security headers
  (X-Content-Type-Options, Referrer-Policy, etc.).
- **CSRF** protection (framework + SameSite cookies).
- **Rate limiting** on auth, extraction, export, and API endpoints (also protects
  against runaway AI cost/abuse).
- **Dependency hygiene:** automated vulnerability scanning; keep libraries
  patched; pin versions; verify integrity.
- **Secrets never in the repo**; only `app/.env.example` (names only) is committed.
- **Logging without secrets/content:** never log page bodies, keys, or tokens.

### 7.6 AI-specific security

- **Untrusted input:** page content may contain hidden instructions
  ("prompt injection"). Always send writer text as **data to analyse, never as
  instructions**; keep the extraction instructions in a separate, privileged
  channel; **strictly validate** the AI's structured output and reject anything
  malformed.
- **Small blast radius by design:** the AI can only **read** pages and **create**
  assist files; it can never edit pages or take destructive actions, so a
  successful injection can't corrupt the writer's work.
- **Transparency:** the writer is told which provider their text is sent to (and
  supplies that provider's key themselves).

### 7.7 Operational security

- Separate environments (dev/staging/prod) with separate secrets.
- Principle-of-least-privilege service accounts and DB roles.
- **Audit logging** of significant actions (sign-in, key/storage connect &
  disconnect, deletions, contradiction resolutions, API token use).
- Regular security review of the diff (there is a `security-review` workflow).
- Plan for periodic dependency and (eventually) penetration testing before public
  launch.

### 7.8 Privacy & data lifecycle

- Collect the minimum personal data needed.
- **Delete account** removes the app's copies of content, keys, and tokens
  (subject to a short, documented backup-retention window — §8, §16 Q13).
- Publish a plain-English privacy policy before any non-internal use.

---

## 8. Backups & disaster recovery

The system must be **designed to be backed up regularly**, and restores must be
**tested**, so that if something happens the backups can be pulled.

### 8.1 What we back up

- **The database** (all structured data: users, projects, categories, pages,
  page versions, entities, facts, links, jobs, audit log).
- **Encryption keys** (KMS-managed, backed up per the KMS provider's guarantees —
  without keys, encrypted backups are useless).
- Config/infrastructure definitions (as code, in version control).

### 8.2 How

- **Automated, scheduled backups** of PostgreSQL:
  - **Continuous point-in-time recovery (PITR)** via write-ahead logs where the
    managed database supports it, plus
  - **Daily full snapshots**.
- **Encrypted** backups (at rest and in transit).
- **Off-site / separate-region** copies (don't keep backups only next to prod).
- **Retention policy:** e.g. daily for 30 days, weekly for a few months (exact
  numbers = §16 Q13); documented and enforced.
- **Restore testing:** periodically restore into a scratch environment and verify
  integrity — a backup that has never been restored is not a backup.

### 8.3 Recovery objectives (targets, to confirm)

- **RPO** (max acceptable data loss): small (minutes, via PITR).
- **RTO** (max acceptable downtime to restore): a few hours.
- A written **runbook** describes exactly how to restore.

### 8.4 User-facing safety nets (complementary, not a substitute)

- **PageVersion** history lets a writer recover an earlier version of a page.
- **Soft deletes** allow undo of accidental deletions.
- **Export/sync** (§9) means the writer always has their own copy too.

---

## 9. Export & sync (user owns their data)

Every writer must be able to **download and/or sync their data to their own
machine or cloud storage.**

### 9.1 One-off export ("Download everything")

- Produces a single **.zip** containing:
  - `pages/` — each page as a readable file (Markdown), organised by category
    folders.
  - `knowledge-base/` — each `AssistFile` (entities, indexes, summaries) as
    Markdown, with working relative links between them.
  - `manifest.json` — machine-readable structure (projects, categories, pages,
    entities, links) so the data can be re-imported or used elsewhere.
- Fully self-contained and human-readable; no lock-in.

### 9.2 Continuous sync to the writer's cloud (Google Drive first)

- **App-folder scope only** (the app sees only the folder it creates).
- Folder layout in the writer's Drive:
  ```
  /AI Writing Assistant/<Project>/
    ├── pages/<Category>/<page>.md
    └── knowledge-base/{characters,locations,items,...}/<entity>.md + index.md
  ```
- Sync triggers: after a page saves and after extraction updates assist files.
- **Content hashing** avoids redundant writes.
- Direction: the app is the writer of these files; hand-editing synced files in
  Drive is discouraged (reconciliation rules = §16 Q6/sync refinement).

### 9.3 Local machine

- The zip export covers "download to my machine".
- A desktop/folder-sync client is **out of scope for now** (revisit later);
  Drive-sync + export cover the requirement.

### 9.4 Formats

- Human-readable: **Markdown** (leaning; confirm §16 Q6).
- Machine-readable: **JSON** manifest.

---

## 10. AI access plan (the core)

This is the heart of the product: AIs read what the writer wrote and **create new
files** to assist them. Two access paths exist: the **built-in extraction
pipeline** (default, does the compiling) and an optional **programmatic AI
access API** (so external AI tools can read content and create assist files under
the writer's control).

### 10.1 What the AI may and may not do

- **May:** read pages (source content); create/update **assist files**
  (entities, facts, links, indexes, summaries); flag contradictions.
- **May not:** edit or delete the writer's pages; invent facts without a source;
  resolve contradictions on its own.
- **Every fact must cite a `TextSpan`.** No source → the fact is rejected.

### 10.2 The built-in extraction pipeline (default path)

Runs as a background job (§12). Steps:

1. **Ingest:** on page save/compile, load the page; create `TextSpan`
   scaffolding for traceability; compute `contentHash` (skip if unchanged).
2. **Chunk:** split the page into overlapping, context-window-sized chunks,
   tracking each chunk's offset so facts keep accurate positions.
3. **Extract (AI call):** for each chunk, ask the provider (using the **writer's
   own key**) to return **structured** entities + facts, **each with the exact
   source quote**, and to **invent nothing**. Send page text as data, not
   instructions (§7.6).
4. **Resolve entities:** new vs. known via name/alias/context. **Ask the writer
   when unsure** (decision Q7); auto-merge only when confident.
5. **Merge facts (timeline-aware):**
   - new info → `ACTIVE` fact with `timelineOrder`;
   - restatement → keep, note extra source;
   - **change over time** (later timeline position) → old fact becomes
     `HISTORICAL`, new becomes current (not a conflict);
   - **same-time conflict** → create a `Contradiction` (subject to the project's
     sensitivity), never overwrite.
6. **Cross-reference:** create/refresh `EntityLink`s (the hyperlink graph).
7. **Render assist files:** (re)generate `AssistFile` Markdown for affected
   entities + indexes — **these are the "new files that assist the writer."**
8. **Persist & sync:** save to DB; enqueue export/sync to the writer's storage
   (§9) if connected.
9. **Surface:** show new/updated entities and any contradictions for review.

### 10.3 Provider abstraction

- One internal interface: `extract(text, instructions) → structured facts with
  sources`. Implementations: `anthropic` (default), `openai` (optional).
- Model is a **setting** (`ProviderCredential.preferredModel` / `ANTHROPIC_MODEL`),
  never hard-coded.
- Keys are the **writer's own**, decrypted in memory server-side only, never sent
  to the browser.

### 10.4 Programmatic AI access API (optional, controlled)

So external AI tools/agents can also "access the content and create new files":

- The writer creates scoped **API tokens** (`ApiToken`) in Settings, e.g.:
  - `READ_CONTENT` — read pages;
  - `READ_KNOWLEDGE` — read entities/facts/links;
  - `WRITE_ASSIST` — create assist files/suggestions only (**never** write to
    pages).
- Tokens are **hashed at rest**, **scopable to one project**, **expiring**, and
  **revocable**; usage is audit-logged and rate-limited.
- The API mirrors the internal rules: reads are read-only; writes can only create
  assist files, never modify the writer's pages. This makes the "AI creates new
  files to assist" capability available to outside tools **without ever risking
  the writer's original work**.
- Consider exposing this as an **MCP-compatible** endpoint later so AI assistants
  can connect directly (refinement; §16).

### 10.5 Cost & safety controls

- Content hashing + per-chunk caching + incremental processing (only
  changed pages) to minimise AI calls.
- Rate limiting; per-job `tokenUsage` recorded for visibility.
- Structured-output validation + confidence scores; low-confidence items surfaced
  rather than trusted silently.

---

## 11. API surface

Internal REST-ish routes (Next.js route handlers), all authenticated and
owner-scoped unless noted. Names are indicative; validate all inputs (§7.5).

**Auth** (mostly via Auth.js): sign-in/out, session, MFA enrol/verify, sessions
list, revoke session.

**Projects:** `GET /projects`, `POST /projects`, `GET/PATCH/DELETE /projects/:id`
(includes `contradictionSensitivity`).

**Categories:** `GET /projects/:id/categories`, `POST …/categories`,
`PATCH/DELETE …/categories/:catId`, reorder.

**Pages:** `GET …/pages`, `POST …/pages`, `GET/PATCH/DELETE …/pages/:pageId`
(PATCH autosave → new `PageVersion`, enqueue extraction), `GET
…/pages/:pageId/versions`.

**Knowledge base:** `GET …/entities` (filter by type), `GET …/entities/:id`
(facts, links, timeline), `PATCH …/entities/:id` (correct/merge/split, confirm
aliases), `GET …/entities/:id/assist-file`.

**Contradictions:** `GET …/contradictions?status=OPEN`, `POST
…/contradictions/:id/resolve` (choice + note).

**Extraction/jobs:** `POST …/pages/:pageId/compile`, `GET …/jobs/:id`.

**AI keys & storage:** `POST/PATCH/DELETE /me/provider-credentials`,
`POST/DELETE /me/storage-connections`, OAuth callbacks.

**Export/sync:** `POST /projects/:id/export` (zip), `POST /projects/:id/sync`,
storage status.

**API tokens (programmatic AI access):** `GET/POST/DELETE /me/api-tokens`; and the
external, token-authenticated surface: `GET /api/v1/projects/:id/pages`,
`GET /api/v1/projects/:id/entities`, `POST /api/v1/projects/:id/assist-files`
(scope-gated; assist-file creation only, never page writes).

---

## 12. Background jobs

- **Queue:** BullMQ + Redis. **Worker:** separate process (`npm run worker`).
- **Extraction job** (per page): runs the §10.2 pipeline. Records status/usage on
  `ExtractionJob`; retries safely on failure; never loses a submission silently.
- **Sync/export jobs:** write pages + assist files to the writer's storage (§9).
- **Maintenance:** periodic backup verification hooks, token/session cleanup.
- Jobs are **idempotent** and safe to retry; use `contentHash`/`sourceHash` to
  avoid redundant work.

---

## 13. Coding standards

The owner is not (yet) a coder and wants to follow along. Therefore, **contrary to
minimal-comment conventions elsewhere, code here is heavily commented in plain
English**:

- Every file starts with a header comment: what it is, why it exists.
- Every function has a plain-English description of its job.
- Non-obvious steps get a one-line explanation of the "why".
- Prefer clear, boring, well-named code over clever code.
- Keep provider- and storage-specific code behind the small interfaces (§10.3, §9)
  so new providers don't ripple through the app.
- Security checklist on every change (from §7): authenticated? owner-scoped?
  inputs validated? secrets from env only? user text never treated as
  instructions? no secrets/content in logs? deletion/disconnect cleans up?

Folder structure (already scaffolded): `app/src/app` (UI + routes),
`app/src/server/{auth,db,storage,ai,extraction,jobs}` (server-only),
`app/src/shared` (safe shared types).

---

## 14. Configuration & environment

All config via environment variables; only names live in `app/.env.example`.
Required names include:

- `DATABASE_URL`, `REDIS_URL`
- `AUTH_SECRET`, `AUTH_URL`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`
- `KMS_KEY_ID` / `TOKEN_ENCRYPTION_KEY` (envelope-encryption master key reference)
- `ANTHROPIC_MODEL` (default model); **writers supply their own API keys in-app**
- `DEV_ANTHROPIC_API_KEY` (local dev only; blank in prod)
- `NODE_ENV`

Local dev: Docker for PostgreSQL + Redis; ESLint + Prettier for consistency.

---

## 15. Build order (phases → features)

Maps `docs/11` to the features above. Each phase stays shippable.

- **Phase 0 (now):** this spec + scaffold + schema. **Follow-up:** update
  `app/prisma/schema.prisma` to add `Category`, `Page`, `PageVersion`,
  `AssistFile`, `ApiToken` and the new enums per §6.
- **Phase 1 — walking skeleton:** front page (5.1), Google sign-in (5.2),
  onboarding + AI key (5.3), projects (5.4), categories + pages + editor with
  autosave & encryption (5.5–5.7), extraction pipeline (10.2) producing entities
  with sourced facts, basic entity pages (5.8). Content encryption + owner scoping
  + RLS from the start.
- **Phase 2 — knowledge base proper:** timeline on entity pages, cross-reference
  hyperlinks, aliases/entity resolution with "ask when unsure", search, priority
  ordering, assist-file rendering.
- **Phase 3 — contradictions & review:** detection (timeline-aware), review queue,
  sensitivity slider, jump-to-source (5.9–5.10).
- **Phase 4 — export & sync + backups hardening:** zip export, Google Drive
  app-folder sync (§9), verify backup/restore runbook (§8).
- **Phase 5 — provider choice, API tokens, security hardening, polish:**
  programmatic AI access API (10.4), 2FA, settings, privacy/export/delete,
  full security pass (§7).
- **Phase 6 — beyond:** more storage providers, story-time/flashbacks, MCP
  endpoint, collaboration, naming/branding & launch readiness.

---

## 16. Still open

Decide each when it's needed; none block Phase 1.

- **Q6** — confirm knowledge-base/export file format (leaning Markdown + JSON
  manifest) and Drive hand-edit reconciliation rules.
- **Q9** — product name & branding (affects front-page copy/visuals).
- **Q12** — offer email/password + which MFA methods at launch (Google-only is
  fine to start).
- **Q13** — exact data-retention & deletion windows (incl. backup retention);
  needed before non-internal launch + privacy policy.
- **Q14** — in-app source editing vs. edit-in-your-own-tool for the "edit the
  source" contradiction resolution.
- **Q16** — bring-your-own-key onboarding details (which providers at launch, how
  much in-app guidance, invalid/no-credit handling).
- **Refinements** — explicit story-time for non-linear narratives; MCP endpoint
  for direct AI-assistant access; desktop folder-sync client; hosting platform
  choice.

---

*End of master build reference. Keep this file updated as the source of truth.*
