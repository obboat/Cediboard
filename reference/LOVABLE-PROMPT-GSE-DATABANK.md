# Cediboard GSE Scoreboard from the Databank Recap: Lovable prompt

How to use: paste the three messages below into Lovable **one at a time**, in
order, and check each result before sending the next. Copy only the text inside
each grey code block. Attach the sample PDF named in each message.

Decisions already made (do not reopen):
- Source: the Databank Research "Ghana: Daily Market Recap" PDF, emailed daily
  after the close by research@databankgroup.com. Finex Insights has an
  agreement with Databank to republish these figures with credit.
- Intake: the editor uploads the PDFs in the newsroom (no email service, no
  Gmail connection). Automatic forwarding can be added later with a Google
  Apps Script that pushes the PDFs to the same upload path.
- The Recap feeds the GSE page AND Macro Watch (T-bill rates and inflation).
- The Return Analysis Sheet (the second daily PDF) is stored but not used yet.

---

## Message 1 of 3 (upload only, revised 28 Oct 2026): getting the PDF into the app

If Lovable has already started on inbound email or Gmail access, this message
tells it to remove that.

```
We are rebuilding the GSE Scoreboard (page type "gse") on a new source: the
Databank Research "Ghana: Daily Market Recap" PDF, which Databank emails to
me every trading day after the close. Finex Insights has an agreement with
Databank to republish its figures with the credit "Source: Databank
Research". This message covers intake only.

INTAKE: MANUAL UPLOAD ONLY
- No inbound email service, no Mailgun or Postmark, and no Gmail
  connection. The app must never read my mailbox. If any of this was
  started, remove it, including any stored Gmail tokens or connector.
- In the GSE editor, add an "Upload Databank PDFs" drop zone that accepts
  one or more PDFs at once (I will usually drop both daily files together).
- Save each file to a private storage bucket "market-reports". Classify by
  file name: "Ghana_Daily_Market_Review*" or "*Daily Market*" = recap;
  "*Return_Analysis*" = return-sheet; anything else = other (stored, not
  processed).
- Extraction (Message 2) starts automatically right after upload.

STORAGE
Create a table market_reports (id uuid, uploaded_at timestamptz,
uploaded_by uuid, kind text ('recap'|'return-sheet'|'other'), file_path
text, report_date date null, status text
('received'|'extracted'|'checks_failed'|'error'), extracted jsonb null,
checks jsonb null, error text null). Unique on (kind, report_date) once
report_date is known: uploading the same day's recap again asks "Replace
the {27 Oct} recap?" before overwriting.
Add a migration in supabase/migrations: RLS on, staff-only read/write via
public.is_staff(auth.uid()), all to service_role, no anon access. Same rules
for the storage bucket. Show me the policies.

NEWSROOM "INTAKE" PANEL (in the GSE editor, under the drop zone)
List the last 10 uploads: time, who, kind, report date, status, and a link
to open the PDF.

When done, I will upload one recap as a test. Do not extract anything yet.
```

---

## Message 2 of 3: reading the Recap and checking it

Attach the sample PDF "Ghana_Daily_Market_Review_-_27th_October_2026.pdf".

```
Now extract the figures from each recap PDF. The attached file is a real
example (27 October 2026). Only page 1 is needed. Pages 2 to 5 hold
dividends, upcoming events, an Africa markets table, contacts and
disclaimers.

EXTRACTION
- Server-side, when a recap is uploaded, send page 1 of the
  PDF to Claude with the existing ANTHROPIC_API_KEY. Use the official
  @anthropic-ai/sdk, model "claude-opus-5-5", the PDF as a document content
  block, and structured outputs (output_config.format with a JSON schema).
  Set output_config.effort to "medium". Check stop_reason before reading;
  on refusal or error set status 'error' with the message. Follow the
  current Anthropic API docs for exact parameter shapes.
- Instruction to Claude: copy numbers exactly as printed; never compute or
  guess; use null for anything not printed.

SCHEMA (store in market_reports.extracted)
{
  reportDate: "YYYY-MM-DD",                       // top right of page 1
  stats: {                                         // "GSE MARKET STATISTICS SUMMARY"
    gseCi:   { current, previous, pctChange },
    gseFsi:  { current, previous, pctChange },
    dsi20:   { current, previous, pctChange },
    marketCapGhsM: { current, previous, pctChange },
    ytdGseCi: number, ytdGseFsi: number, ytdDsi20: number,   // percent
    volumeShares:  { current, previous, pctChange },
    turnoverGhs:   { current, previous, pctChange },
    countersTraded:{ current, previous }
  },
  gainers:  [{ ticker, price, change, pct }],      // "Top Gainers", all rows
  laggards: [{ ticker, price, change, pct }],      // "Worst Laggards", all rows
  topByValue:  [{ ticker, price, valueGhs }],      // "Top Traders by Value"
  topByVolume: [{ ticker, price, volume }],        // "Top Traders by Volume"
  tbills: { d91: {current, previous}, d182: {...}, d364: {...} },  // percent
  inflation: { month: "YYYY-MM", headline: {current, previous},
               food: {current, previous}, nonFood: {current, previous} },
  policyRate: number | null   // page 2 Africa table, Ghana row, "Policy Rate%"
}
Laggard change and pct are negative numbers. For the 27 Oct sample, expect
gseCi.current 14052.96, previous 14014.16, pctChange 0.28, ytdGseCi 60.23,
turnoverGhs.current 2004537, countersTraded.current 27, gainers GOIL first
(5.50, 0.24, 4.56), tbills.d91.current 4.61, inflation.month "2026-09",
headline.current 5.20, policyRate 14.00.

CHECKS (store pass/fail per check in market_reports.checks)
1. For gseCi, gseFsi, dsi20: (current - previous) / previous x 100 rounds
   to pctChange within 0.01.
2. reportDate is a weekday, not in the future, and not older than 4 days at
   receipt.
3. Every gainer pct > 0 and every laggard pct < 0; gainers sorted high to
   low, laggards low to high.
4. countersTraded.current >= number of gainers + laggards.
5. Sum of topByValue valueGhs <= turnoverGhs.current.
6. If a return-sheet for the same date exists, its "Market Aggregate"
   turnover equals turnoverGhs.current within GH₵1 (skip if absent).
All pass: status 'extracted'. Any fail: status 'checks_failed'; the data is
kept but NOT used on any page until an editor clicks "Use anyway" or fixes
a value in the editor.

GSE HISTORY
Also upsert into gse_daily (report_date date primary key, gse_ci, gse_fsi,
dsi20, market_cap_ghs_m, volume, turnover_ghs, ytd_gse_ci, source_report
uuid). Same RLS pattern as above, with a migration.

Run it on the attached sample and show me the extracted JSON and the check
results. No page changes yet.
```

---

## Message 3 of 3: the GSE page, Macro Watch feed, editor and publish guard

```
Now use the extracted recap on the GSE page and in Macro Watch.

WHICH RECAP
Each edition uses the latest recap with status 'extracted' (or "Use
anyway") whose reportDate is on or before the edition date. Store its id
on the page content so a published edition never changes afterwards.
"Fetch latest" on the GSE page picks it up; also wire the GSE page into
"Fetch all". The old kwayisi and gse.com.gh scraping in fetchGse is no
longer used by v2 pages; leave the code for older editions.

GSE PAGE (src/components/cediboard/pages/Gse.tsx, new content.v === 2;
keep the old renderer for older editions). 810 x 1080, shared PageHeader
and PageFooter, existing tokens, Google Sans Flex, 48 px padding.
One idea per page. Top to bottom:
1. Eyebrow "GHANA MARKET <b>PULSE</b>". Title "THE GSE AT THE CLOSE",
   34 px, 700. Subtitle 15 px muted: "Close of {Tuesday 27 Oct 2026}".
2. Hero: label "GSE COMPOSITE INDEX"; level "14,052.96" 80 px gold; beside
   it the day change "▲ 38.80 pts (+0.28%)" (colour by direction: up is
   emerald here, since a rising index is good news for holders) and
   "+60.23% this year" 20 px bold.
3. Context line, 15 px: "Financials (GSE-FSI) 7,358.62, +0.07% · Market
   cap GH₵264.96bn · Turnover GH₵2.00m, -14.54%".
4. Breadth strip: one full-width bar split into three, widths proportional
   to counts: gainers (emerald), laggards (coral), traded unchanged (track
   colour) = countersTraded - gainers - laggards. Label under it:
   "4 up · 5 down · 18 traded flat · 27 stocks traded".
5. Movers: two columns, "GAINERS" and "LAGGARDS", every row from the recap
   (ticker bold, price, pct with arrow). If a side is empty, show "No
   {gainers} today".
6. Where the money went: "Top 5 by value", one row per stock with a bar
   showing its share of the day's turnover against a fixed 100% ceiling
   (label "share of turnover"), value in GH₵ and the % share.
7. One-sentence story, 15 px: written by Claude from the extracted numbers
   in our own words (do not copy Databank's prose), e.g. "MTN Ghana and
   Kasapreko took 59% of the day's turnover." Editable in the editor;
   regenerate button.
8. PageFooter source: "Source: Databank Research, Ghana Daily Market Recap
   ({27 Oct 2026}); Ghana Stock Exchange."
Remove the old KPI tiles, the "share of top trader" bars and the Finex
View box. No em dashes on the page.

MACRO WATCH FEED
From the same recap, update the Macro Watch page content on fetch:
- T-bill cells: 91-day, 182-day and 364-day rates with change in basis
  points vs the recap's "previous" column (▼ 1bp emerald / ▲ coral,
  "no change" gold). Detail line "Primary auction, via Databank Research".
- Inflation: when inflation.month is newer than the last month in the
  12-month bar series, append it and drop the oldest; set the big number to
  headline.current, the label to "{SEPTEMBER 2026} · Y/Y INFLATION", and
  the pill to RISING / EASING / STEADY from current vs previous. Food and
  non-food go in the detail line. Never overwrite a month already in the
  series with a different value without flagging it in the editor.
- Policy rate: if policyRate is present and differs from the page's policy
  rate cell, update it and flag it in the editor for a quick check (the
  Africa table on page 2 is less authoritative than the MPC statement).
- Macro source line adds "Databank Research (T-bills, inflation)".
Keep the Bank of Ghana T-bill fetcher as a fallback only when no recap is
available for the last 7 days.

EDITOR
GSE editor: the Intake panel from Message 1; the chosen recap with its
check results (green ticks, red crosses with the failing numbers); every
extracted value editable (edits are logged); "Use anyway" for a recap with
failed checks; regenerate story.

TODAY'S DESK (only when something needs doing)
- "GSE: upload the Databank recap for {Tue 27 Oct}" from 07:00 Accra on a
  weekday after a trading day, if none has been uploaded (on public
  holidays the GSE does not trade and there is no recap).
- "GSE: recap of {27 Oct} failed {n} checks".
- "Macro: policy rate changed in the recap, please confirm".

PUBLISH GUARD
Block publish while the visible GSE page uses a recap with failed checks
that has not been "Used anyway", or has no recap at all.

QA
- Fits 810 x 1080 (scrollHeight equals clientHeight) with 0, 5 and 10
  movers per side, single and spread mode.
- Every number on the GSE and Macro pages traces to the extracted JSON or
  an editor edit.
- Older editions still render their old GSE and Macro pages.
- Regenerate both share thumbnails.
Show me screenshots of both pages built from the 27 Oct sample.
```
