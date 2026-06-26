# `src/server/` — Backend Logic (server-only)

Everything here runs **only on the server**. The browser never receives this
code, so this is where all secrets, database access, cloud-storage tokens, and AI
API keys live.

> **Security boundary:** never import anything from `server/` into browser-facing
> code in `src/app/`. The frontend reaches the backend only through API routes.
> See `../../../docs/05-security-and-privacy.md`.

## Sub-folders

| Folder | Responsibility | Doc |
|--------|----------------|-----|
| `auth/` | Sign-in and session handling. | 05 |
| `db/` | Database access via Prisma; the only place that talks to Postgres. | 04 |
| `storage/` | Cloud storage (Google Drive first), behind one interface. | 06 |
| `ai/` | AI provider integrations, pluggable behind one interface. | 07 |
| `extraction/` | The extraction pipeline, fact merging, contradiction detection. | 07, 09 |
| `jobs/` | The background job queue and worker process. | 02, 07 |

## Cross-cutting rules

- Validate **all** input at the boundary before using it (doc 05).
- Read secrets from environment variables only — never hard-code them.
- Keep provider- and storage-specific code behind the small interfaces in `ai/`
  and `storage/`, so we can add providers without touching the pipeline.
- Comment generously in plain English (commenting standard — `../../README.md`).

## Status

Plan only. Built phase by phase per `../../../docs/11-development-roadmap.md`.
