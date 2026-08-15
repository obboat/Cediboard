# CEDIBOARD — Design Specification

**Version 1.0 · July 2026 · A Finex Insights publication (Finex Skills Hub, Accra)**

This document is the single source of truth for Cediboard's visual identity,
interaction model and content rules. When porting to Lovable (React + Tailwind +
Supabase), treat everything here as a hard constraint, not inspiration.
**Do not reimagine the design. Port it.**

---

## 1. Product definition

Cediboard is a daily interactive data newspaper on the topical issues shaping
Ghana — the cedi, the markets, the streets, the weather and the scores — told
through simple visualizations that let anyone get the insight in seconds.

- **The promise: today's Ghana in 2 minutes.** Every design decision serves it.
- The reading metaphor is a **physical paper you flip**, not a website you scroll.
- One big idea per page. The hero number/insight is always the largest element.
- A new edition publishes daily at 06:00 GMT.

---

## 2. Design tokens (Ivory Charcoal system)

Port these verbatim as CSS custom properties / Tailwind theme colors.

| Token | Value | Usage |
|---|---|---|
| `--ink` | `#1a1814` | Primary text, axes, rules, dark panels |
| `--muted` | `#6a6458` | Labels, secondary text, sources |
| `--ivory` | `#f5f3ee` | Page background; text on dark panels |
| `--paper` | `#fbfaf6` | Card backgrounds on ivory pages |
| `--gold` | `#c8960c` | Brand accent: current-period bars, eyebrow keywords, hero highlights |
| `--emerald` | `#3BA874` | GOOD direction (falling inflation, flowing traffic, cedi gains) |
| `--coral` | `#E05040` | BAD direction (rising prices, heavy traffic, losses) |
| `--track` | `#e4e1d8` | Bar/progress track backgrounds, hairline borders |
| `--barsc` | `#cdc9bf` | Non-current bars, dotted leaders, dividers |
| Ghana flag | `#CE1126` / `#FCD116` / `#006B3F` + black star | Footer brand chip only |
| Muted-on-dark | `#a49d8e` | Secondary text on ink panels |
| Dark divider | `#3a362e` | Hairlines inside ink panels |

**Non-negotiable:** emerald/coral follow **meaning, not sign**. Falling inflation
is emerald even though the delta is negative. Never introduce new hues; if a
section needs identity, use tints/weights of these tokens.

---

## 3. Typography

- **Typeface:** Google Sans Flex (Google Fonts), weights 400–800.
  Fallback stack: `"Google Sans Flex","Google Sans","Segoe UI",Roboto,Arial,sans-serif`.
- All numbers and UI use this face. (Optional future direction: a serif display
  face for headlines only — decide once, apply everywhere.)

Type scale on the 810×1080 page grid (px):

| Role | Size / weight / tracking |
|---|---|
| Masthead | 78–96 / 800 / +2px |
| Page hero number | 110–170 / 800 / −4 to −7px, line-height .85 |
| Page title (fx boards) | 46–62 / 800 / −1px |
| Section headline (h2) | 16.5–19 / 800 |
| Body / captions | 12.5–14.5 / 400–600, line-height 1.4–1.5 |
| Eyebrow / section labels | 12–15 / 600–800 / +3 to +4.5px, UPPERCASE, muted with keyword in gold |
| Wordmark | 18 / 700 / +3px ("FINEX" ink + "INSIGHTS" gold) |
| Source line | 12.5 / 400 / muted |
| SAMPLE chip | 10.5 / 800 / +2px, gold text + 1.5px gold border, pill |

Spacing: 48px page side margins, 40px top. Vertical rhythm in 8px steps.

---

## 4. Page geometry & chrome

- **Every page is 3:4** — an 810 × 1080 design grid, scaled to fit the viewport.
- Page background ivory; "dark" pages (back cover) invert to ink with ivory text.

**Header (every page):** eyebrow top-left (series label, keyword in gold) ·
wordmark top-right (`FINEX INSIGHTS` + `DATA · CLEARLY TOLD`).

**Footer (every page):** source line (muted, names the institution) · 3px ink
rule · Ghana flag chip with black star · handles
`@finexskillshub · X & LinkedIn · @finex insights` · folio `PAGE NN · MONTH YEAR`.

**SAMPLE rule:** any figure not confirmed from a primary source carries a
visible SAMPLE chip next to the eyebrow or the affected block. Never ship
placeholder numbers unflagged.

Sources cite the original authority: Ghana Statistical Service, Bank of Ghana,
CediRates, Databank Research, GSE, Ghana Meteorological Agency.

---

## 5. Component inventory

Build these once, reuse everywhere:

- **PageChrome** — header + footer wrapper described above.
- **DeltaChip** — `▲ 0.06` coral (up), `▼ 0.04` emerald (down), `● no change`
  gold. One component; used on FX cards, street prices, macro cells, glance strip.
- **FxCard** — hanging-tag card: gold top tab with two ivory slots, provider
  name, DeltaChip + `¢` + rate (800 weight), rate-type label, change line.
- **BarRow** — label · track (`--track`) · fill · value. Fill colors: gold for
  "current/leader", `--barsc` for the rest, emerald/gold/coral when the bar
  encodes status (traffic). **Every bar chart states its ceiling on the page**
  (e.g. "bars scaled to a fixed 150k-mention ceiling"). Never scale silently
  to the leading value.
- **IndexPanel** — dark ink panel: label (12px, tracked, muted-on-dark), big
  value (44px, 800), pts d/d, gold pill `▲ +0.01%`, YTD block right-aligned.
- **DirPill** — filled pill (coral/emerald/gold) with arrow + word (RISING /
  HEAVY / HOLD).
- **StoryCard** — icon chip (tinted bg + emoji/glyph) + headline, illustration
  (rounded 9px, object-fit cover), 2-line clamped caption.
- **FinexView / Hotspot box** — dark ink rounded box, gold 12px tracked label,
  14.5px ivory body with gold bold highlights. This is the editorial voice slot.
- **TocRow** — number (gold) · title · dotted leader · description; whole row
  is a link to its page.
- **AdSlot** — corporate sponsorship placeholder: 2px dashed gold border,
  rounded 14px, faint gold-tinted fill, centered "ADVERTISEMENT" eyebrow
  (10px, tracked, muted) + "Place your advert here" (17px, 800) + contact
  line with the phone number in gold ("Contact us on +233 24 478 2356").
  The number is a tap-to-call link (`tel:+233244782356`) and is followed by a
  "WhatsApp us" link (`https://wa.me/233244782356`, new tab).
  Currently placed on the Sports, Macro Watch and Tomorrow pages; in the
  ported app the slot should render from edition data (`{ "type": "ad",
  "booked": false }`) so a sold slot can swap in real creative per edition.
- **NoteStrip** — track-colored rounded strip for "why it matters" one-liners.

---

## 6. The flip interaction (port verbatim)

- **≥820px viewport: two-page spread.** Sheets rotate about the spine
  (`transform-origin: left center`, `rotateY 0 → −180°`), front/back faces with
  `backface-visibility: hidden`, book `perspective: 2800px`.
- **Timing: 1.7s, `cubic-bezier(.42,.03,.18,1)`** — slow lift, long settle.
  A travelling shade (gradient overlay, opacity .45) fades in/out over 0.85s
  while a sheet turns. The turning sheet holds top z-index for 1.8s.
- Closed cover and closed back-cover **re-center themselves** (book shifts
  ±half a page, same 1.7s curve).
- **<820px: single-page mode.** Page fold-turn: `rotateY 0 → −98°` about the
  left edge, 950ms (420ms during multi-page riffles), same easing, slight
  brightness dip. Cancel WAAPI animations after commit (no `fill:forwards`
  residue).
- **Stacked paper edges** on both sides of the book, thinning as pages are
  consumed (≈3px per remaining sheet).
- **Navigation:** tap zones (outer 16% of the book), ‹ › buttons, ← → keys,
  swipe (60px threshold), progress dots, INDEX overlay, cover TOC links.
  Index jumps riffle page-by-page (420ms stagger) — never teleport.
- **Hover peek:** top sheet tilts −7° when hovering the turn zone (0.8s).
- **Auto-open:** if the reader hasn't interacted ~1.6s after load, the cover
  opens itself once. Never auto-flip beyond that.
- **Reduced motion:** honor `prefers-reduced-motion` with a simple crossfade.
- Keep the **page count even** — every sheet has two sides.

Bottom toolbar: `CEDIBOARD` wordmark · dots · `Page N / M` · INDEX button, in a
blurred ink pill fixed at bottom-center. Desk background: warm dark radial
(`#46423a → #201e1a`).

---

## 7. Edition data schema (Supabase)

One table drives everything:

```sql
create table editions (
  id          uuid primary key default gen_random_uuid(),
  edition_no  int not null,
  publish_date date not null unique,
  status      text not null default 'draft',  -- draft | published
  data        jsonb not null
);
```

`data` shape (types map 1:1 to page templates):

```jsonc
{
  "edition": { "no": "001", "day": "Friday", "date": "3 July 2026", "city": "Accra" },
  "glance": [ { "label": "USD → GHS", "value": "¢11.36", "delta": 0.06 } ],
  "lead":   { "headline": "...", "image": "...", "page": 2 },
  "pages": [
    { "type": "stories",  "items": [ { "headline": "", "caption": "", "image": "", "chip": "" } ] },
    { "type": "fx_board", "code": "USD", "vsDate": "1 July", "sample": false,
      "rates": [ { "name": "Bank of Ghana", "v": 11.36, "d": 0.06, "lbl": "Selling rate" } ] },
    { "type": "gse",      "indices": [], "kpis": [], "gainers": [], "laggards": [],
      "traders": [], "view": "..." },
    { "type": "sports",   "match": {}, "table": [], "abroad": [] },
    { "type": "fuel",     "window": "July W2", "sample": true,
      "averages": { "petrol": 12.90, "dPetrol": 0.08, "diesel": 13.40, "dDiesel": 0.09 },
      "omcs": [ { "name": "GOIL", "petrol": 13.05, "dPetrol": 0.15, "diesel": 13.55, "dDiesel": 0.15 } ] },
    { "type": "street",   "items": [ { "name": "", "price": 0, "unit": "", "d": 0, "emoji": "" } ] },
    { "type": "weather",  "hero": {}, "days": [], "regions": [], "note": "" },
    { "type": "traffic",  "citywide": 72, "level": "HEAVY", "corridors": [], "hotspot": "" },
    { "type": "trending", "spotlight": {}, "items": [], "ceiling": 150000 },
    { "type": "macro",    "hero": {}, "months": [], "cells": [] },
    { "type": "tomorrow", "teasers": [] },
    { "type": "sponsor",  "booked": false }
  ]
}
```

The `fuel` page's live source is CediRates (`cedirates.com/fuel-prices/gh/`).
Note: that page renders prices client-side and the site returns 403 to plain
fetchers — the backend needs either their API (if access is granted), a
headless-browser scrape, or manual daily entry. Fuel prices change per NPA
pricing window (roughly twice monthly), not daily.

Rendering rule: page components are pure functions of this JSON. Publishing a
new edition = inserting one row. Assets (story illustrations) go in Supabase
storage or `/assets`.

---

## 8. Backend & data pipeline (daily production)

Target: a fresh draft edition ready for review by **08:00 GMT**, reviewed and
published by **09:00 GMT**, every day.

### Architecture

1. **Collector job** — scheduled script (GitHub Actions cron or Supabase Edge
   Function, ~05:45 GMT). Fetches every source, assembles the full
   `edition.data` JSON (§7), inserts it into `editions` as `status: draft`.
2. **Editorial pass** — Claude API, inside the same job, writes the prose
   slots only (headline summaries, Finex View, Window watch, Read of the
   room) from the fetched numbers. **Numbers never come from the LLM.**
3. **Morning status message** (~08:00, WhatsApp/email to the editor):
   per-page checklist — ✅ fresh · ⚠️ stale (last-known-good) · ✍️ manual
   entry needed — plus a draft preview link.
4. **Review gate** — editor approves in a minimal admin page; publish flips
   `status: published`. If not reviewed by 09:00, auto-publish with
   staleness badges (a daily paper that says "as of yesterday" beats one
   that doesn't come out).
5. **Fallback rule (global)** — a failed source keeps its last-known-good
   data and the page renders an "as of <date>" badge automatically. No
   single feed may block the edition.

Freshness is a feature: every page footer shows its source **and** fetch
timestamp ("as of 06:00, 4 July").

### Source mapping

| Page | Source | Automation | Notes |
|---|---|---|---|
| USD/GBP/EUR boards | CediRates API (request key — see below) | Full | Fallback: BoG official rates (bog.gov.gh), Mastercard settlement API (free, public), Visa rates calculator, Binance P2P public endpoint |
| Fuel Board | CediRates API / NPA windows | Full, low-frequency | Prices move per NPA window (~1st & 16th); manual entry twice a month is acceptable |
| GSE Scoreboard | GSE daily results + kwayisi.org GSE API | Full | Previous day's close, final by ~5pm — fetch the evening before |
| Weather | Open-Meteo (free, keyless) | Full | Covers Accra + all regional cities incl. rain probability; preferred over AccuWeather's paid API |
| Traffic Watch | TomTom Traffic API (free tier) or Google Distance Matrix (paid) | Full | Snapshot the 7 corridors at 06:30 (morning peak); congestion = time lost vs free-flow |
| Trending Now | trends24.in/ghana or getdaytrends scrape | Mostly | Official X API trends ≈ $200/mo — skip at launch. Label the page "X trends"; TikTok/IG have no public trend APIs |
| The Headlines | RSS (MyJoyOnline, Citi Newsroom, GhanaWeb, 3News, GNA) → Claude drafts 6 cards | **Draft-only, human-approved** | Story selection is editorial judgment; never auto-publish. Illustrations: icon chips by default; AI images optional and reviewed |
| Street Prices | Own survey (WhatsApp/Google Form, 2–3 market contacts) | Manual, weekly | No API exists — this page is a differentiator *because* it can't be scraped |
| Macro Watch | GSS CPI release (monthly) · BoG T-bill auctions (weekly scrape) · MPC (per meeting) | Semi | CPI hand-entered on release day; cedi YTD computed from stored FX history |
| Sports Board | API-Football (api-sports.io: GPL, Black Stars, Ghanaians abroad) | Full (~$0–30/mo) | Or manual on match days at launch |
| Cover | Derived: glance from FX/GSE/CPI, lead = top headline | Auto with editor override | Lead-story choice surfaces in the review step |
| Tomorrow / Sponsor / Back | Templates + bookings table | Auto | Teasers auto-fill from fixtures/release calendar |

### The CediRates dependency

Their pages render prices client-side and return 403 to plain fetchers —
scraping is not a plan. They offer developer API access: **request a key as
Finex Insights**, with attribution on every board (already the house style).
Until granted, the boards run on the fallback primary sources above, which
read credibly on their own ("Source: Bank of Ghana; Mastercard; Binance").

### Build order

1. Weather + Traffic + GSE (free, keyless/easy APIs — proves the pipeline).
2. CediRates API conversation in parallel (lead time).
3. Headlines RSS + Claude drafting behind the review gate.
4. Trending scrape, Sports API, then the manual routines (street prices,
   macro calendar) as documented checklists inside the review dashboard.

## 9. Editorial house rules (from the Finex Insights system)

1. Bars scale to **fixed, declared ceilings** — never silently to the leader.
2. One big idea per page; the hero element dominates.
3. Captions in plain language; no unexplained jargon; drama-first scoreboard tone.
4. **Inferred or unverified figures are flagged SAMPLE** on the page.
5. Direction colors follow meaning, not sign.
6. For Ghana fiscal data, the outturn column is authoritative — say so on the page.
7. Every page cites its source institution in the footer.

---

## 10. Lovable porting checklist

- [ ] CSS custom properties copied exactly (§2), exposed through the Tailwind theme.
- [ ] Google Sans Flex loaded from Google Fonts; no font substitution.
- [ ] Flip engine ported as-is (§6) — not replaced with a carousel/swiper library.
- [ ] Page components render from `edition.data` JSON only (§7); no hardcoded numbers.
- [ ] PageChrome, DeltaChip, BarRow, IndexPanel, DirPill, FinexView built as shared components (§5).
- [ ] SAMPLE chip renders automatically wherever `sample: true`.
- [ ] Even page count enforced (pad with the Tomorrow page).
- [ ] `prefers-reduced-motion` fallback.
- [ ] Deep links `#page=<id>`; OG tags per edition; PWA manifest + service worker.
- [ ] Scheduled collector job + draft/publish review gate per §8, with the
      morning status message and per-source last-known-good fallbacks.
- [ ] "As of <time>" freshness badge rendered from each source's fetch timestamp.
- [ ] Reference implementation: `index.html` in this folder — when in doubt, match it.
