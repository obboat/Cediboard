# The Lovable master prompt

Prerequisites (Phase 0 of LOVABLE-HANDOVER.md): DESIGN-SPEC.md pasted into
Lovable's Knowledge; `reference/index.html` and `reference/assets/` (the six
story JPEGs) committed to the connected GitHub repo.

Copy everything between the lines into Lovable's chat as one message.

---

Build "Cediboard" — an interactive daily data newspaper for Ghana that reads
as a real paper you flip. A complete, working implementation exists at
reference/index.html in this repo. That file is the single source of truth:
your job is a faithful PORT to React, not a redesign. The project Knowledge
file (the Cediboard design spec) is binding. Where anything is ambiguous,
open reference/index.html and copy what it does.

STACK & THEME
- React + TypeScript + Tailwind (no UI kit restyling of the pages).
- Register these design tokens exactly in the Tailwind theme and as CSS
  variables: ink #1a1814, muted #6a6458, ivory #f5f3ee, paper #fbfaf6,
  gold #c8960c, emerald #3BA874, coral #E05040, track #e4e1d8,
  bars #cdc9bf, flag red #CE1126, flag yellow #FCD116, flag green #006B3F.
  No other hues anywhere.
- Typography: "Google Sans Flex" (Google Fonts, weights 400–800) for
  EVERYTHING, fallback "Google Sans","Segoe UI",Roboto,Arial. Do not
  substitute any other font.
- Desk background behind the book: radial gradient #46423a → #2e2b25 →
  #201e1a.

PAGE GEOMETRY
- Every page is a 3:4 sheet on a fixed 810×1080px design grid, scaled with
  a CSS transform to fit the viewport (never reflowed).
- Viewport ≥820px: two-page spread (open book). Below 820px: single page.

THE FLIP ENGINE — port from the reference EXACTLY; do not use any carousel,
swiper, or slider library, and do not simplify the animation:
- Spread mode: sheets absolutely positioned at the book's right half,
  transform-origin left center, flipping rotateY(0 → −180deg); each sheet
  has .front/.back faces with backface-visibility:hidden; book perspective
  2800px; transition 1.7s cubic-bezier(.42,.03,.18,1); a gradient shade
  overlay fades to opacity .45 over .85s while a sheet turns; the turning
  sheet holds top z-index for 1.8s; stacked paper-edge strips on both sides
  ~3px per remaining sheet; the closed cover and closed back-cover
  re-center the book by half a page width with the same 1.7s curve.
- Single mode: page fold-turn rotateY(0 → −98deg) about the left edge,
  950ms (420ms during multi-page riffles), slight brightness dip, Web
  Animations API with the animation cancelled after commit (no persisted
  fill), z-index swap between outgoing/incoming.
- Auto-open: if the user hasn't interacted 1.6s after load, flip the cover
  once. Hover on the turn zone tilts the top sheet −7° (0.8s). Index jumps
  riffle page-by-page at 420ms stagger — never teleport.

NAVIGATION: click zones on the outer 16% of the book, round prev/next
buttons, ← → keys, 60px swipe, progress dots (one per spread), a bottom
toolbar pill (CEDIBOARD wordmark · dots · "Page N / 16" · INDEX button), an
INDEX overlay listing all pages, and clickable index rows on the cover.

CONTENT — 16 pages, ids and order:
cover, headlines, usd, gbp, eur, fuel, gse, sports, street, weather,
traffic, trending, macro, tomorrow, sponsor, back.
- All page content renders from ONE typed `edition` JSON object (schema in
  the Knowledge file §7). Seed it with the exact data hardcoded in
  reference/index.html — every number, caption, SAMPLE flag, and source
  line. No lorem ipsum anywhere.
- Every page carries the shared chrome from the reference: header (eyebrow
  label with gold keyword + FINEX INSIGHTS wordmark + "DATA · CLEARLY
  TOLD") and footer (source line, 3px rule, Ghana flag chip with black
  star, "@finexskillshub · X & LinkedIn · @finex insights", "PAGE NN ·
  JULY 2026" folio).
- Build as shared components: PageChrome, DeltaChip (▲ coral / ▼ emerald /
  ● gold — colors follow MEANING not sign), FxCard (gold hanging-tab card),
  BarRow (declared fixed ceilings, stated on the page), IndexPanel,
  DirPill, StoryCard, FinexView dark box, NoteStrip, TocRow, AdSlot
  (dashed gold box, "Place your advert here", tel:+233244782356 and
  https://wa.me/233244782356 links), full-page sponsor layout.
- The six story images live at reference/assets/ — copy them into the app's
  public assets keeping the exact filenames and use them on the headlines
  page and the cover lead story.

DEFINITION OF DONE
Opening the app beside reference/index.html in two browser windows shows no
visible difference in layout, spacing, colors, typography, image placement,
or flip behavior — at 1380px width (spread) and 390px width (single page).
Specifically: the cover auto-opens after ~1.6s; a flip takes ~1.7s with a
slow settle and travelling shadow; SAMPLE chips appear on gbp, eur, fuel,
sports, street, weather, traffic, trending and parts of macro; the ad
slots' call and WhatsApp links work; the INDEX overlay and cover rows jump
correctly; nothing overflows any page.

Do not add features, pages, animations, or styling beyond this. If any
detail is unclear, match reference/index.html.

---

## Upload checklist

| Asset | Destination | Notes |
|---|---|---|
| DESIGN-SPEC.md contents | Lovable → Settings → Knowledge | Paste as text |
| index.html | GitHub repo → reference/index.html | The ground truth |
| assets/story1…story6 (.jpg ×6) | GitHub repo → reference/assets/ | Keep exact filenames |
| LOVABLE-HANDOVER.md | GitHub repo → reference/ (optional) | Phases 2–4 prompts |
| Original 3 sample cards | Keep on PC | Attach in chat only if disputing a visual detail |
| CLAUDE.md / .claude folder | Do NOT upload | Claude Code context, not Lovable's |

Fonts need no upload (Google Fonts). If skipping GitHub: paste the prompt,
then send index.html's contents in 2–3 chunks with "store this as
reference/index.html".
