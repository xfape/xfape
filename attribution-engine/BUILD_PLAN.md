# BUILD_PLAN.md — milestone-by-milestone build order

Work these in order. **Each milestone must end runnable, tested, and demoable** before you
start the next. Do not scaffold future milestones early. After each milestone, update
`README.md` and append a one-line entry to `docs/CHANGELOG.md`.

---

## ▶ Kickoff prompt (paste this as your first message to Claude Code in the new project)

> Read `CLAUDE.md`, `SPEC.md`, and `BUILD_PLAN.md` in full before doing anything. Confirm
> back to me, in 5 bullets, your understanding of (1) the one thing we're building, (2) the
> hard constraints, (3) what's explicitly out of scope, (4) the tech stack, and (5) the
> "money demo" we're driving toward. Then begin **Milestone 0** only, and stop for my review
> when its acceptance criteria are met. Do not proceed past a milestone without my OK. Ask me
> about any banking rule (income-stream mapping, revenue recognition) you'd otherwise have to
> guess — record assumptions in `docs/ASSUMPTIONS.md`.

---

## Milestone 0 — Scaffold & skeleton
**Goal:** an empty-but-real, runnable system.
- `pyproject.toml` (pinned deps), `ruff` config, `pytest` config.
- `docker compose` with `app` + `db`; Tesseract in the app image.
- FastAPI app with `/health`, settings via env, `.env.example`, SQLAlchemy engine, Alembic init.
- Auth skeleton: `User` model, login/logout, password hashing (argon2), role enum, a seeded
  admin user, role-checking dependency.
- `make up/down/test/lint` (or scripts). Jinja2 base layout + vendored static dir (empty ok).
**Done when:** `docker compose up` serves `/health`; you can log in as admin; `pytest` and
`ruff` pass; nothing reaches the internet.

## Milestone 1 — Append-only hash-chained ledger
**Goal:** the trustworthy core.
- `Loan` + `Event` models per `SPEC.md`. Append-only enforcement (no update/delete paths;
  DB-level guard if practical).
- Hash chain: each event stores `prev_hash` and `hash` over a canonical serialization.
- `append_event()` service, `verify_chain()` service, `GET /ledger/verify`.
- Thorough tests: ordering, hash continuity, tamper detection, compensating events.
**Done when:** events can be appended only via the service, the chain verifies, and a
deliberately tampered row is detected by `/ledger/verify`. Driven by unit tests (no UI yet).

## Milestone 2 — Attribution rules & revenue recognition
**Goal:** the numbers.
- `IncomeStream`, `AttributionRule` models; data-driven rules engine (priority match → assign).
- Revenue recognition for fee income and interest income, with the rule documented in ONE place.
- Rollup query/service + `GET /attribution/revenue?by=income_stream|product|tribe`.
- `GET /loans/{id}` returning the full thread + per-income-stream revenue.
- Seed sample rules. Tests: mapping correctness, unmatched-event flagging, rollup totals,
  recognition math.
**Done when:** feeding synthetic events through the engine yields correct revenue rollups and
a correct single-loan thread, all test-verified. Still no documents/UI.

## Milestone 3 — Document ingestion & OCR + synthetic generator
**Goal:** get real-shaped paper in.
- `Document` model + upload endpoint (local volume, hashing, status).
- OCR: render + Tesseract → per-page text + word boxes; text-layer fast path via pdfplumber.
- `fixtures/generate.py`: produce N fake scanned-style loan PDFs + ground-truth JSON.
- Tests: a generated PDF round-trips through OCR and yields text containing the known fields.
**Done when:** `make seed` generates sample PDFs; uploading one runs OCR and stores text/boxes.

## Milestone 4 — Field extraction (draft)
**Goal:** paper → structured draft.
- `Extractor` interface. Default `RuleExtractor` (regex + positional anchors per doc template)
  producing `Extraction.fields` with `confidence`, `page`, `bbox`.
- Map fields → proposed `Loan` + ordered `Events` (`mapping`), **not committed**.
- Optional `LLMExtractor` (local Ollama, OFF by default) behind the same interface.
- `POST /documents/{id}/extract`, `GET /documents/{id}/extraction`.
- Tests against ground-truth: measure field accuracy; assert low-confidence flagging works.
**Done when:** a seeded PDF produces a sensible draft extraction + proposed mapping, scored
against ground truth in tests.

## Milestone 5 — Human review → approve → commit
**Goal:** the trust step.
- Review UI: source page image with field highlights beside editable extracted values;
  low-confidence fields flagged. `PATCH` corrections (audited, `method: manual`).
- `POST /approve` → append `Loan` + `Events` to the ledger, run attribution, mark approved.
  `POST /reject` with reason. Full audit-log entries.
**Done when:** a reviewer can open a document, correct a field, approve it, and see the
resulting events appear in the ledger with income streams assigned — end to end.

## Milestone 6 — Dashboard (the money demo)
**Goal:** what the CEO sees.
- Revenue overview (by income stream / product / tribe + date filter, vendored Chart.js).
- Loan thread ("follow one loan") with links to source pages.
- Source document view with highlights.
- Review queue + ledger integrity badge + rules admin.
**Done when:** a non-technical person can, from the dashboard alone, see total revenue by
income stream and trace any single loan from its document page to its recognized revenue.

## Milestone 7 — Hardening, demo & docs
**Goal:** walk-in-ready.
- `docs/DEMO.md`: a <10-minute scripted walkthrough for a non-technical exec.
- `docs/SECURITY.md`: no egress, PII handling, append-only ledger, audit, offline install,
  how to point ingestion at real documents.
- `make demo`: seed + launch + print the demo URL. PII masking in logs verified. Final
  `ruff`/`pytest`/`mypy` pass.
**Done when:** on a clean machine with no internet, `make demo` brings up a populated system
and `docs/DEMO.md` reliably reproduces the money demo.

---

## Guardrails for the whole build
- Re-read `CLAUDE.md` §2 (hard constraints) before each milestone.
- Keep `docker compose up` green at every commit.
- Tests + lint pass before a milestone is "done."
- Anything tempting but out of scope → `docs/BACKLOG.md`. Any guessed banking rule →
  `docs/ASSUMPTIONS.md`.
