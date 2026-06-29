# CLAUDE.md — Attribution Engine (Phase 1 Proof)

> This file is the persistent instruction set for Claude Code working in this repository.
> Read it fully before writing any code. It defines what we are building, the hard
> constraints you must never violate, and how to behave. When in doubt, re-read this file.

---

## 1. Mission (the one thing)

Build an **on-premise Attribution Engine** that, for **one loan product line**, turns the
bank's **scanned loan documents** into an **immutable, attributed event ledger**, and lets
finance read **revenue by income stream, by product, and by tribe — live**, and **follow a
single loan from origination to the income stream it lands in**.

That is the whole job. It is the first stretch of a larger "process rail," but **none of the
larger vision is in scope here.** If a feature does not serve the sentence above, it does not
belong in this project.

The single most important user-facing outcome — the "money demo" — is:

> Load a batch of scanned loan documents → review/approve what was extracted →
> see total revenue split by income stream → click any one loan and see its complete
> thread, from the source document page to the recognized revenue and its income stream.

Build toward that demo relentlessly.

---

## 2. Hard constraints (NEVER violate these)

This is bank infrastructure handling PII. These are not guidelines — they are invariants.

1. **No external network calls, ever.** No cloud OCR (Textract, Vision, etc.), no cloud LLM
   APIs, no telemetry, no analytics, no CDN-loaded assets, no "phone home." Everything runs
   on the local host/network. Vendor all front-end assets locally. If you think you need an
   outbound call, stop and flag it instead.
2. **The event ledger is append-only.** Never `UPDATE` or `DELETE` a committed event. Corrections
   are new compensating events. The ledger is hash-chained (each event stores the previous
   event's hash) and must be verifiable.
3. **PII is sacred.** Scanned documents contain names, IDs, account numbers. Never log raw PII.
   Mask it in logs and error messages. Access is role-gated. Store the minimum needed.
4. **Humans approve before anything is committed.** Machine extraction is always a *draft*.
   No extracted data becomes a ledger event until a human reviewer approves it. This is a
   feature, not a limitation — it is the trust model.
5. **No agents, no autonomy in this phase.** No background actors taking actions on their own.
6. **Deterministic by default.** The system must work end-to-end with zero ML/LLM if needed
   (rules/template extraction). Any local LLM is an *optional enhancement*, never a hard dep.
7. **Reproducible & air-gap friendly.** Everything runs via `docker compose up`. Document an
   offline install path. No step may require internet at runtime.

If a requirement ever conflicts with one of these, the constraint wins. Ask the user.

---

## 3. Scope

### In scope
- Ingest scanned PDF loan documents (one product line, one or two document types to start).
- On-prem OCR of those PDFs.
- Field extraction (rules/template first; optional local LLM) → a draft extraction with
  per-field confidence and the location on the page it came from.
- Human review/correction UI; on approval, emit canonical events.
- Append-only, hash-chained event ledger with integrity verification.
- Attribution rules engine: map events → income streams (fee income, net interest income,
  etc.), product, tribe/channel, cost center.
- Revenue recognition + rollups.
- Read API + a minimal dashboard (revenue views + single-loan thread + source-doc view +
  ledger integrity badge).
- Role-based auth (admin / finance-reviewer / finance-viewer).
- Synthetic sample-document generator so the whole system can be built and tested **without
  any real PII files**.

### Explicitly OUT of scope (do not build, do not scaffold "for later")
- AI agents / autonomous actions.
- A swimlane / process designer.
- Writing back to any source system (read/ingest only).
- Multiple product lines (design the data model to allow it; implement only one).
- Real-time integration to core banking (ingest is document/batch based here).
- Multi-currency complexity beyond a single configurable currency.
- SSO / enterprise IdP integration (local auth is enough for the proof).

When tempted to build something out of scope, add it to `docs/BACKLOG.md` instead.

---

## 4. Tech stack (decided — do not substitute without asking)

- **Language:** Python 3.12
- **API:** FastAPI + Uvicorn
- **DB:** PostgreSQL 16 (via SQLAlchemy 2.x + Alembic migrations)
- **OCR:** Tesseract (`pytesseract`) + `pdf2image`/`pypdfium2` for rendering, `pdfplumber`
  for any text-layer PDFs. All local.
- **Optional extraction LLM:** Ollama running locally (localhost only), behind an interface,
  default OFF. Never required.
- **Web UI:** Server-rendered Jinja2 + htmx + vanilla JS. Charts via a **locally vendored**
  Chart.js. **No Node build step, no npm at runtime.** Keep it one service.
- **Auth:** Session cookies, password hashing with `argon2`. Roles enforced server-side.
- **Tests:** `pytest` + `httpx` test client. Aim for meaningful coverage of the ledger,
  attribution rules, and extraction mapping.
- **Packaging:** `docker compose` (app + postgres). `pyproject.toml` with pinned deps.
- **Lint/format:** `ruff` + `ruff format`. Type hints everywhere; `mypy` clean where practical.

---

## 5. Architecture (modules)

Keep the codebase as a clear, layered modular monolith. Suggested package layout:

```
app/
  main.py                # FastAPI app wiring
  config.py              # settings (env-driven, no secrets in code)
  db.py                  # engine/session
  models/                # SQLAlchemy models (Loan, Event, Document, Extraction, Rule, User, ...)
  ledger/                # append-only, hash-chained event store + verify
  ingest/                # pdf -> ocr -> text/layout
  extract/               # field extraction (rules impl + optional llm impl) behind an interface
  review/                # draft extraction review -> approve -> emit events
  attribution/           # rules engine, income-stream mapping, revenue recognition, rollups
  api/                   # REST routers
  web/                   # Jinja2 templates, htmx views, vendored static assets
  auth/                  # users, sessions, roles
  audit/                 # who/what/when audit log
fixtures/                # synthetic sample-document generator + seed data
tests/
docs/
```

**Data flow:** `PDF upload → OCR (ingest) → draft extraction (extract) → human review (review)
→ approved → canonical events appended (ledger) → mapped to income streams (attribution)
→ read via API/dashboard.`

---

## 6. Behavioral rules for you (Claude)

- **Work milestone by milestone** as defined in `BUILD_PLAN.md`. Do not jump ahead. Each
  milestone must end in a demoable, tested state before starting the next.
- **Write tests with the feature, not after.** The ledger and attribution math especially must
  be test-covered — these are the numbers a CEO will trust.
- **Keep it runnable at all times.** `docker compose up` should always work. Never leave the
  tree in a broken state between milestones.
- **Match the spec.** `SPEC.md` holds the data model, API surface, and pipeline detail. If you
  need to deviate, update `SPEC.md` in the same change and say why.
- **Small, well-described commits**, one logical step each.
- **Prefer boring, legible code** over clever abstractions. A bank engineer should be able to
  read any file and understand it. Comment the *why*, not the *what*.
- **Surface uncertainty.** If a requirement is ambiguous or you must guess at a document
  format / income-stream rule, write down the assumption in `docs/ASSUMPTIONS.md` and proceed
  with a sensible default — don't silently invent banking rules.
- **Never fabricate financial logic.** Income-stream mapping and revenue recognition are
  configurable rules, not hardcoded guesses. Make them data-driven and easy for finance to
  edit/review.

---

## 7. Definition of done (the proof is "done" when…)

1. `docker compose up` brings up the full system on a clean machine, no internet.
2. The synthetic generator can produce a batch of sample scanned loan PDFs.
3. Those PDFs can be uploaded, OCR'd, extracted, reviewed, corrected, and approved.
4. Approved data appears as hash-chained events in the ledger; `GET /ledger/verify` confirms
   integrity.
5. The dashboard shows revenue by income stream / product / tribe, and lets you open any
   single loan and trace it from the source document page to its recognized revenue and
   income stream.
6. Tests pass (`pytest`), lint passes (`ruff`), and `docs/DEMO.md` walks a non-technical
   person (your CEO) through the money demo in under 10 minutes.
7. A short `docs/SECURITY.md` documents: no egress, PII handling, append-only ledger, audit
   trail, and how to point ingestion at real documents.

---

## 8. Context: why this exists (read once, for judgment — not scope)

The bank runs on siloed "tribes," each with its own systems, so processes and data are
trapped and finance cannot trace a dollar from disbursement to income stream. This proof is
**Phase 1** of a plan to build a shared "rail" where processes run once and every step is
attributed — later enabling generated processes and, eventually, governed agents for routine
middle-office work. **You are building only Phase 1.** Knowing the bigger picture should help
you make the data model clean and forward-compatible — never to expand scope.
