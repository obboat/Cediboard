# Cediboard Macro Watch redesign: Lovable prompt

One message. Paste the text inside the grey code block into Lovable.

Why: the Macro Watch chart only holds months that arrived in a Databank recap
(so far August and September 2026), leaving two bars and a large empty area.
This backfills 2026 from Ghana Statistical Service releases, adds the Bank of
Ghana target band, and fills the page with the one story the numbers tell
together: what inflation does to savers' returns.

Backfill values (headline year-on-year CPI, 2026), as reported from GSS
releases:
Jan 3.8 · Feb 3.3 · Mar 3.2 · Apr 3.4 · May 3.7 · Jun 5.3 · Jul 4.6 · Aug 5.0
(Sep 5.2 comes from the Databank recap.) Aug 5.0 matches the recap's
"previous" column.

---

```
Macro Watch (v2) leaves most of the page empty: the inflation chart only has
months that arrived in a Databank recap (Aug and Sep), and the bottom of the
page is blank. Redesign it. Keep the Databank recap feed, the editor flags
and the old renderer for older editions. 810 x 1080, shared PageHeader and
PageFooter, existing tokens, Google Sans Flex, 48 px padding.

1. INFLATION HISTORY (data)
- Store the monthly series in a table cpi_monthly (month date primary key,
  headline numeric, food numeric null, non_food numeric null, source text,
  confirmed boolean default false). Migration with RLS: staff-only
  read/write, all to service_role, no anon access (readers see the figures
  through the published page content). Show me the policies.
- Seed 2026 headline year-on-year inflation from Ghana Statistical Service
  releases, source "GSS", confirmed = false:
  2026-01 3.8, 2026-02 3.3, 2026-03 3.2, 2026-04 3.4, 2026-05 3.7,
  2026-06 5.3, 2026-07 4.6, 2026-08 5.0.
  Sep 2026 (5.2, food 4.00, non-food 6.20) comes from the recap; also set
  Aug food 3.00 and non-food 6.80 from the recap's "previous" column.
- The recap feed adds each new month as it appears. If a recap's
  "previous" value differs from the stored month, flag it in the editor.
- Editor: a small table of the months with a "Confirm" tick per seeded
  month. The page uses unconfirmed months too; the tick is only for my
  checking.

2. PAGE LAYOUT, top to bottom (content width 714 px)
a. Eyebrow "GHANA MACRO <b>WATCH</b>" (as now).
b. Hero (as now, slightly tighter): "SEPTEMBER 2026 · Y/Y INFLATION",
   5.2% big, RISING / EASING / STEADY pill, "Food 4.0% · Non-food 6.2%".
   Always show one decimal (5.0, not 5).
c. Chart "INFLATION IN 2026, YEAR ON YEAR (%)": one bar per month from
   January of the current year to the latest month (9 bars today). Fixed
   y-axis 0 to 12%, declared with gridlines at 0, 4, 8, 12. A shaded band
   from 6% to 10% in a light gold tint, labelled "Bank of Ghana target
   8 ± 2%". Value label on every bar, one decimal. Latest month gold,
   others the non-current bar colour. Month labels Jan to Sep.
   In January, when only one month of the new year exists, show the last
   12 months instead and title the chart "LAST 12 MONTHS".
d. "WHAT INFLATION DOES TO YOUR MONEY" panel: one row each for 91-day
   T-bill, 182-day T-bill, 364-day T-bill and policy rate:
   name · nominal rate · "real" return = rate minus latest inflation, in
   percentage points, one decimal, signed · a diverging bar around a zero
   line on a fixed scale of -10 to +10 points (declared on the panel).
   Positive emerald, negative coral. Today: 91-day 4.61% gives -0.6 pts
   (coral, "earns less than inflation"); 182-day +1.1; 364-day +4.5;
   policy rate +8.8. Panel headline, computed: if any T-bill is negative,
   "A {91-day} bill now earns less than inflation"; otherwise
   "Every T-bill beats inflation".
   Footnote 11 px: "Real return is the simple difference between the
   rate and headline inflation."
e. Rates row (as now): policy rate, 91 / 182 / 364-day with change in bp.
   Keep "pending MPC confirmation" on the policy rate until I tick it.
f. Remove the empty advert slot on v2 pages unless a sponsor booking is
   attached to this edition.
g. PageFooter source: "Inflation: Ghana Statistical Service (Sep 2026 via
   Databank Research); T-bills and policy rate: Databank Research; target:
   Bank of Ghana."

No em dashes on the page.

QA
- Fits 810 x 1080 (scrollHeight equals clientHeight), no large empty
  areas, single and spread mode; also with 1, 9 and 12 bars.
- Every number traces to cpi_monthly, the recap JSON or an editor edit.
- The real-return arithmetic is shown correctly for today's data:
  4.61 - 5.2 = -0.59, shown as -0.6.
- Older editions still render their old Macro page.
Show me a full-size screenshot.
```
