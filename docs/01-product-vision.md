# 01 — Product Vision

## The problem

Writers of long, detailed fiction — novels, series, game design documents,
tabletop RPG (TTRPG) campaigns — accumulate an enormous amount of internal
"canon": what a character looks like, how they speak, who owns which sword, what
the weather is like in a particular city, which faction controls which territory.

As a project grows, keeping this consistent becomes painful:

- A detail mentioned "in passing multiple chapters ago" is hard to find again.
- Contradictions creep in (blue eyes in chapter 2, green eyes in chapter 18).
- Maintaining a separate "story bible" by hand is tedious, so most people don't,
  or they let it fall out of date.

## The solution

A website that **reads the writer's own prose and builds the story bible for
them automatically** — then keeps it cross-referenced and consistent.

The writer keeps writing the way they always have. The assistant quietly:

1. **Scans** each new piece of text for clues about people, places, and things.
2. **Compiles** what it finds into one structured file per entity.
3. **Merges** new details into existing entity files as more text arrives.
4. **Cross-references** everything with hyperlinks, so any mention of a place,
   person, or item links straight to that entity's file.
5. **Flags contradictions** for the writer to resolve, jumping them to the exact
   spot in the source text — but never resolving them automatically.

The outcome: a comprehensive, always-current, browsable reference library that
the writer can consult instead of re-reading their manuscript.

## The guiding principle (non-negotiable)

> **The AI compiles and cross-references. It never writes or edits the story.**

This shapes every feature decision:

- The AI **extracts** facts the writer already wrote; it does not **invent** them.
- The AI never rewrites, paraphrases into the manuscript, or "improves" prose.
- When two facts conflict, the AI **asks**, it does not **decide**.
- Every fact in the knowledge base is **traceable** to a specific location in the
  source text. If the AI can't point to where it learned something, it shouldn't
  claim it.

The AI is a **librarian and assistant**, never a co-author.

## Who it's for

The primary user is a **solo fiction writer or worldbuilder** who:

- Writes long-form or multi-part work where consistency matters.
- Already has a body of text (or is actively producing it).
- Wants a reference tool, not a writing tool.

Concrete user types we design for:

| User type | Example need |
|-----------|--------------|
| Novelist | Track a large cast across a trilogy without continuity errors. |
| Series writer | Keep canon consistent between books written years apart. |
| Game designer | Maintain a coherent design bible for lore, items, and systems. |
| TTRPG game master | Build a campaign wiki from session notes and prep documents. |
| Worldbuilder / hobbyist | Organise a sprawling personal world for fun. |

## What success looks like

- A writer pastes in a chapter and, seconds to minutes later, sees new and
  updated entity files with correctly attributed facts.
- The writer can click any character/place/item name and jump to its file.
- When the writer introduces a contradiction, they are told clearly, shown both
  sources, and taken to the exact spot to fix it.
- The writer trusts the knowledge base enough to rely on it instead of their
  manuscript.

## Explicit non-goals (at least for now)

These are deliberately **out of scope** to keep the product focused. They are
recorded so we remember we chose to exclude them on purpose.

- **AI as a writing tool** — no drafting, ghost-writing, or prose suggestions.
- **Auto-editing the manuscript** — the AI never changes the writer's words.
- **Being the primary file store** — cloud storage (the writer's own) is the
  permanent home for files; the site keeps only what it needs to operate.
- **Real-time multi-author collaboration** — single-writer focus first.
  (Revisit later; tracked in doc 00.)
- **Publishing / export-to-print pipelines** — not what this is for.

## Core values baked into the design

1. **The writer owns their work.** Their files live in *their* cloud storage.
   They can disconnect and walk away with everything.
2. **Nothing is fabricated.** Every fact is sourced. No guessing presented as
   truth.
3. **The writer is always in control.** The AI suggests and flags; the human
   decides.
4. **Privacy first.** A person's unpublished fiction is sensitive. We treat it
   that way (see doc 05).
5. **Understandable by humans.** Both the product and its code are built to be
   followed by non-experts.
