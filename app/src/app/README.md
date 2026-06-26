# `src/app/` — Pages & Routes (what the writer sees)

This folder holds the **user-facing** part of the website: the screens described
in [`../../../docs/10-user-interface-and-flows.md`](../../../docs/10-user-interface-and-flows.md),
plus the small server routes (API endpoints) the frontend calls.

In Next.js, this folder defines both pages and routes by file structure.

## Planned screens (from doc 10)

- **Welcome / sign in** — explains the product and the one rule; "Sign in with Google".
- **Connect storage** — connect/disconnect Google Drive (doc 06).
- **Projects dashboard** — list/create projects.
- **Project workspace** — knowledge-base navigator + main view + review badge.
- **Add / import text** — paste or import; shows background-job progress.
- **Entity page** — the wiki-style reference page with sourced facts + links.
- **Source document viewer** — supports deep-linking to an exact span.
- **Review queue** — resolve contradictions (doc 09).
- **Settings** — AI provider/model, storage, account, privacy.

## Rules for code here

- This code may run in the browser, so it must **never** import secrets, the
  database, or provider keys. It talks to the backend through API routes only
  (security boundary — doc 05).
- Keep components small and clearly named; comment what each screen does
  (commenting standard — `../../README.md`).

## Status

Plan only. Screens are built starting in Phase 1 (doc 11).
