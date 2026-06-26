# 02 — System Architecture

This document describes the **major pieces** of the system and **how data flows**
between them. It stays at a conceptual level; specific technologies are in doc 03
and the data model is in doc 04.

## The pieces at a glance

```
                          ┌─────────────────────────────┐
                          │         The Writer          │
                          │      (in a web browser)     │
                          └──────────────┬──────────────┘
                                         │ HTTPS
                                         ▼
┌──────────────────────────────────────────────────────────────────────┐
│                          WEB APPLICATION                               │
│                                                                        │
│  ┌────────────────┐   ┌────────────────┐   ┌───────────────────────┐  │
│  │  Frontend UI   │   │  Backend API   │   │  Background Workers    │  │
│  │  (what the     │◄─►│  (auth, data,  │◄─►│  (long-running AI      │  │
│  │   writer sees) │   │   orchestration)│   │   extraction jobs)     │  │
│  └────────────────┘   └───────┬────────┘   └───────────┬───────────┘  │
│                               │                        │              │
└───────────────────────────────┼────────────────────────┼──────────────┘
                                │                        │
              ┌─────────────────┼────────────┐           │
              ▼                 ▼            ▼           ▼
     ┌────────────────┐ ┌──────────────┐ ┌────────┐ ┌──────────────────┐
     │  App Database  │ │ Object/Cache │ │  Cloud │ │   AI Provider    │
     │ (Postgres):    │ │  storage     │ │ Storage│ │  (Claude / GPT)  │
     │ accounts,      │ │ (temp files, │ │ (Google│ │  extraction +    │
     │ entities,      │ │  job queue)  │ │  Drive)│ │  cross-reference │
     │ facts, links   │ │              │ │ = the  │ │                  │
     │                │ │              │ │ writer's│ │                  │
     │                │ │              │ │ memory │ │                  │
     └────────────────┘ └──────────────┘ └────────┘ └──────────────────┘
```

## Component responsibilities

### 1. Frontend UI
What the writer interacts with in their browser.
- Sign in, connect cloud storage.
- Import/paste source text.
- Browse the knowledge base (entity files, hyperlinks).
- Review and resolve flagged contradictions.
- Configure which AI provider to use.

It talks **only** to the Backend API — never directly to the database or AI
providers. This keeps secrets (API keys, tokens) off the user's device.

### 2. Backend API
The brain that coordinates everything.
- Handles authentication and sessions.
- Reads/writes the application database.
- Talks to cloud storage on the writer's behalf (using their authorised token).
- Decides what work needs doing and hands long jobs to the workers.
- Enforces all security and access rules.

### 3. Background workers
Extraction is **slow** (large text + AI calls), so it must not block the web
request. When the writer submits text, the API creates a **job** and a worker
picks it up:
- Splits the text into manageable chunks.
- Calls the AI provider to extract entities and facts.
- Merges results into the knowledge base.
- Detects contradictions and records them for review.
- Updates the writer's cloud storage copy of the knowledge base.

This is the classic **job queue** pattern: the website stays responsive while
heavy work happens in the background, and the writer sees progress.

### 4. Application database (PostgreSQL)
The **structured, queryable** record of everything: accounts, projects,
entities, facts, the links between them, and the contradiction flags. Chosen
because the data is highly relational (entities link to facts link to source
locations link to other entities). Full schema in doc 04.

### 5. Object / cache storage
For things that don't belong in the relational database:
- Temporary uploads while a job is processing.
- The job queue itself (or a dedicated queue service).
- Caches to avoid re-doing expensive AI work.

### 6. Cloud storage (the writer's own — e.g. Google Drive)
The **permanent memory**. The writer's source documents and a serialised copy of
their knowledge base live here, in their own account. The app's own database is,
in effect, a fast, structured index over this permanent store. If the writer
disconnects, they keep everything. See doc 06.

### 7. AI provider (Claude / ChatGPT / others)
The extraction engine. Given a chunk of text and instructions, it returns
structured data: which entities appear, what facts about them, and where in the
text each fact came from. The provider is **pluggable** — see doc 07. Default
provider is **Anthropic Claude** (latest model).

## The primary data flow: "writer adds a chapter"

```
1. Writer pastes/imports text in the UI.
2. Frontend → Backend API: "here is new source text for project X".
3. API stores the source text (DB record + copy to cloud storage) and
   creates an extraction JOB. Responds immediately: "processing…".
4. A background WORKER picks up the job:
     a. Split text into chunks that fit the AI's context window.
     b. For each chunk, ask the AI: "what entities and facts are here,
        and where did each come from?"
     c. Match extracted entities against existing ones (is this a NEW
        character or one we already know?).
     d. For known entities, MERGE new facts.
        - If a new fact AGREES with or ADDS to what we know → store it.
        - If a new fact CONTRADICTS what we know → create a CONTRADICTION
          flag instead of overwriting. Never silently change a fact.
     e. Build/refresh cross-reference links between entities.
     f. Write the updated knowledge base back to cloud storage.
     g. Mark the job complete; record any contradictions for review.
5. Frontend shows: new/updated entities, and a review queue of any
   contradictions, each linking to the exact source location.
```

## Why this shape?

- **Separation of concerns.** UI, coordination, and heavy lifting are distinct,
  so each can be understood, changed, and scaled on its own.
- **Responsiveness.** Background jobs keep the site usable during slow AI work.
- **Security.** Secrets and tokens live only in the backend, never the browser.
- **Portability of the writer's data.** Cloud storage as permanent memory means
  the writer is never locked in.
- **Provider flexibility.** Swapping or adding an AI provider touches only the
  extraction layer.

## Trust boundaries (security-relevant)

Each arrow that crosses a boundary is a place we must validate and protect:

- **Browser ↔ Backend** — all over HTTPS; the backend authenticates every
  request and never trusts input from the browser without validation.
- **Backend ↔ AI provider** — outbound only; we send text + instructions, we
  receive structured data. Provider API keys live server-side only.
- **Backend ↔ Cloud storage** — uses the writer's OAuth token, stored encrypted,
  scoped to the minimum permissions needed (doc 06).

Details of how each boundary is protected are in doc 05.
