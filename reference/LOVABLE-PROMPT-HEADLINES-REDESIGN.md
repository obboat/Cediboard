# Cediboard Headlines page redesign: Lovable prompt

How to use: paste the three messages below into Lovable **one at a time**, in
order, and check each result before sending the next. Copy only the text inside
each grey code block.

Attach images:
- **Message 2:** attach 2 or 3 of the manual-production samples (29 Sep, 30 Sep,
  1 Oct, 2 Oct 2026) as the layout reference.
- **Message 3:** attach the same samples again as the illustration-style reference.

Decisions already made (do not reopen):
- The page stays a live React page. Titles, captions and sources are real text;
  only the six illustrations are AI-generated, one per card.
- The redesign covers both the visuals and how the six stories are chosen and written.
- Illustrations are made with OpenAI's image model (the manual production uses ChatGPT).
- The date sits on its own line in gold under the title, with no dash.
- The manual samples are 1200 x 1600, exactly 3:4 like the 810 x 1080 page, so
  the layout scales by about 0.675.

---

## Message 1 of 3: choosing and writing the six stories

```
We are rebuilding the "In the Headlines: 6 Major Stories Shaping Ghana Today"
page (page type "headlines"). This message covers story selection, writing and
the content schema only. Do not change the page design yet.

GOAL
Every edition shows six major Ghana stories written in our house style:
- Title: 3 to 6 words, Title Case, at most 45 characters. Lead with the actor
  and the action. Put the key figure in the title when the figure IS the story
  ("Tullow Loses $196.5m Tax Case", "Hohoe Floods Displace 5,140 Residents").
- Caption: exactly one complete sentence, at most 160 characters, present
  perfect where natural ("has set", "has intercepted"). Name the institution or
  person responsible and include the key figure if there is one.
- No clickbait, no questions, no opinion, no exclamation marks, no em dashes.

SELECTION RULES (Claude does this; an editor approves)
From the RSS items fetched by fetchHeadlines (src/lib/fetchers/media.server.ts),
shortlist up to 15 candidates and rank them. Pick the top six that:
- matter nationally: policy, economy, money, courts, health, security,
  infrastructure, sport results, disasters;
- prefer stories with a concrete figure (amount, count, percentage, date);
- skip anniversaries, congratulations, church/party PR, opinion pieces,
  celebrity gossip and duplicates of the same event from different outlets;
- use at most two stories from the same category.

FACT RULES
- Use only facts present in the RSS item (title + summary). Never invent
  numbers, names or events.
- keyFigure must appear verbatim in the item text. After Claude replies,
  check this in code (normalise spaces and currency symbols); if it does not
  match, set keyFigure to null and flag the card for the editor.

CLAUDE CALL
Keep the existing ANTHROPIC_API_KEY secret. Use the official
@anthropic-ai/sdk package rather than raw fetch, model "claude-opus-5-5".
Use structured outputs (output_config.format with a JSON schema) instead of
"reply with JSON only". Set output_config.effort to "medium". Check
stop_reason before reading content; on "refusal" or a failed call, keep the
previous stories and write fetch_error. Follow the current Anthropic API docs
for exact parameter shapes.

Schema Claude must return, per story:
{
  "n": number,                        // index in the shortlist
  "title": string,                    // rules above
  "body": string,                     // the caption, rules above
  "category": one of the CATEGORY ids below,
  "keyFigure": string | null,         // verbatim, e.g. "GH₵6.2bn", "610 rounds", "4.6%"
  "sourceInstitution": string,        // who the facts come from, e.g. "GRA Customs Division", "ISSER, University of Ghana"
  "brief": string                     // 1 to 2 sentences for the illustrator: hero object, 2 to 3 props, one Ghana cue
}

CATEGORY ids:
education, health, economy, finance, energy, transport, courts, crime,
security, politics, sport, environment, disaster, housing, construction,
technology, agriculture

CONTENT SCHEMA (src/lib/edition-content.ts, HeadlinesContent.stories[])
Add these optional fields (older editions without them must still render):
category?: string; keyFigure?: string | null; sourceInstitution?: string;
imageStatus?: "none" | "generating" | "ready" | "approved" | "failed";
imageModel?: string; approved?: boolean.
Keep srcName and link (the outlet and URL) for the editor's reference only;
they are not shown on the page.

The page's source line becomes:
"Sources: " + the unique sourceInstitution values joined with " · ".
The outlets (MyJoyOnline, Citi, 3News, GNA) are no longer listed on the page.

When done, run one fetch and show me the six drafted stories as JSON, with
the key-figure check result for each. No design changes yet.
```

---

## Message 2 of 3: the page layout (810 x 1080)

```
Now redesign the Headlines page component
(src/components/cediboard/pages/Headlines.tsx and the .hl-* / .story styles in
src/styles/cediboard.css) to match the attached samples.

The attached images are our manual production at 1200 x 1600. Our page is
810 x 1080, the same 3:4 shape, so match their layout and proportions at
about 0.675 scale. They are the LAYOUT reference; follow the sizes and rules
in this message where they differ. Keep the paper's shared PageHeader
("GHANA IN NUMBERS" eyebrow + FINEX INSIGHTS wordmark) and PageFooter
(source line, flag, handles, folio) unchanged. Font stays Google Sans Flex.
Do not touch the Book flip engine or page size.

LAYOUT, top to bottom (content width 714 px, 48 px side padding)
1. Title block, centred:
   - "In the Headlines: 6 Major Stories Shaping Ghana Today", 28 px, 800,
     ink, one line.
   - Edition date on its own line in gold #c8960c, 24 px, 800, with an
     ordinal day: "6th October 2026". No dash before it.
   - 3 px ink rule under it, full content width, 12 px below the date.
2. Story grid (14 px below the rule): 2 columns x 3 rows, 12 px gaps.
   Cards: white #ffffff, radius 14 px, soft shadow
   (0 2px 10px rgba(26,24,20,.06)), no border, 12 px padding.
   Card height is fixed so all six are equal (about 245 px; tune so the page
   fits, see QA).
   Inside each card:
   a. Top row: category icon tile + title.
      - Icon tile: 42 x 42, radius 11 px, solid category colour, white
        lucide-react icon 22 px, soft shadow (0 2px 6px rgba(0,0,0,.15)).
      - Title: 17 px, 800, ink, line-height 1.15, max 2 lines, vertically
        centred against the tile.
   b. Illustration: full card width, about 120 px tall, object-fit cover,
      centred, on white (no grey box, no border, no radius on the image).
   c. Caption: 12 px, ink #1a1814 at 85%, line-height 1.4, max 3 lines.
      It must never be cut off with "...": the writing rules in Message 1
      keep it short enough. If it still overflows, flag the card in the
      newsroom instead of truncating.
3. PageFooter with the "Sources: ..." line from Message 1.

CATEGORY ICONS AND COLOURS
These colours are an approved exception to the paper's palette, used only
for the Headlines icon tiles (they come from our manual production).
| category     | colour  | lucide icon     |
| education    | #6B4FBB | CalendarDays    |
| health       | #12807F | Cross           |
| economy      | #C8960C | Landmark        |
| finance      | #C8960C | Landmark        |
| energy       | #E07B24 | Droplet         |
| transport    | #E07B24 | Bus             |
| courts       | #1F3B5C | Gavel           |
| crime        | #8E2A35 | ShieldAlert     |
| security     | #1F3B5C | Shield          |
| politics     | #1F3B5C | Folder          |
| sport        | #2E7D4F | Trophy          |
| environment  | #3E6B48 | Trash2          |
| disaster     | #D93A2B | Flame           |
| housing      | #D93A2B | House           |
| construction | #C0392B | BrickWall       |
| technology   | #2F6FD1 | Smartphone      |
| agriculture  | #2E7D4F | Wheat           |
Stories without a category (older editions) keep their emoji chip.

QA (check before saying you are done)
- The page fits 810 x 1080: scrollHeight equals clientHeight, footer fully
  inside, in single and spread mode.
- All six cards are the same size; no title over 2 lines, no caption over 3
  lines, nothing truncated with "...".
- Test with the longest allowed text: 45-character titles and
  160-character captions in all six cards.
- Older editions (No. 001) still render.
Show me a screenshot of today's draft at 810 x 1080.
```

---

## Message 3 of 3: illustrations, newsroom editor and publish guard

```
Now upgrade the per-card illustrations and the Headlines editor. The attached
images are the STYLE reference for the illustrations: match their look,
richness and density.

IMAGE MODEL
Our manual production uses OpenAI (ChatGPT) for these illustrations, so
generate with OpenAI's image model:
- Add an OPENAI_API_KEY backend secret (ask me for it).
- First check whether the Lovable AI gateway offers an OpenAI image model; if
  it does, use it through the gateway. If not, call the OpenAI Images API
  directly from the server route (src/routes/api/illustration.ts) with the
  newest OpenAI image model available to the key.
- Landscape 1536 x 1024 output.
- For cards with an attached real photo (the existing personImage flow), use
  OpenAI's image edit endpoint with that photo as the input image.
- Keep Gemini (google/gemini-3-pro-image) as a fallback, chosen by a
  newsroom setting "Illustration model: OpenAI | Gemini" (default OpenAI).
  Record imageModel on each story.

STYLE RULES (update src/lib/illustration-style.ts)
Keep the existing rules (glossy editorial vector with soft 3D volume, one
hero object, 2 to 3 props, number badge, category icon, max two short
labels, Ghana anchoring, people rules, no real likeness) with these changes:
1. BACKGROUND: replace the "pure flat white, knocked out to transparency"
   rule. Draw a light, bright scene behind the objects (sea and port,
   stadium, road and city, sky, fields) that fades softly to pure white at
   all four edges, like the samples. Stop running transparent-png.ts on
   these images; the card is white, so the fade blends in.
2. FRAMING: the image is shown cropped to about 1.9:1, so keep the hero,
   badge and labels inside the middle 80% of the height. Nothing important
   in the top or bottom 10%.
3. NUMBER BADGE: use keyFigure from the story (never invent one). Default
   badge: rounded rectangle in bright blue #1F5FD1 with heavy white text.
   Money penalties, losses and price rises may use a red tag (#D93A2B).
   Write the figure exactly as given ("GH₵6.2bn", "610 ROUNDS", "$196.5M").
4. WEAPONS: seized weapons may appear only as inert evidence (in a crate, on
   a table, beside a customs box). Never held, pointed or fired; no blood,
   no injured people.
5. Pass the caption, category, keyFigure and brief into the prompt.

GENERATION FLOW
- "Generate illustrations" in the editor runs all six, two at a time, and
  shows each card's status (generating, ready, failed).
- Each card has "Regenerate", "Upload my own image" (existing ImageUploader)
  and "Approve". Editing the brief and regenerating keeps the previous image
  until the new one is ready.
- Store images in the existing private bucket behind the image proxy.

NEWSROOM EDITOR (Headlines, Manual mode composer)
For each of the six cards, show and allow editing of:
| Field                | Control                          | Rule |
| Title                | text, live counter (max 45)      | 3 to 6 words |
| Caption              | textarea, live counter (max 160) | one sentence |
| Category             | dropdown of the 17 categories    | sets icon and colour |
| Key figure           | text                             | shows a warning if not found in the source item |
| Source institution   | text                             | feeds the Sources line |
| Illustration brief   | textarea                         | used for generation |
| Illustration         | preview + Regenerate / Upload / Approve | |
| Outlet + link        | read-only                        | editor reference |
Plus "Swap story": replace a card with another shortlisted candidate.

TODAY'S DESK AND PUBLISH GUARD
Publish stays blocked while the visible Headlines page has: fewer than six
stories, a title or caption over its limit, a caption that overflows three
lines, a card without an approved image, or an unresolved key-figure
warning. Add matching Today's Desk items ("Headlines: approve 4
illustrations", "Headlines: caption 3 is too long").

QA
- Generate a full set for today and show me the page next to one attached
  sample at the same size.
- No em dashes in any title, caption or Sources line.
- Every number on the page appears in its source RSS item or was typed by
  the editor.
- Regenerate the Headlines share thumbnail after the redesign.
```
