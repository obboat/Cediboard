# Cediboard Fuel page redesign: Lovable prompt

How to use: paste the three messages below into Lovable **one at a time**, in
order, and check each result before sending the next. Copy only the text inside
each grey code block.

If you already know the CediRates fuel endpoint (for example from your Claude
Project), paste it into Message 1 where it says FUEL ENDPOINT and Lovable can
skip the discovery step.

Decisions already made (do not reopen):
- CediRates is the base source for every OMC price. No routine typing.
- Show every OMC that CediRates lists, sorted by petrol price.
- Products: petrol and diesel (per litre).
- The app flags only OMCs whose CediRates price looks stale: any OMC priced
  below the current NPA floor, plus a short core list (GOIL, Star Oil, Shell,
  TotalEnergies) when unchanged 24 hours into a window. An editor can
  override a single price; the override clears itself as soon as CediRates
  shows a newer update for that OMC.
- The CediRates fuel endpoint was found by Lovable on 19 Oct 2026 and is
  filled into Message 1.

---

## Message 1 of 3: CediRates fuel data, staleness and history

```
We are rebuilding the Fuel page (page type "fuel"). Today it has no fetcher:
every price is typed in and the page shows a hard-coded SAMPLE badge. This
message covers data only. Do not change the page design yet.

SOURCE: CEDIRATES
CediRates (cedirates.com/fuel-prices/gh) lists pump prices by OMC. We already
use its public API for FX from a server fetcher, with these rules, which also
apply here: no API key; every request sends a normal browser User-Agent
header, otherwise the API rejects it as a bot:
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/124.0.0.0 Safari/537.36

FUEL ENDPOINT (found and confirmed):
GET https://api.cedirates.com/api/v1/prices?isToday=true
Response: { success, data: [...], datetime, pagination }. One record per
OMC: company.companyName (display), company.uniqueId and companyId (stable
IDs, match OMCs over time by company.uniqueId), petrol and diesel (GH₵ per
litre), lastUpdatedAt (the OMC's last price change). date, createdAt and
updatedAt are CediRates' daily copy times: never use them as freshness.
Ignore "premium".
Fallback: if the main call fails twice, try once
GET https://api.cedirates.com/api/v1/fuelPrices/<d-M-yyyy> (Accra date) and
accept it only if its records have the same fields. Record which endpoint
supplied the data (fetchedFrom).

STEP 0: FIND THE ENDPOINT (only if FUEL ENDPOINT above is empty)
The FX endpoints live under https://api.cedirates.com/api/v1/. Find the
endpoint that the cedirates.com fuel prices page uses (for example by
reading the page's JavaScript for api.cedirates.com URLs). Call it once from
a server function and show me: the URL, HTTP status, top-level keys, and two
sample records. Stop there and wait for my OK before building on it. Do not
guess field meanings: tell me which fields hold the OMC name, petrol price,
diesel price, and the last-updated time.

FETCHER
Add fetchFuel in a new src/lib/fetchers/fuel.server.ts, wired into the
existing run.server.ts and AUTO_FETCH_TYPES so "Fetch latest" and "Fetch all"
include it. Default mode for the Fuel page becomes Auto.
- Keep every OMC CediRates returns for Ghana, with petrol and diesel prices
  per litre and that OMC's last-updated time. Keep the latest record per OMC.
- Round to 2 decimals, half up, with decimal arithmetic (the same helper as
  the FX pages).
- Retry a failed call once. If it fails again, keep the previous values,
  write fetch_error and stop. Never fill a gap with an estimate.
- An OMC that has a petrol price but no diesel price (or the reverse) keeps
  the missing one empty, shown as a dash.

PRICING WINDOWS
Ghana prices are reviewed in two-week windows starting on the 1st and 16th
of each month (Africa/Accra time). windowStart = the most recent 1st or 16th
on or before today.

STALENESS (decides what the editor is asked to check)
lastUpdatedAt is the OMC's last price CHANGE, so an old date alone does not
mean CediRates is out of date. A row is "stale" (needs a look) when EITHER
a) the NPA floor for the current window is known AND the row's petrol is
   below floorPetrol or its diesel is below floorDiesel; OR
b) the OMC is on the CORE list AND lastUpdatedAt is before windowStart AND
   at least 24 hours have passed since windowStart.
CORE list: a newsroom setting, default GOIL, Star Oil, Shell, TotalEnergies,
matched by company.uniqueId. Store the reason ("below floor" / "core
unchanged") with the flag.
Stale is a prompt for the editor, not an error: the row still shows its
latest CediRates price. Every row whose lastUpdatedAt is before windowStart
shows "unchanged since <date>" on the public page (see Message 2).

HISTORY AND DELTAS
Create a table fuel_price_snapshots (omc text, day date, petrol numeric,
diesel numeric, source text, updated_at timestamptz, primary key (omc, day)),
staff-readable only. On every fetch, upsert today's row per OMC.
Delta per product = current price minus that OMC's last price in the
PREVIOUS window (the latest snapshot dated before windowStart). No previous
snapshot: show "new", no arrow.

NPA PRICE FLOOR (optional)
Add two optional fields, floorPetrol and floorDiesel, with floorWindow (the
windowStart they belong to). They are carried forward on clone. When
floorWindow is not the current window, treat the floor as unknown and hide it.

CONTENT SCHEMA (src/lib/edition-content.ts)
Add a versioned type; keep the old FuelContent and its renderer so older
editions still display.
type FuelContentV2 = {
  v: 2;
  windowStart: string;                 // YYYY-MM-DD
  fetchedAt: string | null;
  floorPetrol: number | null; floorDiesel: number | null; floorWindow: string | null;
  rows: {
    omc: string;
    petrol: number | null; diesel: number | null;           // from CediRates
    cediUpdatedAt: string | null;                            // CediRates last-updated
    prevPetrol: number | null; prevDiesel: number | null;    // previous window
    stale: boolean;
    confirmedAt?: string | null;       // editor clicked "Still correct" this window
    override?: { petrol: number | null; diesel: number | null;
                 note: string; at: string; by: string } | null;
  }[];
  sample: boolean;
};

OVERRIDE RULE (applied on every fetch)
If a row has an override and CediRates now shows a last-updated time later
than override.at, delete the override: CediRates has caught up. Otherwise
the override stays and wins on the page.
A row with an override or with confirmedAt in the current window is never
stale.

When done, show me the raw stored content for the Fuel page after one
"Fetch latest", with the stale flags. No design changes yet.
```

---

## Message 2 of 3: the page design (810 x 1080)

```
Now redesign the Fuel page component
(src/components/cediboard/pages/Fuel.tsx) for content.v === 2. Keep the old
renderer for older editions. Keep the shared PageHeader and PageFooter, the
existing tokens, Google Sans Flex and 48 px side padding. Do not touch the
Book flip engine or page size. Remove the hard-coded SAMPLE badge; show
SAMPLE only when content.sample is true.

LAYOUT, top to bottom (content width 714 px)
1. PageHeader eyebrow: "FUEL <b>BOARD</b>".
2. Title block:
   - "WHAT A LITRE COSTS TODAY", 34 px, 700.
   - Subtitle 15 px muted: "Pump prices by OMC, GH₵ per litre · pricing
     window from {16 Oct 2026}".
   - Legend, 12.5 px: coral ▲ up since last window, emerald ▼ down,
     gold ● no change.
3. Glance row: four stat tiles, equal width:
   CHEAPEST PETROL (value + OMC), CHEAPEST DIESEL (value + OMC),
   AVERAGE PETROL (simple mean of shown OMCs, labelled "simple average"),
   NPA FLOOR PETROL (value, or hide this tile when the floor is unknown and
   show AVERAGE DIESEL instead).
4. Ranked table, sorted by petrol price low to high (OMCs without a petrol
   price go last). Columns:
   OMC | Petrol: value, delta chip, 5 px bar | Diesel: value, delta chip,
   5 px bar | Status.
   - Bars use fixed ceilings, never the largest value: GH₵22.00 for both
     products, minimum width 4%. When the NPA floor is known, draw a thin
     gold tick on each bar track at the floor value.
   - The cheapest petrol and cheapest diesel values are emerald and bold.
   - Status column, 11 px muted:
     changed this window (lastUpdatedAt on or after windowStart): "as of 17 Oct";
     not changed this window: "unchanged since 2 Oct";
     overridden row: "✎ 17 Oct" (newsroom-updated);
     confirmed row: "checked 17 Oct".
   - Row height adapts so up to 14 OMCs fit in one column. With 15 to 28
     OMCs, switch to two side-by-side columns with the same columns in
     compact form. With more than 28, show the 28 most recently updated
     and say "{n} more on CediRates" in the note line.
5. Note line, 13 px muted, max 2 lines: what the deltas compare against;
   "✎ Prices updated by the newsroom from the OMC's own announcement" when
   any override is shown; "Average is a simple mean, not volume-weighted".
6. PageFooter source: "Source: CediRates (OMC pump prices); NPA (price
   floor); Finex Insights newsroom where marked ✎ · window from {date}."

DELTA RULES
delta = current minus previous-window price. > 0 coral ▲ (dearer),
< 0 emerald ▼ (cheaper), = 0 gold ●. Show the delta to 2 decimals.

No em dashes anywhere on the page.

QA (check before saying you are done)
- Fits 810 x 1080 (scrollHeight equals clientHeight, footer fully inside)
  with 6, 14, 15 and 28 OMCs, in single and spread mode.
- Every value on one line; no clipped OMC names (shorten with a title
  attribute only if longer than 18 characters).
- Bars use the fixed GH₵22.00 ceiling.
- Older editions still render their old Fuel page.
Show me screenshots with today's real data.
```

---

## Message 3 of 3: newsroom editor, overrides and publish guard

```
Now build the Fuel editor in the newsroom (Auto mode summary for fuel
v2 pages). The goal is the least possible manual work: the editor only acts
on rows the app flags.

EDITOR LAYOUT
1. Top bar: window start date, last fetch time, "Fetch latest" button,
   and the count of stale rows ("3 OMCs to check").
2. "Needs a look" list first: only stale rows, each with
   - the CediRates price and its last-updated date;
   - "Still correct" button (sets confirmedAt, one click, no typing);
   - "Update price" button: opens two small inputs (petrol, diesel, either
     may stay blank to keep CediRates' value) and a short required note
     ("GOIL Facebook post 16 Oct"). Saving creates the override with the
     editor's name and time.
3. "All OMCs" table below, collapsed by default, with the same two
   buttons on every row and a "Remove override" link on overridden rows.
4. NPA floor panel: floorPetrol and floorDiesel inputs for the current
   window, prefilled from the last window and marked "last window" until
   the editor saves them. The page hides the floor when unknown, and the
   "below floor" staleness check only runs once it is entered.
5. Core list setting: the OMCs checked when unchanged after a window opens
   (default GOIL, Star Oil, Shell, TotalEnergies).

OVERRIDE BEHAVIOUR
- An override replaces only the product(s) the editor typed.
- It clears itself automatically when CediRates shows a newer update for
  that OMC (rule from Message 1). Show the editor a toast the next time
  they open the page: "CediRates caught up with GOIL; your override was
  removed."
- Every override is logged (omc, old value, new value, note, editor, time)
  in a fuel_overrides table, staff-readable only.

TODAY'S DESK
Add items only when there is something to do:
"Fuel: enter the NPA floor for the 16 Oct window" (until it is entered);
"Fuel: 2 OMCs to check" (links to the "Needs a look" list, with each
reason). Nothing appears on days with no stale rows.

PUBLISH GUARD
Publish is NOT blocked by stale rows (they show honestly as "unchanged
since"). Publish IS blocked when the visible Fuel page has a failed last
fetch, or fewer than 3 OMCs with a petrol price.

QA
- Override GOIL petrol, re-fetch: the override stays while CediRates is
  older and disappears once CediRates' last-updated time is newer.
- "Still correct" removes a row from the stale list until the next window.
- No em dashes in the editor-generated page text.
- Regenerate the Fuel share thumbnail after the redesign.
```
