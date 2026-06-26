# `server/extraction/` — The Extraction Pipeline

The heart of the product: turning raw text into a structured, cross-referenced,
contradiction-checked knowledge base. The full step-by-step design is in
[`../../../../docs/07-ai-extraction-pipeline.md`](../../../../docs/07-ai-extraction-pipeline.md)
and the contradiction logic in
[`../../../../docs/09-contradiction-detection-and-review.md`](../../../../docs/09-contradiction-detection-and-review.md).

## The steps implemented here

1. **Chunk** the source text into pieces that fit the AI's context window,
   tracking each chunk's offset so facts keep accurate source positions.
2. **Extract** by calling the AI provider (`../ai/`) for each chunk.
3. **Resolve entities** — decide whether each mention is a new entity or an
   existing one (using names/aliases; prefer asking over wrong merges).
4. **Merge facts** — add agreeing facts; for conflicts, create a
   `Contradiction` instead of overwriting (the careful part).
5. **Cross-reference** — build/refresh `EntityLink`s (the hyperlink graph).
6. **Persist** to the database and hand off the cloud-storage sync.

## The unbreakable rules (doc 01)

- The AI **compiles and cross-references; it never writes or edits the story.**
- **Every fact cites a source.** Facts without a traceable source are dropped.
- **Conflicts are flagged, never auto-resolved.** Nothing is overwritten silently.

## Runs in the background

This pipeline is executed by a background worker (`../jobs/`), not during a web
request, because it can take seconds to minutes (doc 02/07).

## Status

Plan only. Core pipeline (steps 1–2, basic 3–6) in Phase 1; contradictions in
Phase 3 (doc 11).
