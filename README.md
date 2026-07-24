# LEDGER — How Banking Works

An animated, self-contained flashcard game that teaches how banking works, built
for anyone aiming at C-level management. It runs as a single HTML file with no
build step, no dependencies, and no network calls — just open `index.html`.

## The curriculum

100 cards arranged as a learning path across 10 modules, starting from *how
interest forms* and building up to the metrics a bank CEO is judged on:

1. **Interest & Money** — time value, compounding, real vs nominal rates, the yield curve
2. **The Bank Model** — borrow short / lend long, net interest margin, money creation
3. **Balance Sheet** — the inverted bank balance sheet, ROE/ROA, leverage, price-to-book
4. **Lending & Credit** — the 5 Cs, PD/LGD/EAD, expected loss, provisioning, NPLs
5. **Risk** — the four risk types, VaR, IRRBB, the three lines of defense, stress testing
6. **Capital & Rules** — Basel, RWA, CET1, LCR/NSFR, deposit insurance, lender of last resort
7. **Treasury & ALM** — asset-liability management, FTP, duration, hedging, ICAAP
8. **Payments & Ops** — clearing & settlement, interchange, SWIFT, RTGS, AML/KYC, cost-to-income
9. **Markets & Fees** — capital markets, securitization, wealth management, derivatives
10. **Strategy & Value** — ROE vs cost of equity, economic profit, fintech disruption, the CEO scorecard

Each card flips from a concept prompt to a detailed explanation plus a **C-Suite
Lens** — a one-line strategic takeaway.

## Languages

The whole curriculum — all 100 cards, module names, and the interface — is
available in three languages, switchable live from the top-right control:

- **English (EN)**
- **Russian (РУ)**
- **Khmer (ខ្មែរ)**

The chosen language and your "mastered" progress persist in the browser via
`localStorage`.

## Features

- 3D card-flip animation and a live guilloché (banknote-engraving) canvas backdrop
- Filter by module, shuffle the deck, and mark cards as mastered
- Full keyboard control — `←`/`→` navigate, `Space` flip, `K` master, `S` shuffle
- Light and dark themes (follows the OS/browser preference)
- Respects `prefers-reduced-motion`
- Responsive layout for desktop and mobile

## Usage

Open `index.html` in any modern browser, or serve the folder statically:

```sh
python3 -m http.server
# then visit http://localhost:8000
```

## Structure

Everything lives in `index.html`. Content is data-driven:

- `CARDS` — the English base curriculum (module, concept, explanation, lens)
- `I18N` — per-language overrides keyed by card number, plus translated module
  names and UI strings
- `UI_EN` — the English interface strings

**Adding a language** is mechanical: translate the same structure (10 module
names, ~21 UI strings, 100 cards), add one entry to `I18N`, and add one button
to the language switcher.
