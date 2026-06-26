# The Application (`app/`)

This folder will hold **the website itself**. Right now it is a **documented
scaffold**: the folder structure and the key configuration/design files exist,
with READMEs explaining what each part is *for*, so the plan is concrete before we
write feature code. Actual feature code is added phase by phase (see
[`../docs/11-development-roadmap.md`](../docs/11-development-roadmap.md)).

## The commenting standard (read this first)

The owner of this project is not currently a programmer and wants to be able to
follow the code. Therefore, **the normal "comment only when non-obvious" rule
does NOT apply here.** Instead:

> **Every meaningful section of code carries a plain-English comment explaining
> what it does and why it exists.** Files begin with a short header describing
> the file's purpose. We err on the side of *over*-explaining.

Concretely:
- Each source file starts with a comment block: what this file is, why it exists.
- Each function has a comment in plain English describing its job.
- Non-obvious steps inside functions get a one-line explanation.
- We prefer clear, boring code over clever code.

This is a deliberate, documented choice specific to this project.

## Planned technology (see `../docs/03-technology-stack.md`)

- **TypeScript** — one language for the whole app.
- **Next.js (React)** — the web framework (pages + server/API in one project).
- **PostgreSQL + Prisma** — the database and a readable way to talk to it.
- **Auth.js** — "Sign in with Google" and secure sessions.
- **A job queue (BullMQ + Redis)** — runs slow AI extraction in the background.

## Planned folder structure

```
app/
├── README.md            ← you are here
├── .env.example         ← every config value the app needs (names only)
├── package.json         ← planned dependencies and scripts
├── prisma/
│   └── schema.prisma    ← the database design, as runnable schema (doc 04)
└── src/
    ├── README.md
    ├── app/             ← Next.js pages & routes (what the writer sees)
    ├── server/          ← backend logic the browser never sees
    │   ├── auth/        ← sign-in & session handling (doc 05)
    │   ├── db/          ← database access via Prisma (doc 04)
    │   ├── storage/     ← cloud-storage integration, e.g. Google Drive (doc 06)
    │   ├── ai/          ← AI provider integrations (pluggable) (doc 07)
    │   ├── extraction/  ← the extraction pipeline & merge logic (doc 07, 09)
    │   └── jobs/        ← background job queue & workers (doc 02, 07)
    └── shared/          ← types & helpers used by both frontend and backend
```

Each of these folders contains its own `README.md` describing its responsibility,
so a newcomer can navigate by reading folder by folder.

## How to run it (placeholder)

Setup instructions will be filled in when Phase 1 lands. Defaults will target the
owner's environment (Windows VM / Unraid as noted in their profile) and use
Docker to run PostgreSQL and Redis locally without manual installation.

## Status

Scaffold and design only. No feature code yet. Building begins once the Priority 1
open questions (`../docs/00-open-questions.md`) are answered.
