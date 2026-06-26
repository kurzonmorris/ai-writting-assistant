# 12 — Glossary

Plain-English definitions of every term used across these documents. If a term in
another doc is unclear, it should be explained here.

| Term | Meaning |
|------|---------|
| **Writer / user** | The person using the website to manage their fiction. |
| **Project** | One body of work — a novel, a campaign, a game design bible. |
| **Source text / source document** | The prose the writer imports or pastes in. The original words. |
| **Knowledge base** | The collection of compiled entity files — the "story bible" the AI builds. |
| **Entity** | A tracked "thing" with its own reference file: a character, place, item, creature, etc. |
| **Attribute** | A category of detail about an entity (e.g. eye colour, occupation, speech style). |
| **Fact** | A single statement about an entity (attribute + value), always tied to a source location. |
| **Alias** | An alternative name for an entity (e.g. "Will" for "William"). |
| **Cross-reference / link** | A connection between two entities (e.g. a character *owns* an item), shown as a hyperlink. |
| **Text span** | An exact location (character range) within a source document; what makes facts traceable. |
| **Contradiction** | Two facts about the same entity/attribute that can't both be true; flagged for the writer. |
| **Canon** | The version of a fact the writer has decided is true. |
| **Extraction** | The process of reading text and pulling out entities and facts. |
| **Extraction pipeline** | The full step-by-step process from raw text to compiled knowledge (doc 07). |
| **Provider** | An AI service that does the extraction (Anthropic Claude, OpenAI, etc.). |
| **Model** | A specific AI within a provider (e.g. a particular Claude model). |
| **Background job / worker** | Slow work (like extraction) run outside the web request so the site stays responsive. |
| **Job queue** | The mechanism that hands background work to workers and tracks its status. |
| **Cloud storage / permanent memory** | The writer's own storage (e.g. Google Drive) where their files durably live. |
| **OAuth** | The standard way to grant an app limited access to your account without sharing your password. |
| **Token** | The credential OAuth gives the app to act on the writer's behalf, within granted permissions. |
| **Scope** | The specific permissions a token grants (we request the minimum needed). |
| **Least privilege** | The security principle of granting only the minimum access required. |
| **Encryption at rest** | Storing sensitive data (like tokens) in encrypted form so a leak of the database isn't a leak of secrets. |
| **Prompt injection** | An attack where malicious instructions hidden in input try to make the AI misbehave. |
| **ORM** | "Object-Relational Mapper" — a tool (we use Prisma) that lets code work with the database safely and readably. |
| **Migration** | A controlled, reviewable change to the database structure over time. |
| **Soft delete** | Marking a record as deleted (so it can be recovered) instead of erasing it. |
| **UUID** | A long random identifier used as a primary key; doesn't leak counts and is safe in URLs. |
| **RLS (Row-Level Security)** | A database feature that stops one user's queries from ever returning another user's rows. |
| **Content hash** | A fingerprint of a document used to detect whether it changed, so we don't reprocess unchanged text. |
| **MVP** | "Minimum Viable Product" — the smallest version that's genuinely useful. |
| **OPEN** | A decision not yet made; tracked in doc 00. |
