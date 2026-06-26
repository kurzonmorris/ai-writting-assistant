# 07 — AI Extraction Pipeline

This document describes **how raw text becomes structured knowledge** — the
heart of the product. It is written so a non-coder can follow each step and
understand why it exists.

Remember the guiding rule (doc 01): **the AI compiles and cross-references; it
never writes or edits the story.** Every step below extracts and organises facts
the writer already wrote. Nothing is invented.

## The pipeline, step by step

```
 Source text
     │
     ▼
[1] Ingest & store          → save original, create source document + spans
     │
     ▼
[2] Chunk                   → split into pieces that fit the AI's context window
     │
     ▼
[3] Extract (AI call)       → per chunk: find entities + facts + their sources
     │
     ▼
[4] Resolve entities        → is this a NEW entity or one we already know?
     │
     ▼
[5] Merge facts             → add agreeing facts; flag conflicting ones
     │
     ▼
[6] Cross-reference         → build hyperlinks between entities
     │
     ▼
[7] Persist & sync          → save to DB and to the writer's cloud storage
     │
     ▼
[8] Surface results         → show new/updated entities + contradictions to review
```

### Step 1 — Ingest & store
The writer pastes or imports text. We save the original to cloud storage and
create a `SourceDocument` plus the `TextSpan` machinery (doc 04) so that every
fact we later extract can point back to an exact character range. Traceability
starts here.

### Step 2 — Chunk
AI models can only read so much at once (their "context window"). Long chapters
are split into overlapping chunks at sensible boundaries (paragraphs/scenes). We
track each chunk's offset within the document so extracted facts keep accurate
source positions. Overlap avoids losing facts that straddle a boundary.

### Step 3 — Extract (the AI call)
For each chunk we ask the AI a **tightly constrained** question, roughly:

> "Here is a passage of fiction. Identify the entities that appear (characters,
> locations, items, creatures, etc.). For each, list the concrete facts the text
> states or clearly implies about it. For every fact, quote the exact phrase it
> came from. Do **not** invent anything not supported by the text. Return the
> result in this exact structured format."

Key properties:
- The writer's text is supplied as **data to analyse, never as instructions**
  (prompt-injection defence — doc 05).
- We require **structured output** (e.g. JSON) so the result is machine-readable.
- We require a **source quote** per fact; facts without a traceable source are
  rejected.
- We instruct the model to extract a rich range of detail — appearance, clothing,
  personality, **how a person speaks**, relationships, possessions, history —
  anything the text genuinely supports (entity-specific attributes in doc 08).

### Step 4 — Resolve entities (new vs. known)
Is "the captain" in this chapter the same person as "William" from chapter 2?
Entity resolution decides this using:
- Name and **alias** matching (doc 04 `EntityAlias`).
- Context similarity (the surrounding facts).
- When uncertain, we **prefer to ask** rather than wrongly merge two entities.
  Over-merging is worse than a duplicate, because it corrupts the knowledge base.
- **OPEN:** how aggressive auto-merge should be, and the review UX for uncertain
  matches (doc 00).

### Step 5 — Merge facts (the careful part)
For each extracted fact about a known entity:
- **New information** (we didn't know it) → store it as an `ACTIVE` fact.
- **Restatement** (we already knew it, consistent) → keep one fact, optionally
  noting the additional source.
- **Contradiction** (conflicts with an existing `ACTIVE` fact) → **do not
  overwrite.** Create a `Contradiction` record linking the two facts and mark
  them as `DISPUTED`. The writer decides later (doc 09).

This is where "blue eyes vs. green eyes" is caught. The AI never picks a winner.

### Step 6 — Cross-reference
We create/refresh `EntityLink` rows (doc 04) wherever the text shows a
relationship: a character **owns** an item, a building is **located in** a city,
a person is a **member of** a faction, or one entity simply **mentions** another.
These links become the hyperlinks the writer follows in the UI.

### Step 7 — Persist & sync
Results are written to the database (the live index) and the updated knowledge
base is serialised to the writer's cloud storage (doc 06). Provenance is recorded
on the `ExtractionJob` so we always know which run produced which facts.

### Step 8 — Surface results
The UI shows what changed: newly discovered entities, updated entities, and a
**review queue** of any contradictions, each linking straight to the exact source
locations (doc 09, doc 10).

## Running as a background job

Steps 2–7 happen inside a **background worker** (doc 02), not during the web
request, because they can take seconds to minutes. The writer gets an immediate
"processing…" response and sees results when the job finishes. Each job's status
(`QUEUED`/`RUNNING`/`SUCCEEDED`/`FAILED`) is tracked so the UI can show progress
and so failures can be retried safely.

## Provider abstraction (pluggable AI)

We talk to AI providers through a small internal **extraction interface** with one
core operation: *given text + the extraction instructions, return structured
facts with sources.* Each provider (Anthropic Claude, OpenAI, etc.) is one
implementation behind that interface. Benefits:

- The writer can **choose** their provider/model.
- We can **add** providers without touching the rest of the pipeline.
- We default to **Anthropic Claude (latest model)** for its strong long-context,
  structured extraction.

The chosen model is a **setting**, not hard-coded, so we can adopt newer models
easily.

## Controlling cost and avoiding rework

AI calls cost money and time. We keep them down by:
- **Content hashing** (doc 06): never re-process unchanged text.
- **Caching** extraction results per chunk hash.
- **Incremental processing**: only new/changed documents are extracted.
- **Rate limiting** (doc 05) to prevent runaway usage.
- Recording `tokenUsage`/cost on each `ExtractionJob` for visibility.

## Quality safeguards (keeping the AI honest)

- **Source required** — facts without a quotable source are dropped.
- **Schema validation** — the AI's output must match the expected structure or
  it's rejected and retried.
- **Confidence scores** — low-confidence extractions can be surfaced for review
  rather than trusted silently.
- **No fabrication instruction** — the prompt explicitly forbids inventing facts,
  and the source-quote requirement makes fabrication detectable.
- **OPEN:** how much to trust auto-merge vs. asking the writer; thresholds for
  confidence; whether to show the writer a per-fact "sources" list (doc 00).
