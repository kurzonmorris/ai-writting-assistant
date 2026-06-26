# `server/auth/` — Authentication & Sessions

Handles **who the writer is** and keeps them signed in securely. We use a managed
library (Auth.js) rather than rolling our own — hand-built auth is a leading
cause of breaches (see `../../../../docs/05-security-and-privacy.md`).

## Responsibilities

- "Sign in with Google" (OAuth). No passwords for us to store.
- Issue and validate secure, http-only session cookies.
- Expose a reliable "who is the current user?" check that every API route uses
  to scope data to that user (and only that user).

## Important notes

- The same Google OAuth app is used for **sign-in** and for **connecting Google
  Drive** (doc 06), but Drive *file* permissions are requested separately and
  stored as a `StorageConnection` (doc 04) — not mixed into the login session.
- Session secrets and OAuth credentials come from environment variables
  (`AUTH_SECRET`, `GOOGLE_CLIENT_ID`, `GOOGLE_CLIENT_SECRET`).

## Status

Plan only. Implemented early in Phase 1 (doc 11).
