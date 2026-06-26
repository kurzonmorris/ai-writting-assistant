# `src/shared/` — Shared Types & Helpers

Code here is safe to use on **both** the browser and the server. It contains the
common "shapes" of our data and small pure helper functions, so the frontend and
backend agree on the same definitions and don't drift apart.

## What belongs here

- **Types** describing our core concepts: Entity, Fact, Contradiction, Project,
  etc. (mirroring the data model in `../../../docs/04-database-design.md` and the
  knowledge model in `../../../docs/08-entity-knowledge-model.md`).
- **Constants** like the list of entity types or relationship types.
- **Small pure helpers** (formatting, validation shapes) with no side effects.

## What must NOT go here

- ❌ Anything secret (API keys, tokens).
- ❌ Database access (that's `server/db/`).
- ❌ Network calls to AI providers or storage (those are server-only).

Because this code can run in the browser, putting any secret or server-only logic
here would leak it. Keep it pure and safe (security boundary — doc 05).

## Status

Plan only. Populated as shared types are needed from Phase 1 onward.
