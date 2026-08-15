# Cediboard → Lovable: the plain-English guide

This is the beginner's companion to `LOVABLE-HANDOVER.md` (the technical
playbook with the exact prompts to copy-paste). Read this first; copy
prompts from the other file when told to.

**The big picture:** you're hiring a builder (Lovable) to turn the model
house we made (`index.html`) into a real house people can visit online —
then setting up a delivery system that restocks it with fresh news every
morning before 9am.

---

## Jargon, translated

| Word | What it actually is |
|---|---|
| **Lovable** | A website builder you talk to in plain English. You type instructions, it builds and hosts the app. |
| **Knowledge file** | Lovable's "brand manual." Paste text there once and it re-reads it before every job. Ours is `DESIGN-SPEC.md`. |
| **GitHub** | A shared filing cabinet for code in the cloud. We put our finished newspaper in it so Lovable copies the real thing instead of imagining it. |
| **Supabase** | The app's filing room — a database where each day's edition is stored as one record. |
| **API** | A data tap another company lets your app drink from (weather, traffic, rates). |
| **API key** | Your membership card for that tap. |

---

## Step 1 — Set the table (~30 minutes, once)

1. Go to **lovable.dev** → create an account → **New Project** → name it
   Cediboard.
2. **Settings → Knowledge**: open `DESIGN-SPEC.md` from this folder
   (Notepad/Word is fine), select all, copy, paste into the Knowledge box,
   save. *You just handed the builder the brand manual.*
3. Click Lovable's **GitHub → Connect** (create a free GitHub account when
   asked). This creates the filing cabinet.
4. Put our files in the cabinet — the one fiddly step. Easiest routes:
   - GitHub's website → your new repo → **Add file → Upload files** → drag
     in `index.html`, the `assets` folder, `DESIGN-SPEC.md` and
     `LOVABLE-HANDOVER.md`, inside a folder named `reference`; **or**
   - open a Claude Code session in this folder and say *"push the reference
     files to my Lovable GitHub repo"* — it can do it for you.
5. Lovable → **Integrations → Supabase → Connect** → let it create a new
   Supabase project (accept defaults).

**Checkpoint:** Lovable has the brand manual, the real newspaper to copy,
and a filing room. Nothing is built yet.

---

## Step 2 — Have it build the paper (an afternoon)

Copy the **Phase 1 prompt** from `LOVABLE-HANDOVER.md`, paste into Lovable's
chat, send, and wait for the preview.

**Your only job is comparing.** Open our original (double-click
`index.html`) next to Lovable's preview:

- Do pages **flip** the slow, elegant way — or slide like a photo carousel?
- Same cream paper, same gold, same fonts?
- All 16 pages? SAMPLE badges showing? Ad slots with working phone links?

If something's off, no technical words needed — type:
*"The flip animation doesn't match reference/index.html — re-read it and
copy it exactly."* Expect 2–4 correction rounds; that's normal.

**Do not move to Step 3 until the copy is faithful.** Everything else
builds on this.

---

## Step 3 — Give it a memory (an evening)

Paste the **Phase 2 prompt**. This creates:

- the shelf where each day's edition lives (one record per day),
- your private **admin page** — the editor's desk — to review tomorrow's
  draft, fix anything, and press **Publish**,
- a login so only you get in.

**Checkpoint test:** log into admin, change one number in the draft, press
Publish, confirm the public paper changed. If yes, the plumbing works.

---

## Step 4 — Hire the delivery boys (a weekend, one at a time)

The automation: robots that wake at 5:45am, fetch fresh data, and fill in a
draft edition before you're up.

1. Paste the **Phase 3 prompt** — it hires the first three robots
   (**weather, traffic, stock exchange**), chosen because they're free and
   need nobody's permission. Test each from the admin page: does the
   weather page show today's *actual* forecast?
2. Add the rest **one at a time** with the follow-up prompts in the
   playbook: currency rates, fuel, headlines, trending, sports.
3. Two things only YOU can do (start early — they have waiting time):
   - **Email CediRates** asking for API access. Mention you're Finex
     Insights and credit them on every board.
   - Create free **TomTom** (traffic) and **API-Football** (sports)
     accounts; each gives you a key to paste into Lovable when it asks.

**The morning routine you end up with:** an 8am email — "Weather ✅,
Traffic ✅, Rates ✅, Headlines: please review" — with a link. You glance at
the draft on your phone, tweak a headline, press Publish. Done by 9. If you
oversleep, it publishes anyway with small "as of yesterday" tags on
anything stale.

---

## Step 5 — Make it spread (a weekend)

Four short prompts from **Phase 4**, in order:

1. Links that open a specific page (for WhatsApp sharing).
2. "Install as an app" on phones, works offline.
3. A **share-this-page-as-image** button — the audience-growth feature;
   people forward the rates board on WhatsApp with your brand on it.
4. The ▶ auto-play "2-minute briefing" mode.

---

## Costs & timeline (realistic)

- **Money:** Lovable ~$25/mo · Supabase free tier · weather free · TomTom
  free · API-Football $0–30/mo · CediRates hopefully free with attribution
  · Claude API for headline drafts: a few dollars/month.
- **Time:** 2–3 weeks of evenings, mostly Steps 2 and 4.

## Golden rules

1. **One thing per prompt.** Test it, then the next.
2. **Never accept "close enough" in Step 2.** Point it back to the
   reference file until it matches.
3. **Numbers come from sources or from you — never from an AI.** The AI
   only writes prose, and headlines always wait for your approval.
4. **Stuck?** Open Claude Code, say what Lovable did, paste a screenshot —
   it will write the exact correction prompt for you.
