# Assumptions

Record here every banking rule or document-format detail that was guessed rather than
confirmed, so finance can correct it. Claude: append to this file instead of silently
inventing financial logic.

| # | Area | Assumption made | Needs confirmation from |
|---|------|-----------------|--------------------------|
| 1 | Product line | Proof targets a single term-loan product (`SME_TERM`). | Business |
| 2 | Income streams | Origination fee → `FEE_INCOME`; interest → `NET_INTEREST_INCOME`. | Finance |
| 3 | Revenue recognition | Fee recognized at `FeeCharged`; interest on a simple accrual from rate/tenor/principal. | Finance |
| 4 | Document type | One standard loan/disbursement document layout to start. | Operations |

> Replace/extend as real rules are confirmed.
