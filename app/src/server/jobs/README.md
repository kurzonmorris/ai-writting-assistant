# `server/jobs/` — Background Jobs & Worker

Slow work — especially AI extraction — must not run inside a web request, or the
writer's browser would freeze. This folder holds the **job queue** and the
**worker** that processes jobs in the background (see
`../../../../docs/02-system-architecture.md` and `../../../../docs/07-ai-extraction-pipeline.md`).

## How it works

1. When the writer submits text, an API route creates an **extraction job**
   (recorded in the `ExtractionJob` table, doc 04) and responds immediately with
   "processing…".
2. The job is placed on the queue (BullMQ backed by Redis).
3. The **worker** (a separate process, started with `npm run worker`) picks up the
   job and runs the extraction pipeline (`../extraction/`).
4. Job status is updated (`QUEUED` → `RUNNING` → `SUCCEEDED`/`FAILED`) so the UI
   can show progress and failures can be retried safely.

## Why a separate worker process

Keeping extraction in its own process means heavy AI work never slows down the
website, and we can scale the workers independently if volume grows.

## Rules

- Every job records provenance and usage on its `ExtractionJob` row (doc 07).
- Failures are captured (`error` field) and surfaced for retry; we never lose a
  submission silently.
- Redis connection comes from `REDIS_URL` (environment only).

## Status

Plan only. Introduced in Phase 1 so extraction is non-blocking from the start
(doc 11).
