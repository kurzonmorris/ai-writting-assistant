# 06 — Cloud Storage Integration

The writer's own cloud storage is the **permanent memory** of the system. The
website keeps a small, fast working store; the durable home for the writer's
source documents and their compiled knowledge base is **their** storage account.
Google Drive is the first provider we integrate.

## Why cloud storage as permanent memory?

- **The writer owns their work.** It lives in their account, under their control.
- **No lock-in.** If the writer leaves, they keep everything in a readable form.
- **The app stays light.** We don't have to be a large, expensive file host; our
  database is an *index* over the writer's files.
- **Resilience.** If our database were ever lost, the writer's own copy survives.

## What we store where

| Data | Lives in app DB | Lives in cloud storage |
|------|-----------------|------------------------|
| Source documents (original prose) | reference + spans needed for traceability | **canonical copy** |
| Compiled knowledge base (entity files) | structured, queryable copy | **human-readable serialised copy** |
| Cross-reference links | yes (queryable) | embedded in serialised files |
| Account, jobs, tokens | yes | no |

The app DB is optimised for **fast queries and the live UI**. The cloud copy is
optimised for **durability and writer ownership**.

## Folder layout we create in the writer's storage

When a writer connects storage, we create a single app-owned root folder so we
never clutter or touch the rest of their Drive:

```
/AI Writing Assistant/
├── <Project Title>/
│   ├── source/                ← original imported documents
│   │   ├── chapter-01.md
│   │   └── ...
│   └── knowledge-base/        ← the compiled story bible (human-readable)
│       ├── characters/
│       │   └── <character>.md
│       ├── locations/
│       ├── items/
│       └── index.md           ← overview + links
└── ...
```

The exact file format (Markdown is the leading candidate — readable, portable,
supports links) is **OPEN** and tracked in doc 00.

## How connecting works (OAuth, in plain English)

1. The writer clicks **"Connect Google Drive"**.
2. We send them to Google's own consent screen. They log in *to Google*, not to
   us — we never see their Google password.
3. Google asks them to approve the **specific, minimal permissions** we request.
4. Google hands our backend a **token** that lets us act on the writer's behalf
   within those permissions.
5. We **encrypt and store** that token (doc 05) and create a `StorageConnection`
   record (doc 04).

## Minimal permissions (least privilege)

We request the **smallest scope that works**. The preferred option is the
**app-folder / per-file scope**, where the app can only see and manage files it
itself created — *not* the writer's entire Drive. This dramatically limits what a
stolen token could expose.

- **OPEN:** confirm the exact Google Drive scope. The trade-off is convenience
  (importing the writer's existing documents from anywhere in their Drive) vs.
  privacy (only touching app-created files). Tracked in doc 00.

## Syncing rules (keeping DB and storage in agreement)

- **On import:** the original text is saved to `source/` in cloud storage, and a
  `SourceDocument` + `TextSpan`s are created in the DB.
- **After extraction:** the updated entity files are written to
  `knowledge-base/` in cloud storage, reflecting the DB state.
- **Conflict handling:** the DB is the source of truth for live editing; cloud
  storage is refreshed from it. If a writer edits a knowledge-base file directly
  in their Drive, how we reconcile that is **OPEN** (doc 00) — the safe default
  is to treat the app as the writer of knowledge-base files and warn against
  hand-editing them.
- **Content hashing:** we store a `contentHash` per source document so unchanged
  text is never needlessly re-processed (saving time and AI cost).

## Disconnecting

- The writer can disconnect storage at any time. We **revoke** the token with the
  provider and **delete** it from our database.
- Their files remain in their own storage, untouched. They keep everything.

## Designing for more providers later

The integration is written against a small internal **storage interface**
(connect, list, read file, write file, create folder, disconnect). Google Drive
is the first implementation. Adding Dropbox or OneDrive later means writing
another implementation of the same interface, without changing the rest of the
app. (Which providers, and when, is OPEN — doc 00.)

## Internal storage (the app's small footprint)

The app keeps only what it needs to operate quickly:
- Temporary copies of text while a job runs.
- The structured index (the database).
- Caches to avoid repeating expensive AI work.

It is explicitly **not** intended to be the writer's main file store — that role
belongs to the writer's connected cloud storage.
