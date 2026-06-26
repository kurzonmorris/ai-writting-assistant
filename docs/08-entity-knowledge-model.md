# 08 — Entity Knowledge Model

This document defines the **"things" we track** (entities), the **details we
capture** about each, and how they connect. It expands on the `Entity`, `Fact`,
`EntityAlias`, and `EntityLink` tables from doc 04.

## What is an entity?

An **entity** is any noteworthy thing in the writer's world that deserves its own
reference file. The writer mentioned it; the AI compiles everything known about
it into one place.

## Entity types (the starting set)

| Type | Examples | Notes |
|------|----------|-------|
| `CHARACTER` | people, named NPCs, deities | The richest type; see attributes below. |
| `LOCATION` | cities, regions, realms, rooms | Places at any scale. |
| `BUILDING` | a specific inn, castle, temple | A structure; often `LOCATED_IN` a location. |
| `ENVIRONMENT` | a forest, desert, climate | Natural settings and conditions. |
| `ITEM` | a sword, a letter, a key | Objects, often `OWNED` by a character. |
| `EQUIPMENT` | armour, gear, tools | Treated like items; may be its own subtype. |
| `CREATURE` | a dragon, a monster species | Non-human, often non-unique. |
| `ANIMAL` | a pet, a mount, a horse | Distinguished from creatures where useful. |
| `FACTION` | guilds, houses, armies, religions | Groups characters belong to. |
| `EVENT` | a battle, a festival, a coronation | Things that happened in the world. |
| `OTHER` | anything that doesn't fit yet | Catch-all; promote to a real type later. |

This list is **extensible** — new types can be added as needs emerge. The exact
final set is **OPEN** (doc 00); above is a sensible, broad starting point that
covers the examples the owner gave (characters, animals, pets, buildings, places,
environments, items, equipment).

## Attributes (the details we capture)

Each fact is an `attribute` + `value` pair tied to an entity (doc 04). Different
entity types naturally have different attributes. These lists are **guides for
the AI**, not rigid forms — the AI captures whatever the text supports, and we
can always add attributes.

### Character attributes (illustrative)
- **Identity:** names, aliases, titles, age, species/race, gender.
- **Appearance:** height, build, hair, **eye colour**, skin, distinguishing
  marks, typical **clothing**.
- **Personality:** traits, temperament, values, fears, motivations, quirks.
- **Voice & speech:** accent, vocabulary, catchphrases, **how they talk**
  (formal, terse, rambling), verbal tics.
- **Background:** origin, family, history, occupation, skills.
- **State & arc:** current location, mood, injuries, status (alive/dead),
  changes over time.
- **Relationships:** allies, enemies, family, romantic ties (also captured as
  `EntityLink`s).
- **Possessions:** items owned (also `EntityLink`s).

### Location / building / environment attributes
- Name(s), type, size/scale, geography, climate, notable features.
- Who/what is found there; ruling faction; history; current condition.
- Containment: what it's part of, what's inside it.

### Item / equipment attributes
- Name(s), description, material, function, history/provenance.
- Owner(s) over time; current location; special properties.

### Creature / animal attributes
- Species/type, appearance, behaviour, abilities, habitat.
- For a named individual (e.g. a specific pet): owner, personality, history.

### Faction attributes
- Name(s), purpose, structure, leaders, members, allies/enemies, territory.

### Event attributes
- What happened, when, where, who was involved, causes and consequences.

> **Time / chronology is cross-cutting.** Many attributes change over the story
> ("current location", "status"). How we represent change over time — e.g.
> tying facts to a chapter/timeline so "as of chapter 10 he is in the capital" —
> is an important design question marked **OPEN** in doc 00.

## Aliases (one entity, many names)

Characters especially are referred to many ways: "Will", "William", "the
captain", "her brother". `EntityAlias` (doc 04) records these so all mentions
resolve to a single entity and its one reference file. The AI proposes aliases;
uncertain ones can be reviewed by the writer.

## Relationships / cross-references (the hyperlink graph)

`EntityLink` (doc 04) records typed relationships between entities. A starting
vocabulary of relationship types:

| Relationship | Example |
|--------------|---------|
| `OWNS` | A character owns an item. |
| `LOCATED_IN` | A building is in a city. |
| `MEMBER_OF` | A character belongs to a faction. |
| `RELATED_TO` | Family / personal ties between characters. |
| `ALLY_OF` / `ENEMY_OF` | Political or personal alignment. |
| `PARTICIPATED_IN` | A character took part in an event. |
| `MENTIONS` | A general "these two are referenced together" link. |

Every link cites the `TextSpan` where the relationship was observed, so the
writer can see *why* the link exists and jump to that passage. These links are
what turn the knowledge base into a navigable, hyperlinked wiki.

## What an entity "file" looks like to the writer

In the UI (and in the serialised cloud-storage copy), an entity reads like a
clean wiki page:

```
WILLIAM ("Will", "the captain")            [CHARACTER]
Summary: A weathered sea captain, terse and loyal...

Appearance
  • Eye colour: blue        (chapter 2 — “his pale blue eyes…”)
  • Clothing: long coat     (chapter 5 — “the captain’s worn coat…”)

Speech
  • Terse, clipped sentences (chapter 2)

Relationships
  • Owns → The Sea Dagger [ITEM]
  • Member of → The Harbour Guild [FACTION]

Appears in: Chapter 2, Chapter 5, Chapter 9
```

Each fact shows its **source**; each linked entity name is a **hyperlink** to
that entity's page. Anything disputed is marked and routed to review (doc 09).

## Extensibility

- New **entity types** and **attributes** can be added without restructuring,
  because facts are flexible `attribute`/`value` pairs rather than fixed columns.
- New **relationship types** can be added to the `EntityLink` vocabulary.
- This flexibility is intentional: every writer's world is different, and we don't
  want to force one rigid template (doc 01 value: the writer is in control).
