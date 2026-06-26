# `server/db/` — Database Access

The **only** place in the app that talks directly to PostgreSQL. Everything goes
through Prisma, using the schema defined in
[`../../../prisma/schema.prisma`](../../../prisma/schema.prisma) and explained in
[`../../../../docs/04-database-design.md`](../../../../docs/04-database-design.md).

## Responsibilities

- Create and share a single Prisma client connection for the app.
- Provide small, well-named functions for the queries the app needs (e.g.
  "get all entities for a project", "create a contradiction").
- Ensure **every** query is scoped to the signed-in user's data (ownerId / project
  ownership), so one writer can never read another's work (doc 05).

## Rules

- Centralising DB access here means access-control and soft-delete rules
  (ignore `deletedAt` rows) are applied consistently in one place.
- The connection string comes from `DATABASE_URL` (environment only).
- Structural changes happen through Prisma **migrations**, never by editing the
  database by hand (doc 04).

## Status

Plan only. The Prisma schema is already written; query helpers arrive in Phase 1.
