# 10 — User Interface & Flows

This document describes the **screens** the writer uses and the **journeys**
through them. It's deliberately high-level (no visual design yet) so we agree on
*what* each screen does before deciding how it looks.

> Visual design (colours, branding, layout polish) is **OPEN** — and partly
> waits on naming the product. Tracked in doc 00.

## The main screens

### 1. Sign in / Welcome
- Explains the product in a sentence and the one rule ("compiles, never writes").
- **"Sign in with Google"** button.
- After first sign-in, prompts to **connect cloud storage**.

### 2. Connect storage
- "Connect Google Drive" → Google's consent screen (doc 06).
- Shows connection status; allows **disconnect**.
- Explains, in plain English, what access is granted and why.

### 3. Projects dashboard
- Lists the writer's projects (novel, campaign, etc.) with type and last activity.
- **"New project"** with a title and a kind (doc 04 `Project.kind`).
- Each project card links into its workspace.

### 4. Project workspace (the home base for one work)
A layout with three main areas:
- **Left:** the knowledge base navigator — entities grouped by type (Characters,
  Locations, Items…), searchable.
- **Centre:** whatever the writer is viewing (an entity page, a source document,
  the review queue).
- **Top/banner:** project title, an **"Add text"** action, and a **review badge**
  showing how many contradictions are open.

### 5. Add / import text
- Paste text directly, or import a document from connected storage.
- On submit: shows **"Processing…"** with job progress (doc 07 runs in the
  background, so this screen never freezes).
- When done: a summary of what was found (new entities, updated entities,
  contradictions to review).

### 6. Entity page
- The wiki-style reference page for one entity (the example in doc 08).
- Facts grouped by category, **each showing its source** with a link to the exact
  passage.
- For time-varying details (location, status…), shows the **current** value and a
  **timeline / history** of how it changed (doc 08 chronology).
- Related entities listed as **hyperlinks** (the cross-reference graph).
- Disputed facts visibly marked, with a link into the review queue.
- "Appears in" list of source documents.
- Entity-type groups in the navigator follow the **priority order** (characters,
  places, environments, factions, equipment, items — doc 08).

### 7. Source document viewer
- Read a source document.
- Supports **deep links to an exact span** so "Go to this spot" (doc 09) lands
  precisely and highlights the passage.

### 8. Review queue (contradictions)
- The list of `OPEN` contradictions (doc 09).
- Each item shows both sides with sources and the resolution choices.
- A history view of resolved/dismissed items.
- A **sensitivity slider** (Relaxed ↔ Strict) the writer can adjust on the fly,
  per project, to control how eagerly conflicts are flagged (doc 09).

### 9. Settings
- **AI provider & model** selection (doc 07), with a clear note about which
  provider the writer's text is sent to (doc 05).
- Storage connection management.
- Account, privacy controls, data export/delete.

## Key user journeys

### Journey A — First-time setup
```
Sign in with Google
  → Connect Google Drive (approve minimal permissions)
  → Create first project (title + kind)
  → Add first text
  → See the knowledge base begin to populate
```

### Journey B — Everyday use ("I wrote a new chapter")
```
Open project
  → Add text (paste/import the new chapter)
  → Wait for processing (background job)
  → Review summary: new + updated entities
  → Resolve any flagged contradictions
  → Browse the now-updated knowledge base
```

### Journey C — "What did I say about that place ages ago?"
```
Open project
  → Search or browse to the location in the navigator
  → Read its entity page (all compiled facts, with sources)
  → Follow hyperlinks to related people/items
  (No trawling through chapters.)
```

### Journey D — Resolving a contradiction
```
Notice the review badge
  → Open the review queue
  → Read both conflicting versions with their sources
  → "Go to this spot" to read each in context
  → Choose: keep A / keep B / both valid / edit / dismiss
  → Canon updated; decision recorded
```

## UX principles for this product

1. **Sources are always visible.** Every fact shows where it came from; trust is
   built on traceability (doc 01).
2. **The writer is never blocked.** Slow AI work happens in the background with
   clear progress; the UI stays responsive.
3. **Reviewing is fast.** Contradiction review is two clicks plus a decision,
   with the exact source one click away.
4. **Hyperlinks everywhere.** Any entity name is clickable; the knowledge base
   feels like a personal wiki.
5. **Nothing destructive is silent.** Merges that supersede facts, deletions, and
   disconnects are clearly shown and reversible where possible.
6. **Plain language.** No jargon in the UI; explain what the AI is doing.

## Accessibility & platform

- Web-first, responsive so it works on the desktop and reasonably on tablets.
- Follow basic accessibility practice (keyboard navigation, sufficient contrast,
  semantic markup).
- **OPEN:** whether a mobile-optimised experience or native app is wanted later
  (doc 00). The owner's primary devices are noted in their profile, but the app
  is browser-based so it isn't tied to one OS.
