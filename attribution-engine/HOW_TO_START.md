# How to start the new VSCode project (for you, not for Claude)

These instructions bootstrap the build. Do this once.

## 1. Create the project
```bash
mkdir attribution-engine && cd attribution-engine
git init
```
Copy these files into it (this whole folder is the seed):
`CLAUDE.md`, `SPEC.md`, `BUILD_PLAN.md`, `README.md`, and the `docs/` folder.

Open the folder in VSCode and start Claude Code in it.

## 2. First message to Claude
Paste the **Kickoff prompt** from the top of `BUILD_PLAN.md`. It tells Claude to read the three
docs, confirm understanding, and build **Milestone 0 only**, then stop for your review.

## 3. Drive it milestone by milestone
- Review at the end of each milestone before saying "continue."
- Keep `docker compose up` green; make sure `make test` and `make lint` pass each time.
- When Claude asks about a banking rule (income streams, revenue recognition), give it the real
  answer or let it record an assumption in `docs/ASSUMPTIONS.md`.

## 4. When you have real documents
Build and test entirely on the **synthetic generator** first (no PII). Only point ingestion at
real scanned files once Milestone 5 (human review) works and `docs/SECURITY.md` is in place.
Put real files on the local volume only — never commit them to git.

## 5. Guardrails to hold the line
- If Claude proposes anything in `docs/BACKLOG.md`, say no — keep scope to the one thing.
- The three non-negotiables to repeat if needed: **no external network calls**, **append-only
  ledger**, **humans approve before commit**.

## Why these three docs
- **CLAUDE.md** = behavior + hard constraints (the guardrails Claude reads every session).
- **SPEC.md** = the contract for what to build (data model, pipeline, API, UI).
- **BUILD_PLAN.md** = the order to build it in, with a demoable checkpoint per milestone.

That separation is deliberate: it keeps Claude from wandering, over-building, or quietly
inventing financial logic.
