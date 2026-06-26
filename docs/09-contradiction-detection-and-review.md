# 09 — Contradiction Detection & Review

One of the product's signature features: when the writer's text contradicts
itself, the assistant **notices, flags it, and takes the writer to the exact spot
to fix it** — without ever deciding the answer itself.

This is a direct expression of the guiding rule (doc 01): **the AI flags; the
human decides.**

## What counts as a contradiction?

A contradiction is when two facts about the **same entity** and the **same
attribute** cannot both be true:

- Eye colour is "blue" in chapter 2 and "green" in chapter 18.
- A character is "an only child" in one passage and "has a sister" in another.
- A building is "north of the river" and later "south of the river".
- A character is described as dead, then later speaks with no explanation.

Not every difference is a contradiction. Things that **change legitimately over
time** (location, mood, injuries, status) are not conflicts — they're the story
progressing. Distinguishing "changed over time" from "inconsistent" is subtle and
ties into the chronology question marked **OPEN** in doc 00. The safe default:
when unsure, **flag for review** rather than guess.

## How detection works

During the merge step of the pipeline (doc 07, step 5), each newly extracted fact
is compared against existing `ACTIVE` facts for the same entity + attribute:

1. **Compatible** (same or additive) → store normally.
2. **Conflicting** → create a `Contradiction` record:
   - Link `factA` (existing) and `factB` (new).
   - Mark both facts `DISPUTED` (they stay in the DB — nothing is deleted).
   - Set the contradiction `status` to `OPEN`.

Because every fact carries a `TextSpan` (doc 04), the contradiction automatically
knows the **exact source location of both sides**.

Some conflicts are obvious string differences ("blue" vs "green"); others are
**semantic** ("only child" vs "has a sister") and need the AI to judge meaning.
We use the AI to assess semantic conflicts but **only to detect and explain
them** — never to resolve them.

## The review experience

When contradictions exist, the writer sees a **review queue**. For each one:

```
⚠ Contradiction — WILLIAM · Eye colour

  Version A:  "blue"
              Chapter 2  ·  “…his pale blue eyes studied the horizon.”
              [ Go to this spot ]

  Version B:  "green"
              Chapter 18 ·  “…her reflection in his green eyes.”
              [ Go to this spot ]

  What would you like to do?
   ( ) Keep A as canon        ( ) Keep B as canon
   ( ) Both are valid (e.g. changes over time)
   ( ) Edit the source text
   ( ) Dismiss (not actually a conflict)
```

### "Go to this spot" — jumping to the exact location
This is the key promise. Each side links to its `TextSpan`'s `startOffset`/
`endOffset` in its `SourceDocument`. Selecting it opens the source document
scrolled to and highlighting that exact passage, so the writer can read it in
context and edit if they choose. No more "trawling to find the info".

## Resolution outcomes

| Choice | What happens |
|--------|--------------|
| **Keep A as canon** | Fact A → `ACTIVE`; Fact B → `SUPERSEDED`. Contradiction → `RESOLVED`. |
| **Keep B as canon** | Fact B → `ACTIVE`; Fact A → `SUPERSEDED`. Contradiction → `RESOLVED`. |
| **Both valid (changes over time)** | Both kept; recorded as time-dependent (ties to chronology, doc 00). Contradiction → `RESOLVED`. |
| **Edit the source text** | Writer fixes the manuscript at the exact spot; re-extraction updates the facts. Contradiction → `RESOLVED` once consistent. |
| **Dismiss** | Not a real conflict. Contradiction → `DISMISSED`; both facts return to `ACTIVE`. |

Crucially, **the writer makes the call.** The AI presents both sides with
evidence and never overwrites canon on its own.

> **A note on "Edit the source text":** even here, the *writer* edits their own
> words. The AI does not rewrite the manuscript (doc 01). If we offer any
> in-app editing of source text, it's the writer typing — re-extraction then
> reflects the change. Whether source editing happens in-app or in the writer's
> own tools is **OPEN** (doc 00).

## Recording the decision

Every resolution updates the `Contradiction` record with `resolutionNote`,
`resolvedByUserId`, and `resolvedAt`, and is written to the `AuditLog` (doc 04).
This gives the writer a history of the canon decisions they've made.

## Avoiding nagging

If the same contradiction would re-appear on every extraction, that's annoying.
So:
- A `RESOLVED`/`DISMISSED` contradiction is **remembered**; we don't re-flag the
  same pairing unless the underlying source text changes.
- The writer can see resolved/dismissed items in a history view, and reopen one
  if they change their mind.

## Open design questions (see doc 00)

- How to model **change over time** so "both valid" is represented cleanly.
- How sensitive detection should be (more flags = safer but noisier).
- Whether to batch contradictions by entity, severity, or recency in the queue.
- How much semantic-conflict detection to trust automatically vs. surface.
