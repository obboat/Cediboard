# Cediboard → Lovable handover guide

Follow the phases in order. Do not ask Lovable to build everything in one
prompt — it will cut corners on exactly the things that make Cediboard
distinctive (the flip engine and the design system). Each phase ends with a
verification checklist; do not move on until it passes.

---

## Phase 0 — Pre-flight (10 minutes, you do this once)

1. Create the Lovable project (lovable.dev → New project).
2. **Knowledge file**: Project → Settings → Knowledge → paste the ENTIRE
   contents of `DESIGN-SPEC.md`. This makes every future prompt obey the
   design system without you repeating it.
3. **GitHub sync**: Project → GitHub → Connect and create the repo. Then, on
   your machine, clone that repo and copy in a `reference/` folder containing:
   - `index.html` (the working newspaper — the single most important file)
   - `assets/` (the six story JPEGs)
   - `DESIGN-SPEC.md` and this file
   Commit and push. Lovable can now read the real implementation instead of
   guessing from prose.
4. **Supabase**: Lovable → Integrations → Supabase → Connect (create a new
   Supabase project when prompted). Phase 2 depends on this.

---

## Phase 1 — Static port (the paper itself)

Paste this prompt:

> Build the Cediboard interactive flip newspaper. The complete working
> implementation is in `reference/index.html` in this repo — treat it as the
> source of truth and port it faithfully; do not redesign it.
>
> Requirements:
> 1. React + TypeScript + Tailwind. Map the CSS custom properties from the
>    reference (`--ink #1a1814`, `--ivory #f5f3ee`, `--gold #c8960c`,
>    `--emerald #3BA874`, `--coral #E05040`, `--paper`, `--track`, `--barsc`)
>    into the Tailwind theme. Load Google Sans Flex from Google Fonts. No
>    other fonts, no other hues.
> 2. Port the flip engine from the reference EXACTLY as implemented: 3:4
>    pages on an 810×1080 design grid scaled to the viewport; two-page
>    spread with 3D sheet flips (rotateY about the spine, front/back faces,
>    backface-visibility hidden, perspective 2800px, 1.7s
>    cubic-bezier(.42,.03,.18,1), travelling shade, stacked paper edges,
>    cover auto-centres when closed) for viewports ≥820px; single-page
>    fold-turn (rotateY to −98°, 950ms) below 820px. DO NOT use a carousel,
>    swiper, or page-slider library. DO NOT simplify the animation.
> 3. Navigation: tap zones on the page edges, prev/next buttons, arrow keys,
>    swipe, progress dots, bottom toolbar (CEDIBOARD wordmark · dots ·
>    "Page N / 16" · INDEX button), INDEX overlay, clickable cover index
>    rows, hover corner-peek, auto-open of the cover after 1.6s idle.
> 4. Recreate all 16 pages as React components that render from a single
>    typed `edition` JSON object (shape defined in the Knowledge file §7).
>    Seed it with the exact data currently hardcoded in the reference,
>    including the SAMPLE flags. Every page keeps the shared chrome: header
>    (eyebrow + FINEX INSIGHTS wordmark) and footer (source line, ink rule,
>    Ghana flag chip with black star, handles, PAGE NN folio).
> 5. Shared components: PageChrome, DeltaChip, FxCard, BarRow, IndexPanel,
>    DirPill, StoryCard, FinexView, NoteStrip, TocRow, AdSlot (with
>    tel:+233244782356 and https://wa.me/233244782356 links), and the
>    full-page sponsor layout.
> 6. Serve the six story images from `reference/assets/` (copy them into the
>    app's public assets).
>
> Definition of done: opening the app side-by-side with
> `reference/index.html` shows no visible difference in layout, colors,
> typography, or flip behavior on both desktop and mobile widths.

**Verify before Phase 2** (open the preview and check):
- [ ] Cover auto-opens after ~1.6s; flip takes ~1.7s with a soft settle
- [ ] Desktop shows a two-page spread; narrow window shows single pages
- [ ] All 16 pages present, page numbers correct, INDEX overlay jumps work
- [ ] Cover index rows riffle to the right pages
- [ ] Colors match exactly (spot-check gold #c8960c on eyebrows, coral/emerald deltas)
- [ ] SAMPLE chips visible on GBP, EUR, Fuel, Sports, Street, Weather, Traffic, Trending pages
- [ ] Ad slots render with working tel/WhatsApp links

If the flip looks like a slide/carousel or the fonts changed: reply
"The flip engine and typography must match reference/index.html exactly —
re-read it and port the .sheet/.face CSS and the flip JS as-is."

---

## Phase 2 — Supabase backend (editions + review gate)

Paste this prompt:

> Add the backend per Knowledge file §7 and §8:
> 1. Supabase table `editions` (id uuid pk, edition_no int, publish_date
>    date unique, status text default 'draft', data jsonb). RLS: anyone can
>    read rows where status='published'; only authenticated admins can read
>    drafts or write.
> 2. The public app loads the latest published edition; `?date=YYYY-MM-DD`
>    loads that day's edition (archive).
> 3. Admin area at /admin (Supabase email auth, single admin user):
>    - list of editions with status
>    - draft preview (renders the full flip paper from the draft JSON)
>    - a per-page freshness panel: each page shows its data timestamp and
>      ✅ fresh / ⚠️ stale / ✍️ manual-entry-needed
>    - JSON editor (or simple form fields) for the manual pages: Street
>      Prices, Macro CPI, Headlines text, Tomorrow teasers
>    - one-click Publish (flips status to published)
> 4. Migrate the seeded edition from Phase 1 into the table as edition 001,
>    published. The app must render entirely from the database now — no
>    hardcoded page data left in the frontend.

**Verify:**
- [ ] Public app renders edition 001 from Supabase (network tab shows the query)
- [ ] /admin requires login; draft editions invisible to logged-out users
- [ ] Editing a draft field in admin and publishing updates the public paper
- [ ] `?date=` archive URL works

---

## Phase 3 — Collectors (automated daily draft by 08:00 GMT)

Do these ONE SOURCE AT A TIME (each is a small, testable prompt). Start:

> Create a Supabase Edge Function `collect-edition` scheduled daily at 05:45
> GMT. It builds tomorrow's draft edition row by copying the latest
> published edition and then refreshing, in this order, with a per-source
> try/catch (a failed source keeps the previous data and sets
> `meta.stale=true` and `meta.asOf` for that page — never abort the run):
>
> 1. WEATHER: Open-Meteo API (no key) — Accra hero (temp, condition, rain
>    chance, humidity, wind, UV) + 5-day forecast + today's temp/rain for
>    Kumasi, Tamale, Takoradi, Cape Coast, Ho, Bolgatanga.
> 2. TRAFFIC: TomTom Traffic API (key in Supabase secrets) — current vs
>    free-flow travel time for the 7 corridors listed in the edition JSON;
>    congestion % = time lost vs free-flow; city-wide = weighted average.
> 3. GSE: fetch previous close from the GSE daily results
>    (dev.kwayisi.org/apis/gse for equities; index values from gse.com.gh
>    market summary) — indices, market cap, turnover, counters, top
>    gainers/laggards, top traders by value.
>
> Record per-source status in `data.meta.sources[]` (name, ok, asOf, error).
> Every affected page must render an "as of <time>" badge from meta when
> stale. Keep the SAMPLE chip only on pages whose data is still placeholder.

Then, as separate follow-up prompts, add one at a time:
- FX + Fuel: CediRates API once the key arrives (secrets), else BoG scrape +
  Mastercard settlement API + Binance P2P endpoint per §8 fallbacks.
- Headlines: RSS (MyJoyOnline, Citi Newsroom, GhanaWeb, 3News, GNA) → call
  Claude API (claude-sonnet-5, key in secrets) to select and draft 6 story
  cards → ALWAYS into the draft for human review, never auto-published.
- Trending: scrape trends24.in/ghana; volumes are estimates — label them.
- Sports: API-Football (GPL table, Black Stars fixtures, Ghanaian players).
- Editorial prose pass: Claude writes Finex View / Window watch / Read of
  the room FROM THE FETCHED NUMBERS in the draft (prose only — the model
  never invents figures).
- Status message: at 08:00 GMT email (Resend integration) the admin a
  per-page checklist + draft preview link. Auto-publish at 09:00 GMT if the
  draft has ≥N fresh sources and wasn't manually held (make N configurable).

**Verify each collector:** run the function manually from the Supabase
dashboard, open the draft in /admin, confirm the page shows real data with a
fresh timestamp, and that killing the API key produces a ⚠️ stale badge —
not a broken page.

---

## Phase 4 — Distribution polish

One prompt each, in this order:
1. **Deep links + OG**: `#page=<id>` opens that page directly; per-edition
   OG title/description/image so WhatsApp/X links unfurl properly.
2. **PWA**: manifest + service worker caching the latest published edition
   (offline reading), installable on Android/iOS.
3. **Share-as-image**: a share button on the toolbar renders the current
   page to a PNG (html-to-image) branded with the footer chrome, and invokes
   the Web Share API (falls back to download).
4. **Briefing mode**: a ▶ button that auto-flips every 9 seconds with a thin
   progress bar (16 pages ≈ the 2-minute promise); pauses on interaction;
   honors prefers-reduced-motion.
5. **Mobile legibility**: in single-page mode, allow double-tap to zoom to
   150% with panning, or bump the base type scale ~20% under 480px width.

---

## Standing instructions (repeat if Lovable drifts)

- The Knowledge file (DESIGN-SPEC.md) overrides any aesthetic instinct:
  ivory/ink/gold only, Google Sans Flex only, meaning-colored deltas,
  declared bar ceilings, SAMPLE flags, even page count.
- `reference/index.html` is the visual ground truth. "Match the reference"
  is a complete bug report.
- Numbers come from sources or the editor — never from an LLM.
- Never remove the review gate from the publishing path for Headlines.

## Sequencing with real-world dependencies

| When | Action |
|---|---|
| Now | Phase 0 + Phase 1 |
| Same week | Email CediRates for API access (longest lead item); create TomTom + API-Football accounts; set up Resend |
| After Phase 1 passes | Phase 2, then Phase 3 collectors one by one (Weather → Traffic → GSE first — free and keyless/easy) |
| Before launch | Phase 4; recruit 2–3 street-price market contacts; custom domain (e.g. cediboard.finexskillshub.com) in Lovable settings |
