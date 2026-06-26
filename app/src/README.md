# `src/` — Application Source

This is where the website's code will live. It is split into three top-level
areas so it's always clear whether a piece of code runs in the **browser**, on
the **server**, or is **shared** by both.

| Folder | Runs where | Responsibility |
|--------|-----------|----------------|
| `app/` | Browser + server (Next.js) | The pages and routes the writer interacts with (doc 10). |
| `server/` | Server only | All backend logic; the browser never sees this code or its secrets. |
| `shared/` | Both | Plain types and helpers reused on both sides (no secrets, no DB). |

## The golden rule of this split

> **Secrets, database access, cloud-storage tokens, and AI API keys live ONLY in
> `server/`.** They must never be imported into browser code. This is a core
> security boundary (see `../../docs/05-security-and-privacy.md`).

## `server/` sub-folders

| Folder | Responsibility | Doc |
|--------|----------------|-----|
| `server/auth/` | Sign-in & sessions ("Sign in with Google"). | 05 |
| `server/db/` | Database access via Prisma. | 04 |
| `server/storage/` | Cloud storage (Google Drive first), behind one interface. | 06 |
| `server/ai/` | AI provider integrations, pluggable behind one interface. | 07 |
| `server/extraction/` | The extraction pipeline + fact-merge + contradiction logic. | 07, 09 |
| `server/jobs/` | The background job queue and the worker that runs extraction. | 02, 07 |

Each sub-folder has its own README explaining its job in more detail.

## Status

These folders currently contain only READMEs (the documented plan). Feature code
is added phase by phase per `../../docs/11-development-roadmap.md`, following the
heavy-commenting standard in `../README.md`.
