# SPEC.md — Attribution Engine functional & technical spec

This is the contract for *what* to build. `CLAUDE.md` governs *how* to behave. If you deviate
from this spec, update this file in the same change and note why.

---

## 1. Domain model

### Loan (case)
The unit of attribution. One originated loan.
- `id` (uuid)
- `external_ref` (the bank's loan/application number, from the document)
- `product_code` — single product line for the proof (e.g. `SME_TERM`)
- `tribe` / `channel` — e.g. `conventional`, `digital`
- `officer_ref` — originating officer / RM (pseudonymized id ok)
- `cost_center`
- `customer_ref` — **pseudonymized**; never store raw customer PII as the key
- `currency`
- `origination_date`
- `status` — derived from events
- timestamps, `created_by`

### Event (append-only, hash-chained)
The thread. Never updated or deleted.
- `id` (uuid), `seq` (monotonic), `loan_id`
- `type` — enum: `LeadCreated`, `Qualified`, `Approved`, `Disbursed`, `FeeCharged`,
  `InterestAccrued`, `RepaymentReceived`, `Adjusted`, `Reversed`
- `occurred_at` (business time), `recorded_at` (system time)
- `amount`, `currency` (nullable for non-financial events)
- `payload` (jsonb) — type-specific detail
- **attribution tags**: `product_code`, `tribe`, `officer_ref`, `cost_center`,
  `income_stream` (nullable until mapped)
- `source_document_id`, `source_field_refs` (which extracted fields produced this)
- `prev_hash`, `hash` (sha-256 over canonical serialization incl. `prev_hash`)
- `created_by`, `created_at`
- Corrections are **new** events (`Adjusted` / `Reversed`) referencing the original via payload.

### IncomeStream (reference data)
- `code` (e.g. `FEE_INCOME`, `NET_INTEREST_INCOME`, `OTHER_INCOME`)
- `name`, `gl_account` (optional)

### AttributionRule (data-driven, finance-editable)
Maps an event to an income stream + GL/cost center. No income-stream logic hardcoded.
- `id`, `priority`
- `match` (jsonb predicate): e.g. `{event_type: "FeeCharged", product_code: "SME_TERM"}`
- `assign`: `{income_stream: "FEE_INCOME", gl_account: "..."}`
- `active`, `created_by`, `effective_from`
- Engine: first matching rule by priority wins; unmatched financial events are flagged for
  review, never silently dropped.

### Document
- `id`, `filename`, `sha256`, `page_count`, `mime`
- `status` — `uploaded → ocr_done → extracted → in_review → approved → rejected`
- `uploaded_by`, timestamps
- stored file path (on local volume), `ocr_text` (per page), optional page images

### Extraction (draft, mutable until approved)
- `id`, `document_id`
- `fields` (jsonb): list of `{name, value, confidence, page, bbox, method}` where `method`
  is `rule|template|llm|manual`
- `mapping` (jsonb): proposed Loan + Events derived from fields
- `status` — `draft → corrected → approved → rejected`
- `reviewer`, `corrections` (audit of what the human changed), timestamps

### RevenueRecognition (derived/materialized)
- per loan, per income stream, per period — recognized amount.
- For the proof: fee income recognized at `FeeCharged`; interest income recognized from
  `InterestAccrued`/`RepaymentReceived` per a simple, documented rule. Keep the recognition
  rule explicit and in one place; do not scatter it.

### User / Role / AuditLog
- `User`: username, argon2 hash, role (`admin|finance_reviewer|finance_viewer`).
- `AuditLog`: actor, action, target, timestamp, ip (no PII payloads).

---

## 2. Document → events pipeline

1. **Upload** (`finance_reviewer`+): PDF stored on local volume, hashed, `Document` created.
2. **OCR** (`ingest`): render pages (pdf2image/pypdfium2), Tesseract OCR → per-page text +
   word boxes. If the PDF has a real text layer, use pdfplumber and skip raster OCR.
3. **Extract** (`extract`): produce `Extraction.fields` for the target fields (below). Default
   implementation = template/rules (regex + positional anchors per known document type).
   Optional `LLMExtractor` (local Ollama) behind the same interface; OFF by default.
   Every field carries a `confidence` and the `page`/`bbox` it came from.
4. **Map**: derive a proposed `Loan` + ordered `Events` from the fields (`mapping`). Show the
   proposal to the reviewer; do not commit.
5. **Review** (`review`): reviewer sees the source page with field highlights side-by-side with
   the extracted values; can correct any field (recorded as `method: manual` + audit). Low-
   confidence fields are flagged for attention.
6. **Approve**: on approval, the engine appends the canonical `Loan` + `Events` to the ledger
   (hash-chained), runs attribution rules to assign income streams, and marks the document
   `approved`. Rejection records a reason; nothing is committed.

### Target fields (one product line, starting set — refine in `docs/ASSUMPTIONS.md`)
`loan_ref, customer_name(→pseudonymize), product, principal_amount, currency,
disbursement_date, origination_fee, interest_rate, tenor_months, channel/branch, officer,
cost_center`. Map names → events:
- `Disbursed` (principal_amount, disbursement_date)
- `FeeCharged` (origination_fee) → typically `FEE_INCOME`
- `InterestAccrued` schedule derived from rate/tenor/principal → typically `NET_INTEREST_INCOME`

---

## 3. API surface (REST, JSON)

Auth: session cookie. All endpoints role-gated.

```
POST   /auth/login                      # -> session
POST   /auth/logout

POST   /documents                       # upload pdf (reviewer+)
GET    /documents?status=...
GET    /documents/{id}                  # meta + status
POST   /documents/{id}/ocr              # trigger/redo OCR
POST   /documents/{id}/extract          # produce draft extraction
GET    /documents/{id}/extraction       # draft fields + proposed mapping + page images
PATCH  /documents/{id}/extraction       # correct fields (reviewer)
POST   /documents/{id}/approve          # commit -> emits Loan + Events (reviewer)
POST   /documents/{id}/reject           # reason

GET    /loans/{id}                      # FULL THREAD: loan + ordered events + revenue + source refs
GET    /loans?product=&tribe=&from=&to=

GET    /attribution/revenue?by=income_stream|product|tribe&from=&to=   # rollups
GET    /attribution/rules               # list (viewer); CRUD for admin
POST   /attribution/rules
PATCH  /attribution/rules/{id}

GET    /ledger/verify                   # recompute hash chain -> {ok, broken_at?}
GET    /audit?from=&to=                 # admin

GET    /health                          # liveness
```

---

## 4. Dashboard (Jinja2 + htmx, vendored assets)

Pages, in priority order (build the first three first — they are the demo):
1. **Revenue overview** — totals + chart of revenue **by income stream**, toggle to by product /
   by tribe; date filter. The headline number finance has never had cleanly before.
2. **Loan thread ("follow one loan")** — search/select a loan → timeline of its events from
   origination to revenue, each event showing its income-stream tag and a link to the source
   document page it came from. This is the wow moment.
3. **Source document view** — the page image with extracted fields highlighted next to values.
4. **Review queue** — documents `in_review`, with the correction UI.
5. **Ledger integrity** — a badge/page calling `/ledger/verify`, plus event count, last hash.
6. **Rules** — view/edit attribution rules (admin).

Keep styling clean and legible (a simple dark or light theme). No heavy front-end framework.

---

## 5. On-prem deployment

- `docker compose`: services `app` (FastAPI/uvicorn) and `db` (postgres). Tesseract installed
  in the app image. Local volume for uploaded documents and page images.
- Config via env vars only; no secrets in the repo. Provide `.env.example`.
- `OLLAMA_ENABLED=false` by default; if true, talk only to `http://localhost:11434`.
- Document an **offline install** path (pre-pulled base images, vendored wheels) in
  `docs/SECURITY.md` / `README`.
- Provide `make` or scripts: `up`, `down`, `seed` (load synthetic docs + sample rules), `test`,
  `lint`, `demo` (one command that seeds and prints the demo URL).

---

## 6. Synthetic data generator (so we never need real PII to build)

`fixtures/generate.py` must produce N realistic but **fake** scanned-style loan PDFs for the
target product line: render a loan document template (varying names, amounts, dates, fees,
rates, branches, officers) to PDF, then optionally apply mild "scan" degradation (slight
rotation, noise, grayscale) so OCR is exercised realistically. Also generate matching
"ground-truth" JSON so extraction accuracy can be measured in tests.
