# 11 — Development Roadmap

A **phased** plan that gets a usable, testable product into the owner's hands
early, then layers on capability. Each phase produces something that works
end-to-end, so we always have a real thing to try rather than a half-built
everything.

Phases are about **order and focus**, not fixed dates. Internal development and
testing come first (no official product name yet).

## Phase 0 — Foundations (planning & scaffold) ✅ in progress
**Goal:** agree the plan and stand up the skeleton.
- [x] Design documents (this `docs/` folder).
- [x] Repository structure and documented scaffold (`app/`).
- [x] Database design expressed as Prisma schema.
- [ ] Resolve the highest-priority open questions (doc 00).
- [ ] Confirm technology choices and hosting direction.

## Phase 1 — Walking skeleton (smallest end-to-end slice)
**Goal:** prove the core loop works for one provider, one storage, locally.
- Sign in with Google (Auth.js).
- Create a project.
- Paste text (no cloud storage yet — internal storage only).
- Run extraction with the default AI provider (Claude) in the background.
- See entities and facts with their sources in a basic UI.
**Definition of done:** paste a chapter, get a populated, sourced knowledge base.

## Phase 2 — The knowledge base, properly
**Goal:** make the compiled output genuinely useful to browse.
- Entity pages grouped by type, with attributes and sources (doc 08).
- Cross-reference hyperlinks between entities (`EntityLink`).
- Aliases and entity resolution (new vs. known) (doc 07 step 4).
- Search and navigation in the project workspace (doc 10).

## Phase 3 — Contradictions & review
**Goal:** deliver the signature feature.
- Contradiction detection during merge (doc 09).
- Review queue with both sides + sources.
- "Go to this spot" deep-linking into the source document.
- Resolution outcomes recorded; no re-nagging on resolved items.

## Phase 4 — Cloud storage as permanent memory
**Goal:** the writer owns their data in their own Drive.
- Google Drive connect/disconnect with minimal scope (doc 06).
- Save source documents and serialised knowledge base to the writer's Drive.
- Sync rules and content-hash skip-reprocessing.
- Import existing documents from storage.

## Phase 5 — Provider choice & polish
**Goal:** flexibility and trust.
- Pluggable providers; let the writer choose provider/model (doc 07).
- Cost/usage visibility per project.
- Settings, privacy controls, data export/delete (doc 05/10).
- Security hardening pass (RLS, rate limits, headers, audit log review).

## Phase 6 — Beyond (candidate features, not committed)
Tracked loosely; pulled forward only if wanted (see doc 00):
- More storage providers (Dropbox, OneDrive).
- Timeline / chronology modelling ("as of chapter N").
- Bulk import of an existing manuscript.
- Collaboration / sharing.
- Bring-your-own-API-key / privacy-first provider mode.
- Naming, branding, and public launch readiness.

## How we'll work (process)

- **Internal first.** Everything is tested privately until the owner is happy.
- **Heavily commented code** at every phase (see `app/README.md`).
- **Commit and push** completed work to the working branch as we go.
- **Each phase stays shippable** — we don't leave the product broken between
  phases.
- **Open questions** are revisited at the start of each phase so we decide things
  at the moment we actually need them, not prematurely.

## Suggested immediate next steps

1. Owner answers the priority questions in doc 00.
2. Lock Phase 1 scope.
3. Set up the local development environment (Docker for Postgres/Redis, env vars).
4. Build the walking skeleton (Phase 1).
