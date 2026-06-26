# 05 — Security & Privacy

A writer's unpublished fiction is **highly sensitive**. It may be years of work,
unannounced, and personally important. Security and privacy are therefore core
product features, not afterthoughts. This document records the practices we
commit to.

## Threats we are protecting against

- **Account takeover** — someone gaining access to a writer's account.
- **Token theft** — an attacker stealing the writer's cloud-storage access and
  reading/altering their files.
- **Data leakage between users** — one writer ever seeing another's work.
- **Leaked secrets** — API keys or tokens ending up in the code repo or logs.
- **Injection attacks** — malicious input via the web forms or via text content.
- **Prompt injection** — malicious instructions hidden inside source text trying
  to make the AI misbehave (specific to AI products; see below).

## Authentication (proving who you are)

- We use a **managed authentication library (Auth.js)** rather than writing our
  own. Hand-rolled auth is a leading source of breaches.
- Primary sign-in is **"Sign in with Google"** (OAuth), which also pairs cleanly
  with connecting Google Drive. No passwords for us to store or leak.
- Sessions use **secure, http-only cookies** (not readable by browser
  JavaScript), with appropriate `SameSite` settings to resist CSRF.
- **OPEN:** whether to also offer email/password or other providers (doc 00).

## Authorisation (what you're allowed to do)

- Every user-owned row carries an `ownerId` (doc 04). Every query is scoped to
  the signed-in user. A writer can only ever read or change their own data.
- We plan to back this with **PostgreSQL Row-Level Security (RLS)** as a second
  line of defence, so even a bug in app code cannot cross accounts.
- The backend authorises **every** request. The frontend never decides access.

## Protecting cloud-storage tokens

This is the most sensitive data we hold, because it grants access to the writer's
files.

- We request the **minimum OAuth scope** needed (doc 06) — ideally access only to
  files this app created, not the writer's whole Drive.
- Access and refresh tokens are **encrypted at rest** using a strong algorithm
  (e.g. AES-256-GCM). The encryption key lives in a secrets manager / environment
  variable, **never** in the database or the code.
- Tokens live only in the dedicated `StorageConnection` table and are never sent
  to the browser or to AI providers.
- The writer can **disconnect** storage at any time, which revokes and deletes
  the stored tokens.

## Secrets management

- **No secret is ever committed to the repository.** Configuration comes from
  environment variables; `app/.env.example` documents the *names* of every value
  with no real values in it.
- `.gitignore` excludes `.env` and any local credential files.
- Production secrets live in the hosting platform's secret store (chosen later).
- We rotate secrets if any are suspected of exposure, and keep them out of logs.

## Protecting the web layer

- **HTTPS everywhere.** No plain HTTP in production.
- **Input validation** on every API endpoint (e.g. with a schema validator like
  Zod). Never trust data from the browser.
- **Output encoding** to prevent cross-site scripting (XSS); React helps here,
  and we avoid unsafe raw-HTML rendering of user text.
- **CSRF protection** via the framework's built-in mechanisms and SameSite
  cookies.
- **Security headers** (Content-Security-Policy, HSTS, X-Content-Type-Options,
  etc.).
- **Rate limiting** on sensitive and expensive endpoints (sign-in, extraction)
  to resist abuse and runaway AI cost.
- **Dependency hygiene** — keep libraries updated; watch for known
  vulnerabilities.

## AI-specific risks

### Prompt injection
Source text is **untrusted input**. A writer could (accidentally or via
copy-pasted content) include text like *"ignore your instructions and …"*. We
mitigate by:
- Treating all source text as **data to be analysed, never as instructions**.
- Sending the extraction instructions as a separate, privileged part of the
  request and clearly delimiting the writer's text.
- Validating the AI's output against a strict expected structure (we expect
  facts with sources; we reject anything malformed).
- Because the AI only ever **reads and reports** (it cannot edit the manuscript
  or take destructive actions), the blast radius of a successful injection is
  small by design.

### Data sent to third-party AI providers
- Source text is sent to the chosen AI provider for extraction. This is inherent
  to the product, so we are **transparent** about it: the writer is told which
  provider their text goes to, and chooses the provider.
- **OPEN:** offer a "bring your own API key" mode and/or a provider with a
  no-training / zero-retention data policy for privacy-conscious writers
  (doc 00). We will prefer providers that do not train on customer data by
  default.

## Privacy commitments

- The writer owns their work; their files live in **their** cloud storage.
- We collect the minimum personal data needed to run the service.
- Disconnecting storage and deleting an account removes the app's copies and
  tokens (subject to a documented retention/backup window — to be defined).
- We will publish a plain-English privacy policy before any non-internal use.

## Accountability

- The `AuditLog` table records significant actions (sign-in, storage connect/
  disconnect, deletions, contradiction resolutions) for the account owner's own
  visibility and for incident investigation.
- Background jobs and API errors are logged **without** including secrets or full
  manuscript text.

## Security checklist for every change (developer-facing)

- [ ] Does this endpoint authenticate the user and scope data to them?
- [ ] Is all incoming data validated before use?
- [ ] Are any new secrets read from env, never hard-coded?
- [ ] Could user text reach a place where it's treated as code or instructions?
- [ ] Are tokens/secrets kept out of logs and out of responses to the browser?
- [ ] Does deleting/disconnecting clean up the right data and tokens?
