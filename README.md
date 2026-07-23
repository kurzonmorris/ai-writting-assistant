# AI Writing Assistant (working title)

> An AI-powered **story bible** that reads a writer's prose and automatically
> builds a cross-referenced, hyperlinked reference library — characters, places,
> items, creatures, factions and more — so the writer never has to scroll back
> through chapters to remember what they already established.

This project does **not** have an official public name yet. It is being
developed and tested internally. Everything here is currently a working title.

---

## The core idea in one paragraph

A writer signs in and connects their own cloud storage (e.g. Google Drive),
which acts as the **permanent memory** for all of their files. The website
itself keeps only a small amount of internal storage. The writer pastes or
imports text from their novel, short story, game design doc, or tabletop RPG
(TTRPG) campaign. A connected AI model (Claude, ChatGPT, or another provider)
**scans the text for clues** and compiles what it finds into structured
reference files — one per character, place, item, creature, etc. As the writer
adds more text, the AI merges new details into the existing files. If it finds a
**contradiction** (e.g. a character's eyes are blue in one chapter and green in
another), it does **not** silently fix it — it **flags the conflict for review**
and takes the writer straight to the exact spot in the source text so they can
decide. The end result is a comprehensive, hyperlinked knowledge base the writer
can browse instead of hunting through their manuscript.

## The one rule that defines this product

> **The AI compiles and cross-references. It never writes or edits the story.**

The AI is an **assistant and librarian**, not a co-author. It extracts facts
that the writer already wrote and organises them. It never invents prose, never
rewrites the manuscript, and never "improves" the story. Every fact in the
knowledge base traces back to something the writer actually wrote.

---

## Repository layout

```
ai-writting-assistant/
├── README.md                 ← you are here: project orientation
├── docs/                     ← all design decisions live here (read these first)
│   ├── README.md             ← index + reading order for the design docs
│   ├── 00-open-questions.md   ← decisions still to be made (living document)
│   ├── 01-product-vision.md
│   ├── 02-system-architecture.md
│   ├── 03-technology-stack.md
│   ├── 04-database-design.md
│   ├── 05-security-and-privacy.md
│   ├── 06-cloud-storage-integration.md
│   ├── 07-ai-extraction-pipeline.md
│   ├── 08-entity-knowledge-model.md
│   ├── 09-contradiction-detection-and-review.md
│   ├── 10-user-interface-and-flows.md
│   ├── 11-development-roadmap.md
│   └── 12-glossary.md
└── app/                      ← the website itself (scaffold + planned structure)
    ├── README.md             ← how the code is organised and why
    ├── .env.example          ← every configuration value the app needs
    ├── package.json          ← planned dependencies
    ├── prisma/
    │   └── schema.prisma     ← the database design, as runnable schema
    └── src/                  ← application source (folders documented by READMEs)
```

## How to read this repository

> **Building the site?** Start with
> [`writers knowledge base.md`](writers%20knowledge%20base.md) — the single,
> authoritative, build-ready reference that consolidates every screen, data
> structure, security rule, backup/export plan, and the AI-access design. The
> numbered `docs/` files below are the longer-form reasoning behind each decision.

1. Start with [`docs/01-product-vision.md`](docs/01-product-vision.md) to
   understand *what* we are building and *why*.
2. Then read [`docs/02-system-architecture.md`](docs/02-system-architecture.md)
   for the big-picture *how*.
3. Dip into the numbered docs for the area you care about.
4. [`docs/00-open-questions.md`](docs/00-open-questions.md) tracks every decision
   that is still open — this is updated as the plan is fleshed out.

## A note on code comments (important)

The person who owns this project is **not currently a programmer**. A deliberate
project rule is therefore:

> **Code in this repository is heavily commented in plain English.** Every
> meaningful section explains *what it does* and *why it exists*, so that a human
> (coder or not) can follow along later.

This is the opposite of "comment only when non-obvious." Here, we err on the side
of explaining things. See [`app/README.md`](app/README.md) for the commenting
standard.

---

*Status: planning / internal development. Last updated: 2026-06-26.*
