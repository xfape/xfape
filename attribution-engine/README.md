# Attribution Engine — Phase 1 Proof

Turns the bank's **scanned loan documents** into an **immutable, attributed event ledger** so
finance can read **revenue by income stream / product / tribe — live**, and **follow any single
loan from its source document to the income stream it lands in**. Runs **fully on-premise**, no
cloud, no external network calls.

This repository is **Phase 1** of a larger plan (a shared "process rail"). It deliberately does
**one thing** well. Agents, process design, and write-back are out of scope here.

## Start here (for Claude Code)
1. `CLAUDE.md` — how to behave + the hard constraints (read first).
2. `SPEC.md` — what to build (domain model, pipeline, API, dashboard, deployment).
3. `BUILD_PLAN.md` — the milestone order + the kickoff prompt to paste in.

## Stack
Python 3.12 · FastAPI · PostgreSQL · Tesseract OCR (local) · Jinja2 + htmx · Docker Compose.
Optional local LLM extraction via Ollama (off by default). No cloud dependencies.

## Quickstart (filled in as the build progresses)
```bash
cp .env.example .env
make up        # docker compose: app + postgres
make seed      # generate synthetic sample loan PDFs + sample rules
make demo      # seed, launch, print the demo URL
make test      # pytest
make lint      # ruff
```

## The "money demo"
Load scanned loan documents → review/approve what was extracted → see total revenue split by
income stream → click any loan and trace it from the source page to its recognized revenue.

## Status
- [ ] M0 Scaffold  [ ] M1 Ledger  [ ] M2 Attribution  [ ] M3 Ingestion/OCR
- [ ] M4 Extraction  [ ] M5 Review→Commit  [ ] M6 Dashboard  [ ] M7 Hardening/Demo

> ⚠️ On-prem, PII-handling system. Never add external network calls. The event ledger is
> append-only. Machine extraction is always reviewed by a human before anything is committed.
