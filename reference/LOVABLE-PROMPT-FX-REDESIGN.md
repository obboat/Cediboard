# Cediboard FX pages redesign: Lovable prompt

How to use: paste the three messages below into Lovable **one at a time**, in
order. Check the result of each before sending the next. Each message stands
alone, so Lovable does not need this header.

Decisions already made (do not reopen):
- The page reflows the square "Cedi Watch" card into the paper's 810 × 1080 page (3:4).
  The two-page spread just shows two of these pages side by side; there is no spread-specific layout.
- Theme (ivory or charcoal) is a per-page field. The rest of the paper stays ivory.
- Typeface stays Google Sans Flex, the paper's existing font.
- No separate 1080 × 1080 social PNG. Sharing uses the existing per-page share thumbnails.

---

## Message 1 of 3: data source and content schema

```
We are rebuilding the three FX pages (USD, GBP, EUR) of Cediboard. This message
covers data only. Do not change any page design yet.

CEDIRATES API
These endpoints are already used every morning in our daily production and
work as described. None of them needs an API key. Every request MUST send a
normal browser User-Agent header, otherwise the API rejects it as a bot:

User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36

On the first successful fetch, log the HTTP status and one sample record per
endpoint to the server console, so we can confirm the field names in Lovable.
If a response does not have the fields described below, write fetch_error and
keep the previous values. Do not guess a different shape.

1. Today's rates
   GET https://api.cedirates.com/api/v1/rates?baseCurrency={CUR}&limit=500
   Response: { success, data: [...], datetime, pagination }. data mixes companies
   and recent dates. Keep only records whose `date` (UTC-midnight timestamp)
   equals the target day; per company keep the record with the latest
   `lastUpdatedAt`. Each record has buying, selling, mid (null where not
   applicable), company.companyName and company.subCategory.name
   (Bank | Card | Remittance | Crypto | Forex Bureau).

2. A past day's snapshot
   GET https://api.cedirates.com/api/v1/rates?baseCurrency={CUR}&date=YYYY-MM-DD&limit=500

3. Banks average (CediRates' own figure)
   GET https://api.cedirates.com/api/v1/subcategories?baseCurrency={CUR}
   -> find the id whose name is "Bank" (USD is currently 6865b47aaac289249c77aef1,
   but look it up on every fetch).
   GET https://api.cedirates.com/api/v1/rates/average?baseCurrency={CUR}&quoteCurrency=GHS&isToday=true&subCategory={BANK_ID}
   For a past day replace isToday=true with date=YYYY-MM-DD.
   The parameter name is `subCategory`, NOT `subCategoryId`. The older
   subCategoryId parameter is now ignored and silently returns the blended
   all-channel average.
   Use data.selling ONLY if data.subCategoryName === "Bank". Anything else is the
   blended all-channel average: treat the channel as missing. Never compute our
   own mean of individual banks.

All CediRates calls run server-side (a *.server.ts fetcher next to the existing
ones in src/lib/fetchers/), because browsers cannot set User-Agent and CORS
would block them anyway.

CHANNEL MAPPING
| id         | tile name        | source                       | subCategory  | field   |
|------------|------------------|------------------------------|--------------|---------|
| bog        | Bank of Ghana    | companyName "Bank of Ghana"  | Bank         | selling |
| banks      | Banks (average)  | averages endpoint            | Bank         | selling |
| visa       | Visa             | companyName "Visa"           | Card         | selling |
| mastercard | Mastercard       | companyName "Mastercard"     | Card         | selling |
| binance    | Binance P2P      | companyName "Binance P2P"    | Crypto       | mid     |
| aboki      | Aboki            | companyName "Aboki"          | Forex Bureau | selling |
| wu         | Western Union    | companyName "Western Union"  | Remittance   | buying  |
| google     | Google           | newsroom input               |              |         |
| bloomberg  | Bloomberg        | newsroom input               |              |         |
Binance P2P exists for USD only. GBP and EUR have 8 channels, USD has 9.
Western Union only fills `buying`; that is its payout rate per unit.

This REPLACES the current FX sources in src/lib/fetchers/markets.server.ts
(open.er-api, Mastercard settlement, Binance P2P ads, BoG interbank scrape) for
the FX pages. Leave GSE and Macro fetchers alone.

NEW CONTENT SCHEMA (src/lib/edition-content.ts)
Add a versioned FX content type. Keep the old FxContent type and its renderer
so already-published editions (No. 001) still display exactly as before; the
renderer picks the new design only when content.v === 2.

type FxChannelId = "bog"|"banks"|"visa"|"mastercard"|"binance"|"aboki"|"wu"|"google"|"bloomberg";
type FxContentV2 = {
  v: 2;
  code: "USD" | "GBP" | "EUR";            // fixed per page, not editable
  theme: "ivory" | "charcoal";            // default "ivory"
  headline: FxChannelId;                  // default "bog"
  trendDuration: "1w"|"1m"|"3m"|"6m"|"1y"|"2y";  // default "1m"
  compareDate: string;                    // YYYY-MM-DD, default = edition date minus 1 calendar day (Africa/Accra)
  googleRate: number | null;              // required to publish
  bloombergRate: number | null;           // required to publish
  prevGoogle: number | null;              // optional
  prevBloomberg: number | null;           // optional
  rangeConfirmed: boolean;                // editor ticked "yes, this rate is right" for an out-of-band value
  channels: {
    id: FxChannelId;
    value: number | null;                 // today, rounded
    prev: number | null;                  // on compareDate, rounded
    lastUpdatedAt: string | null;         // ISO, from CediRates
    status: "ok" | "missing" | "stale";
    override?: number | null;             // Manual-mode correction, wins over value
  }[];
  trend: { channel: FxChannelId; points: { date: string; value: number }[] };
  fetchedAt: string | null;
  sample: boolean;
};
The edition date is the publication DATE. Do not add a separate date field.

FETCH RULES
- Today = the edition's date in Africa/Accra time.
- Retry a failed or success:false call once. If it fails again, keep the
  page's previous values, write fetch_error (the existing column) and stop.
  Never fill a gap with an estimate.
- A channel with no record for today: status "missing".
- Freshness: lastUpdatedAt more than 3 days before today -> status "stale".
  Exception: Western Union is never stale as long as its `date` is today.
- prev values come from the compareDate snapshot (endpoint 2, and endpoint 3
  with date=). Do NOT compute FX deltas from the previous edition: editions
  are started by hand and the previous one may be days old.
- Rounding: 2 decimals, half up, with decimal arithmetic (decimal.js or an
  integer-scaling helper), never float Math.round. Add a unit test:
  12.065 -> 12.07, 11.735 -> 11.74, 10.664 -> 10.66.

TREND HISTORY
- Follows the headline channel. If the headline is google or bloomberg (no
  history), the trend uses bog and trend.channel records that.
- Sampling by duration:
  1w, 1m: one point per published session; skip a day whose value AND
          lastUpdatedAt equal the previous day's (weekends, holidays).
  3m, 6m: two points per week (Monday and Thursday).
  1y: one point per week (Monday). 2y: one point per month (1st).
- If a sample date has no record, step back one day at a time, up to 3 days,
  then drop the point.
- The last point is always today's live value.
- Past snapshots never change, so cache them: create a table
  fx_rate_snapshots (currency text, day date, payload jsonb, fetched_at,
  primary key (currency, day)), readable by staff only. Read from it first and
  only call CediRates for days not cached. Limit to 4 concurrent requests.

WHEN TO FETCH
- "Fetch latest" on the page (and the existing "Fetch all") fetches today,
  the compareDate snapshot and the trend.
- Changing headline or trendDuration re-fetches only the trend.
- Changing compareDate re-fetches only the compare snapshot.
- Changing Google, Bloomberg or theme re-renders instantly with no fetch.

SEED
For edition creation (edition-create.server.ts clone), copy theme, headline,
trendDuration from the previous edition's same page. Set compareDate to the new
edition date minus one day. Leave googleRate and bloombergRate empty. If the
previous edition's date equals the new compareDate, prefill prevGoogle and
prevBloomberg with its googleRate and bloombergRate.

When done, show me the logged sample records and the raw stored content for
the USD page after one "Fetch latest". No design changes yet.
```

---

## Message 2 of 3: the page design (810 × 1080)

```
Now redesign the FX page component (src/components/cediboard/pages/FxPage.tsx)
for content.v === 2. Keep the old renderer for older editions. The page is
810 x 1080, inside the existing Book flip engine; do not touch the flip engine
or page size. In spread mode it is simply one of two pages side by side.

Keep the shared chrome: PageHeader (eyebrow + FINEX INSIGHTS wordmark) and
PageFooter (source line, data-as-of stamp, flag, handles, folio). Use the
existing 48 px side padding, existing tokens and Google Sans Flex. No new hues.

THEME
theme "charcoal" adds the existing .page.dark class and these tokens;
"ivory" uses the paper's normal tokens.
| token | ivory   | charcoal |
| bg    | #f5f3ee | #1a1814  |
| ink   | #1a1814 | #f5f3ee  |
| muted | #6a6458 | #a39d90  |
| tile  | #ece9e1 | #26231e  |
| track | #e4e1d8 | #3a362f  |
| grid  | #d9d5ca | #3a362f  |
Accents in both themes: gold #c8960c, emerald #3BA874, coral #E05040.
The share thumbnail capture route must render the chosen theme.

LAYOUT, top to bottom (content width 714 px)
1. PageHeader eyebrow: "CEDI WATCH · <b>{CUR} → GHS</b>" (gold pair).
2. Title block (top 14 px):
   - "WHAT A {DOLLAR|POUND|EURO} COSTS TODAY", 34 px, 700, one line.
   - Subtitle 15 px muted: "Cedis per {US$1|£1|€1} · {n} channels ·
     **{6 Oct 2026}** vs **{29 Sep 2026}**" (dates bold ink, written D Mon YYYY,
     n spelled out and equal to tiles shown + hero).
   - Legend on one line, 12.5 px: coral ▲ rate up · cedi weaker,
     emerald ▼ rate down · cedi stronger, gold ● no change.
3. Hero row (top 20 px, gap 24 px):
   LEFT, 250 px wide, 4 px gold top rule:
   - Label 13 px uppercase muted, letter-spaced, from the headline channel:
     bog OFFICIAL RATE, aboki STREET RATE, visa/mastercard CARD RATE,
     banks BANK RATE, binance CRYPTO RATE, wu REMITTANCE RATE,
     google/bloomberg REFERENCE RATE.
   - Name and type, 19 px bold, e.g. "Bank of Ghana · selling".
   - Value: "₵" (34 px) + value (80 px, 700, gold), never wraps.
   - Delta, 16 px bold: "▲ 0.10 vs 29 Sep 2026".
   - One sentence, 13.5 px muted:
     if hero is bog: "The anchor every other channel is measured against.
     Street and card rates sit {lo}% to {hi}% above it today." where the range
     covers Aboki, Visa and Mastercard premiums, lo rounded down, hi rounded up.
     Otherwise: "{Name} sits {premium} {above|below} the Bank of Ghana official
     rate today."
   RIGHT, remaining width (~440 px), 4 px ink top rule:
   - Line 1: "{CHANNEL} RATE · LAST {1 WEEK|1 MONTH|...}" 13 px muted letter-spaced.
     If the trend fell back to bog, say "OFFICIAL RATE" here.
   - Line 2: trend headline, e.g. coral "▲ ₵0.37" + "+3.2% in a month"
     (arrow colour by the delta rules).
   - SVG line chart ~440 x 220: gold 3.5 px line, 12% gold area fill,
     horizontal gridlines with right-aligned values (step 0.5, 0.25 or 0.1 by
     range, aim for 4 to 6 lines), hollow start dot with its ₵ value above,
     filled gold end dot with its ₵ value to its left, four evenly spaced date
     labels on the x axis. Value labels must never overlap the line: if they
     would, move the label below the dot.
4. Tile grid header (top 22 px): left "{EIGHT|SEVEN|...} OTHER CHANNELS ·
   PER {US$1}" 13 px letter-spaced; right "arrows vs {29 Sep 2026} · bars to a
   ₵{ceiling} ceiling" 12 px muted.
5. Tile grid: 4 columns, 10 px gap (tiles ~171 px wide), 6 px radius, tile
   colour, sorted by value high to low. Each tile:
   - name 16 px bold;
   - type 9.5 px uppercase muted, one line, using these short labels:
     bog OFFICIAL · SELLING, banks BANKS · AVG SELL, visa CARD · SELLING,
     mastercard CARD · SELLING, binance CRYPTO · MID, aboki BUREAU · SELLING,
     wu REMITTANCE · PAYOUT, google REF · MID-MARKET, bloomberg REF · MID-MARKET;
   - value "₵" + value, 30 px bold, one line;
   - row: delta left (12.5 px), premium right ("+4.5% vs BoG", 1 decimal,
     signed with a real minus sign). The bog tile shows "official anchor".
     google/bloomberg with no prev value show muted "no prior", no arrow.
   - 5 px bar on the track colour: width = value / ceiling, minimum 4%.
     Ceilings are fixed: USD 13.00, EUR 15.00, GBP 18.00. Never scale to the
     largest value. The highest tile's value and bar are coral; others ink.
   Missing channels get no tile.
6. Note line, 13 px muted, max 2 lines: trend date range and what each point
   is; "Google and Bloomberg are reference rates entered by the newsroom";
   any missing channel ("Binance P2P not published today") or stale one
   ("Aboki (unverified as fresh)"); any Manual override ("Visa corrected by
   the newsroom").
7. PageFooter with source: "Source: CediRates (live channels and {channel}
   history); Bank of Ghana; Google & Bloomberg reference rates · {6 Oct 2026}
   vs {29 Sep 2026}."

DELTA RULES
delta = today minus compareDate. > 0 coral ▲ (cedi weaker), < 0 emerald ▼
(cedi stronger), = 0 gold ●. Premium = (channel / bog − 1) × 100.

Show the SAMPLE chip only when content.sample is true.
No em dashes anywhere on the page.

Before saying you are done, render USD (charcoal, headline bog, 1 month), GBP
(ivory, headline aboki, 3 months) and EUR (ivory, headline google, 1 week) and
show me screenshots of each at 810 x 1080 and in spread mode.
```

---

## Message 3 of 3: newsroom fields, publish guard and QA

```
Now expose the FX options as fields in the newsroom page editor for FX pages.
They live in a "Board settings" panel at the top of the FX page editor and are
visible in Auto AND Manual mode (Insert mode keeps its current editor).

FIELDS
| Field                   | Control                         | Default                          | Rules |
|-------------------------|---------------------------------|----------------------------------|-------|
| Currency                | read-only label                 | the page's currency              | Not editable. |
| Theme                   | segmented: Ivory / Charcoal     | carried from previous edition    | Instant preview. |
| Headline channel        | dropdown of the 9 channels      | carried, else Official (BoG)     | Binance P2P hidden for GBP/EUR. Refetches trend. |
| Trend duration          | dropdown 1 week to 2 years      | carried, else 1 month            | Refetches trend. |
| Publication date        | read-only, from the edition     | edition date                     | Not a field. |
| Compare date            | date picker                     | edition date minus 1 day         | Must be before the edition date. Refetches compare snapshot. |
| Google rate             | number, 2 decimals, GH₵         | empty                            | Required to publish. |
| Bloomberg rate          | number, 2 decimals, GH₵         | empty                            | Required to publish. |
| Google on compare date  | number, optional                | prefilled only if previous edition date = compare date | Empty -> "no prior" on the tile. |
| Bloomberg on compare date | number, optional              | same as above                    | Same. |

Plausibility bands for Google and Bloomberg: USD 9 to 14, EUR 10 to 17,
GBP 12 to 20. Outside the band, show an inline warning with a "Yes, this rate
is correct" checkbox that sets rangeConfirmed. Use the typed numbers exactly;
never replace them with an API figure.

Under the settings panel:
- Auto mode: the existing AutoSummary card lists every channel with value,
  prev, lastUpdatedAt and an ok / missing / stale badge, plus "Fetch latest".
- Manual mode: the same list with an override input per channel (value only).
  An override is saved in channels[].override, wins on the page, and is named
  in the note line.

TODAY'S DESK
Add checklist items per visible FX page when needed:
"USD board: enter Google and Bloomberg rates", "GBP board: confirm Google rate
(outside usual range)", "EUR board: CediRates fetch failed, re-fetch",
"USD board: Aboki is stale".

PUBLISH GUARD
Publish stays blocked while any VISIBLE FX v2 page has: an empty Google or
Bloomberg rate, an unconfirmed out-of-band rate, a failed last fetch, a fetch
for a date other than the edition date, or a missing Bank of Ghana value (it
is the anchor for every premium). The editor can still hide the page (the
existing even-page guard applies).

QA (check these yourself before telling me it's done, and list the results)
- Each page's content fits 810 x 1080: scrollHeight equals clientHeight, the
  footer is fully inside the page, in both themes and all three currencies.
- Hero and every tile value stay on one line at the largest plausible value
  (USD 13.99, EUR 16.99, GBP 19.99).
- Arrow colours follow the delta rules; flat values show the gold dot.
- Bars use the fixed ceiling for the currency.
- Subtitle channel count = tiles + hero, also when a channel is missing.
- Source line names CediRates, Bank of Ghana and the Google & Bloomberg
  reference rates with both dates.
- No em dashes on the page.
- Every figure traces to the stored CediRates payload or a newsroom field.
- Edition No. 001 still renders its old FX pages unchanged.
- Regenerate the share thumbnails for the FX pages after the redesign.
```
