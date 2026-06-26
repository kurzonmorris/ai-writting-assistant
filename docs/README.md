# Design Documents

This folder holds **every design decision** for the AI Writing Assistant. The
goal is that anyone — including someone who cannot code — can read these in order
and understand exactly what we are building, how, and why.

Each document is numbered so the reading order is obvious. Documents are living:
they will be updated as decisions are made. When a decision changes, update the
relevant doc *and* note it in `00-open-questions.md` if it resolves an open item.

## Reading order

| # | Document | What it covers |
|---|----------|----------------|
| 00 | [Open Questions](00-open-questions.md) | Decisions still to be made. Living checklist. |
| 01 | [Product Vision](01-product-vision.md) | What the product is, who it's for, the guiding principles. |
| 02 | [System Architecture](02-system-architecture.md) | The major pieces and how data flows between them. |
| 03 | [Technology Stack](03-technology-stack.md) | Which technologies we use and *why* each was chosen. |
| 04 | [Database Design](04-database-design.md) | The data model, tables, relationships, and DB best practices. |
| 05 | [Security & Privacy](05-security-and-privacy.md) | How we protect accounts, tokens, and a writer's work. |
| 06 | [Cloud Storage Integration](06-cloud-storage-integration.md) | How Google Drive (and others) act as permanent memory. |
| 07 | [AI Extraction Pipeline](07-ai-extraction-pipeline.md) | How text becomes structured knowledge, step by step. |
| 08 | [Entity Knowledge Model](08-entity-knowledge-model.md) | The "things" we track (characters, places…) and their fields. |
| 09 | [Contradiction Detection & Review](09-contradiction-detection-and-review.md) | How conflicts are found, flagged, and resolved. |
| 10 | [User Interface & Flows](10-user-interface-and-flows.md) | The screens and the journeys through them. |
| 11 | [Development Roadmap](11-development-roadmap.md) | The phased plan from MVP to full product. |
| 12 | [Glossary](12-glossary.md) | Plain-English definitions of every term we use. |

## Conventions used across these docs

- **Writer / user** — the person using the website to manage their fiction.
- **Entity** — any tracked "thing" the AI compiles a file for (a character, a
  place, an item, etc.). Defined fully in doc 08.
- **Source text** — the prose the writer imports or pastes in.
- **Knowledge base** — the collection of entity files the AI builds.
- **Fact** — a single extracted statement about an entity, always traceable back
  to a location in the source text.
- **Provider** — an AI service (Anthropic Claude, OpenAI, etc.).

Anything that is not yet decided is marked with **OPEN** and tracked in doc 00.
