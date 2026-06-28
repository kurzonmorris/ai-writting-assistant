# 00 — Open Questions

A **living checklist** of decisions still to be made. As each is answered, record
the decision here (and update the relevant doc). This is where we capture
everything that isn't settled yet, so nothing gets silently assumed.

Status key: 🔴 needs a decision · 🟡 leaning a way · 🟢 decided

---

## Priority 1 — needed to start building

### Q1. Which AI provider(s) and model to start with? 🟢
- **DECIDED (2026-06-26): Bring your own key.** Each writer supplies their own AI
  API key; the app does not pay for usage. Default provider remains Anthropic
  Claude, kept provider-pluggable. (Implication: signup flow must guide a
  non-technical writer through getting and pasting in an API key — see new Q16.)
- Related: doc 03, doc 07, doc 05.

### Q2. Cloud storage scope vs. convenience? 🟢
- **DECIDED (2026-06-26): App-folder only (private).** The app can only see files
  it created in its own folder, never the rest of the writer's Drive. Writers
  paste text in rather than importing arbitrary existing Drive files.
- Related: doc 06, doc 05.

### Q3. Phase 1 scope — confirm the smallest first slice? 🟢
- **DECIDED (2026-06-26): Core loop, no Drive in Phase 1.** Sign in → create
  project → paste text → background extraction with Claude → browse sourced
  entities. Cloud storage comes later (Phase 4).
- Related: doc 11.

### Q4. Hosting / where it runs? 🟢
- **DECIDED (2026-06-26): Managed cloud hosting.** Build to run locally during
  internal development, deploy to a managed cloud host. Specific platform TBD.
- Related: doc 03.

---

## Priority 2 — needed soon, not blocking

### Q5. How to model change over time (chronology)? 🟢 (with a refinement open)
- **DECIDED (2026-06-26): track a timeline.** Each fact records "true as of
  chapter N" (`Fact.timelineOrder`). A later value for a time-varying attribute
  makes the earlier fact `HISTORICAL` (kept as history, not an error); only facts
  that disagree at the *same* point in time are contradictions.
- **Still open:** explicit story-time for flashbacks / non-linear narratives
  (default uses document/chapter order). Tracked as a refinement.
- Related: doc 08, doc 09, doc 07, doc 04.

### Q6. Knowledge-base file format in cloud storage? 🟡
- Leaning **Markdown** (readable, portable, supports links). Confirm, or prefer
  another format?
- Related: doc 06.

### Q7. How aggressive should entity auto-merging be? 🟢
- **DECIDED (2026-06-26): ask the writer when unsure.** Auto-merge only when
  confident; otherwise prompt "are these the same?". Over-merging is worse than a
  duplicate. (Exact confidence threshold + review UX still to be tuned.)
- Related: doc 07, doc 08.

### Q8. How sensitive should contradiction detection be? 🟢
- **DECIDED (2026-06-26): adjustable per project, changeable on the fly** via a
  slider on the page (`Project.contradictionSensitivity`, default = balanced).
- Related: doc 09, doc 04.

---

## Priority 3 — later / product direction

### Q9. Naming & branding? 🔴
- No official name yet; "AI Writing Assistant" is a working title. Visual design
  partly waits on this.
- Related: doc 10.

### Q10. Single-writer only, or collaboration later? 🟡
- Designed single-writer first. Is sharing/collaboration wanted down the line?
- Related: doc 01, doc 11.

### Q11. Other storage providers (Dropbox, OneDrive)? 🔴
- Google Drive first. Which others, if any, and when?
- Related: doc 06, doc 11.

### Q12. Authentication options beyond Google? 🔴
- Google sign-in first. Also offer email/password or other providers?
- Related: doc 05.

### Q16. Onboarding writers to "bring your own key"? 🔴
- Consequence of Q1. A non-technical writer needs a clear, hand-held flow to
  create an AI provider account, generate an API key, and paste it in safely
  (and the key must be encrypted at rest like other secrets — doc 05).
- Decide: which provider(s) to support keys for at launch; how much in-app
  guidance/screenshots; what happens if a key is invalid or out of credit.
- Related: doc 05, doc 07.

### Q13. Data retention & deletion specifics? 🔴
- Exactly what the app keeps, for how long, and what "delete my account" removes
  (including backups). Needed before any non-internal use + privacy policy.
- Related: doc 05, doc 04.

### Q14. Source-text editing — in-app or in the writer's own tools? 🔴
- For "edit the source" during contradiction resolution: do we build in-app
  editing, or send the writer to edit in their own tool and re-import?
- Related: doc 09.

### Q15. Which entity types and attributes matter most to the owner? 🟢
- **DECIDED (2026-06-26): all matter, in this priority order** — characters,
  places, environments, factions, equipment, items (most important first). Used
  to prioritise extraction and order the navigator. Other types (creatures,
  animals, buildings, events) still tracked.
- Related: doc 08.

---

## Decisions log (record answers here as they're made)

| Date | Question | Decision |
|------|----------|----------|
| 2026-06-26 | Default AI provider | Anthropic Claude, kept pluggable. |
| 2026-06-26 | Primary language/framework | TypeScript + Next.js (pending owner confirmation). |
| 2026-06-26 | Database | PostgreSQL + Prisma (pending owner confirmation). |
| 2026-06-26 | AI usage cost (Q1) | Bring your own key — writers supply their own AI API key. |
| 2026-06-26 | Cloud storage scope (Q2) | App-folder only (most private). |
| 2026-06-26 | Phase 1 scope (Q3) | Core loop, no cloud storage until Phase 4. |
| 2026-06-26 | Hosting (Q4) | Managed cloud hosting. |
| 2026-06-26 | Change over time (Q5) | Track a timeline ("true as of chapter N"); changes become history, not conflicts. |
| 2026-06-26 | Entity merging (Q7) | Ask the writer when unsure; auto-merge only when confident. |
| 2026-06-26 | Contradiction sensitivity (Q8) | Adjustable per project via an on-the-fly slider; default balanced. |
| 2026-06-26 | Entity priorities (Q15) | Order: characters, places, environments, factions, equipment, items. |
