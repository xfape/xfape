# LEDGER — How Banking Works

An animated, self-contained learning app that teaches how banking works, built
for anyone aiming at C-level management. Each of 100 concept cards **morphs into
a full deep-dive article** in 3D; pages turn in space as you move through them.
It runs as a single HTML file with no build step, no dependencies, and no
network calls — just open `index.html`.

**Design:** a mix of Emil Kowalski's restraint (clean geometric sans, precise
spacing, soft spring motion) and the world of *TRON* — a dark neon stage, a
luminescent perspective grid, glowing card edges, and cyan/orange accents. It
ships in two themes: the neon-dark TRON stage and a light "blueprint" variant.

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

A card opens (click, `Space`, or **Read**) and expands into a full article laid
out in labelled sections:

- **What it is** — a plain-language definition
- **How it works** — the mechanism and why it matters to a bank
- **Worked example** — a concrete scenario with real numbers and a calculation
- **C-Suite Lens** — a one-line strategic takeaway

## Languages

The whole app — all 100 cards, module names, and the interface — is available in
three languages, switchable live from the top-right control:

- **English (EN)** — full expanded articles (What it is / How it works / Worked example)
- **Russian (РУ)** — full translation (concept + overview + C-suite lens)
- **Khmer (ខ្មែរ)** — full translation (concept + overview + C-suite lens)

The expanded English worked-examples are not yet translated; in Russian and
Khmer each article shows the concept, a concise overview, and the C-suite lens.
The chosen language and your "mastered" progress persist in the browser via
`localStorage`.

## Features

- **Shared-element morph** — the concept card expands into the full article, and
  collapses back, with a depth (dive-in) transition
- **3D page-turn navigation** — articles and cards rotate through space as you
  move next/previous
- Pointer parallax tilt on the concept card; a live animated TRON perspective
  grid rendered on canvas, with a glowing horizon
- Filter by module, shuffle the deck, and mark cards as mastered
- Full keyboard control — `←`/`→` navigate, `Space` open/close, `Esc` close,
  `K` master, `S` shuffle
- Light and dark themes (follows the OS/browser preference)
- Respects `prefers-reduced-motion` (disables the 3D transforms and parallax)
- Responsive layout for desktop and mobile

## Usage

Open `index.html` in any modern browser, or serve the folder statically:

```sh
python3 -m http.server
# then visit http://localhost:8000
```

## Structure

Everything lives in `index.html`. Content is data-driven:

- `CARDS` — the base curriculum (module, concept, short explanation, lens)
- `EXP` — the expanded English article content keyed by card number
  (`whatIs` / `how` / `example`)
- `I18N` — per-language overrides (concept, overview, lens) keyed by card number,
  plus translated module names and UI strings
- `UI_EN` / `UI_ADD` — the English interface strings and supplemental
  per-language UI strings

**Adding a language** is mechanical: translate the same structure (module names,
UI strings, and the 100 cards), add one entry to `I18N`, and add one button to
the language switcher. **Translating the expanded articles** into Russian/Khmer
means adding `whatIs` / `how` / `example` to each language's card entries; the
renderer will pick them up automatically.
