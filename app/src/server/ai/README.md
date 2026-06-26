# `server/ai/` — AI Provider Integrations

Talks to the AI services that do the actual extraction (see
`../../../../docs/07-ai-extraction-pipeline.md`). Providers are **pluggable**:
the rest of the app uses one interface and doesn't care which provider is behind
it.

## The interface

```
extract(text, instructions) → structured facts with their source quotes
```

The caller (the extraction pipeline) gives a chunk of text plus the extraction
instructions; the provider returns machine-readable facts, each with the exact
phrase it came from. Output is validated against a strict schema; anything
malformed is rejected (doc 07).

## Implementations

- **`anthropic` (default)** — Anthropic Claude, latest model. Strong long-context,
  structured extraction (doc 03).
- **`openai` (optional)** — ChatGPT, if enabled.
- More can be added by writing another implementation of the same interface.

## Rules

- The model is a **setting** (`ANTHROPIC_MODEL`), never hard-coded, so we can
  adopt newer models easily.
- **Writers bring their own API key** (doc 00, Q1). The key is loaded from the
  writer's encrypted `ProviderCredential` (doc 04), decrypted in memory here, and
  stays server-side only (doc 05). A `DEV_*` env key exists for local testing only.
- The writer's text is sent as **data to analyse, never as instructions**
  (prompt-injection defence — doc 05/07).
- **Still OPEN (Q16):** how to onboard a non-technical writer through getting and
  entering their key.

## Status

Plan only. The default Anthropic provider is implemented in Phase 1 (doc 11).
