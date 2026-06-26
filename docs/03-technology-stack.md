# 03 — Technology Stack

This document lists the technologies we plan to use and, importantly, **why each
was chosen**. Choices favour: (1) being mainstream and exceptionally
well-documented (so a non-coder can find help), (2) using **one main language**
where possible (less to learn), and (3) strong security defaults.

> Status: these are **recommended defaults**, not locked in. Anything the owner
> wants to change is tracked in doc 00. Nothing here is hard to swap later.

## Summary table

| Layer | Choice | Why |
|-------|--------|-----|
| Language | **TypeScript** | One language across frontend and backend. Catches mistakes before running. Huge community. |
| Web framework | **Next.js (React)** | Frontend + backend in one well-documented framework. Industry standard. |
| UI styling | **Tailwind CSS** | Readable, consistent styling without a separate CSS skill. |
| Database | **PostgreSQL** | Rock-solid relational database; perfect for cross-referenced data. |
| Database access | **Prisma (ORM)** | Define the schema once in plain, readable form; type-safe queries. |
| Authentication | **Auth.js (NextAuth)** | Handles "Sign in with Google" and sessions securely, so we don't roll our own. |
| Background jobs | **A job queue (e.g. BullMQ + Redis)** | Run slow AI extraction without freezing the site. |
| AI provider (default) | **Anthropic Claude (latest model)** | Strong long-context extraction; default provider. Pluggable (doc 07). |
| Cloud storage (first) | **Google Drive API** | The first "permanent memory" integration (doc 06). |
| Hosting | **OPEN** | Decided later; see doc 00. |

## Why TypeScript everywhere

The owner is not a coder yet and wants understandable code. Using **one
language** for both the part the writer sees (frontend) and the part on the
server (backend) means:

- Half as much to learn or read.
- The same data shapes are reused on both sides, reducing bugs.
- TypeScript flags many mistakes *before* the code ever runs, which is safer.

(An alternative was a Python backend — strong for AI work — but mixing two
languages adds complexity for a solo, non-coder owner. We call the AI providers
over their web APIs, so we don't need Python for that. Revisit only if a future
need genuinely requires it.)

## Why Next.js

Next.js lets us build the website's pages **and** its server logic (the API) in
one project. It is one of the most widely used web frameworks, so:

- There is abundant, beginner-friendly documentation and community help.
- Security best practices (CSRF protection, secure headers, etc.) are
  well-trodden ground.
- It scales from a tiny internal test app to a real product without rewrites.

## Why PostgreSQL + Prisma

Our data is **deeply interconnected**: entities link to facts, facts link to
source locations, entities link to other entities. A relational database is the
natural fit, and PostgreSQL is the most respected open-source choice.

**Prisma** sits on top of PostgreSQL and lets us describe the entire database in
**one readable file** (`app/prisma/schema.prisma`). It then:

- Generates type-safe code so the app can't accidentally read a column that
  doesn't exist.
- Manages **migrations** (controlled, reviewable changes to the database
  structure over time).

This keeps the database design legible to a human — see doc 04.

## Why a managed auth library (Auth.js)

Authentication is one of the easiest things to get dangerously wrong. We will
**not** write our own password handling. Auth.js:

- Provides "Sign in with Google" out of the box (which also pairs naturally with
  connecting Google Drive).
- Manages secure sessions and tokens for us, following current best practice.

See doc 05 for the full security rationale.

## Why a background job queue

AI extraction over a whole chapter can take from seconds to minutes. If that ran
inside a normal web request, the writer's browser would hang. A **job queue**
(such as BullMQ backed by Redis) lets the website hand the work to a background
worker and immediately tell the writer "processing…", then show results when
done. This is the standard, reliable pattern for slow work. See doc 02 and 07.

## Why Anthropic Claude as the default AI provider

- Strong at **long-context reading and structured extraction**, which is exactly
  this product's job.
- Good at returning **structured output** (JSON) with source attribution, which
  we rely on to keep every fact traceable.
- The architecture keeps providers **pluggable** (doc 07), so the writer can
  choose Claude, ChatGPT, or another, and we can add providers without rework.

We default to the latest, most capable Claude model and make the specific model
a configurable setting rather than hard-coding it.

## Things deliberately left OPEN

These do not block design work and are tracked in doc 00:

- **Hosting / deployment platform** — depends on budget and scale expectations.
- **Which cloud storage providers beyond Google Drive** (Dropbox, OneDrive…).
- **Whether to support self-hosting** for privacy-conscious writers.
- **Email/notification provider** for "your extraction is done" messages.

## Local development tooling (planned)

So the project is reproducible on any machine:

- **Docker** to run PostgreSQL and Redis locally without manual installs.
- **`.env` files** for configuration (never committed — see `.env.example`).
- **A linter + formatter** (ESLint + Prettier) to keep code tidy and consistent.
