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
  only the illustrations are AI-generated.
- The editor pastes the six stories in. There is no automatic story picking.
- Illustrations follow the look of the manual samples (glossy objects, number
  badge, short labels), not the flat style.
- All six illustrations are generated as ONE square asset board (the manual
  production method), then cut into six by code. A single card can be redone
  on its own, using the board as a style reference.
- OpenAI image model (the manual production uses ChatGPT).
- The date sits on its own line in gold under the title, with no dash.
- The manual samples are 1200 x 1600, exactly 3:4 like the 810 x 1080 page, so
  the layout scales by about 0.675.

---

## Message 1 of 3: pasting the six stories and writing them up

```
We are rebuilding the "In the Headlines: 6 Major Stories Shaping Ghana Today"
page (page type "headlines"). This message covers how stories get in and how
they are written. Do not change the page design yet.

HOW STORIES GET IN
The editor chooses the six stories; the app never picks them.
- In the Headlines editor, add a "Paste today's six stories" panel with six
  numbered inputs. Each input accepts either a link to the article or the
  pasted article text.
- For a link, fetch the page server-side and extract the article text. If the
  fetch fails or returns too little text, show "Couldn't read this link, paste
  the text instead" on that input. Never write a story from a headline alone.
- The order of the six inputs is the order on the page (1 to 6).
- "Write up stories" sends all six to Claude in one call.
- Remove "headlines" from the automatic fetch types (AUTO_FETCH_TYPES and
  "Fetch all"), and make Manual the default mode for the Headlines page.
  Leave the RSS fetcher code in place but unused by this page.

HOUSE STYLE (Claude writes these; the editor can change them)
- Title: 3 to 6 words, Title Case, at most 45 characters. Lead with the actor
  and the action. Put the key figure in the title when the figure IS the story
  ("Tullow Loses $196.5m Tax Case", "Hohoe Floods Displace 5,140 Residents").
- Caption: exactly one complete sentence, at most 160 characters, present
  perfect where natural ("has set", "has intercepted"). Name the institution or
  person responsible and include the key figure if there is one.
- No clickbait, no questions, no opinion, no exclamation marks, no em dashes.

FACT RULES
- Use only facts in the pasted text. Never invent numbers, names or events.
- keyFigure must appear verbatim in the pasted text. After Claude replies,
  check this in code (normalise spaces and currency symbols such as GH₵ / GHS
  / ¢); if it does not match, set keyFigure to null and flag the card.

CLAUDE CALL
Keep the existing ANTHROPIC_API_KEY secret. Use the official
@anthropic-ai/sdk package rather than raw fetch, model "claude-opus-5-5".
Use structured outputs (output_config.format with a JSON schema) instead of
"reply with JSON only". Set output_config.effort to "medium". Check
stop_reason before reading content; on "refusal" or a failed call, keep what
the editor already has and show the error. Follow the current Anthropic API
docs for exact parameter shapes.

Per story, Claude returns:
{
  "n": 1..6,                          // matches the input number
  "title": string,
  "body": string,                     // the caption
  "category": one of the CATEGORY ids below,
  "keyFigure": string | null,         // verbatim, e.g. "GH₵6.2bn", "610 rounds", "4.6%"
  "sourceInstitution": string,        // who the facts come from, e.g. "GRA Customs Division"
  "brief": string                     // illustration brief: hero object, 2 to 3 props, one Ghana cue, 1 to 2 sentences
}

CATEGORY ids:
education, health, economy, finance, energy, transport, courts, crime,
security, politics, sport, environment, disaster, housing, construction,
technology, agriculture

CONTENT SCHEMA (src/lib/edition-content.ts, HeadlinesContent.stories[])
Add these optional fields (older editions without them must still render):
sourceText?: string; sourceUrl?: string; category?: string;
keyFigure?: string | null; sourceInstitution?: string;
imageStatus?: "none" | "generating" | "ready" | "approved" | "failed";
imageModel?: string; approved?: boolean.
Also add boardImage?: string on HeadlinesContent (the uncut board, kept for
re-cuts and single-card regeneration).

The page's source line becomes:
"Sources: " + the unique sourceInstitution values joined with " · ".

When done, paste six test stories, run "Write up stories" and show me the six
results as JSON with the key-figure check for each. No design changes yet.
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
2. Story grid (14 px below the rule): 2 columns x 3 rows, 12 px gaps, in the
   order 1 2 / 3 4 / 5 6.
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
   b. Illustration: full card width, about 120 px tall, object-fit contain,
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

## Message 3 of 3: the illustration board, cutting, editor and publish guard

```
Now build the illustrations. We generate all six as ONE square asset board,
then cut it into six images in code. One generation keeps the style
consistent across the six. The attached images are the STYLE reference for
the illustrations: match their look, richness and density.

IMAGE MODEL
Our manual production uses OpenAI (ChatGPT), so generate with OpenAI:
- Add an OPENAI_API_KEY backend secret (ask me for it).
- First check whether the Lovable AI gateway offers an OpenAI image model; if
  it does, use it through the gateway. If not, call the OpenAI Images API
  directly from the server route (src/routes/api/illustration.ts) with the
  newest OpenAI image model available to the key.
- Square output at the largest size the model offers (each card shows one
  sixth of the board enlarged, so resolution matters).
- Keep Gemini (google/gemini-3-pro-image) as a fallback, chosen by a
  newsroom setting "Illustration model: OpenAI | Gemini" (default OpenAI).
  Record imageModel on each story.

BOARD PROMPT
Replace buildIllustrationPrompt in src/lib/illustration-style.ts with a board
prompt built from the six stories (n, title, caption, category, keyFigure,
brief). Use this text, filling the six briefs at the end:

  Create a premium editorial illustration asset board for a news
  publication, "Ghana in Numbers" by Finex Insights. The output should feel
  like artwork commissioned for a high-end editorial newsroom.

  CANVAS: square, background pure white #FFFFFF, minimalist composition,
  plenty of negative space.

  LAYOUT: six illustrations in a perfectly aligned 2-column x 3-row grid,
  in reading order: 1 2 / 3 4 / 5 6. Each illustration sits centred in its
  own section. No cards, no borders, no boxes, no frames, no grid lines,
  no section numbers. The illustrations float on the white page with
  perfectly even spacing.

  SIZE: each illustration spans about 60% of its section's width and about
  30% of its area, centred, with generous white space all round. Do not zoom
  in, do not crop, and never let an illustration touch another section or
  the canvas edge.

  STYLE: modern editorial vector illustration with soft 3D volume: smooth
  gradients, glossy highlights, rounded forms and small soft shadows attached
  under objects. Bright, saturated, friendly palette: blue for institutions,
  government and number badges; red for warnings, penalties and declines;
  green for agriculture, transport and growth; gold for gavels, money and
  finance; warm tan for paper, cement and case files. No photorealism, no
  sketch lines, no dark or moody lighting. All six share one consistent
  style and scale.

  SCENE: each illustration is one self-contained vignette: one hero object,
  two or three supporting props, and at most one Ghana cue (Ghana flag, Bank
  of Ghana tower, Accra skyline, kente, the cedi symbol GH₵). A light scenery
  hint (sea and port, stadium, road, sky) may sit behind the objects but must
  fade to pure white within the illustration's own area.

  NUMBER BADGE: when a story has a key figure, show exactly that figure as a
  badge: a rounded blue (#1F5FD1) rectangle with heavy white text, or a red
  (#D93A2B) tag for penalties, losses and price rises. Write it exactly as
  given. If a story has no key figure, no badge. Never invent a number.

  LABELS: at most two short uppercase labels per illustration on objects
  (e.g. CUSTOMS, EOCO, FULL), one to three words, only where they aid
  recognition. No headlines, captions, sentences, dates or source lines
  anywhere on the board.

  PEOPLE: no real person's likeness. A named public figure becomes a
  generic, respectful Ghanaian figure in that role. Accused people are never
  shown; use a gavel, courthouse or case file instead. Hands (a handshake,
  a hand holding a phone) are fine.

  WEAPONS: seized weapons only as inert evidence in a crate or on a table.
  Never held, pointed or fired; no blood, no injured people.

  QUALITY CHECK before rendering: every illustration is smaller than its
  section with large white space around it; perfectly aligned grid;
  consistent style and scale; each illustration understood instantly;
  every badge figure matches the brief exactly.

  ILLUSTRATION BRIEFS
  1. {title}. {caption} Key figure: {keyFigure or "none"}. Brief: {brief}
  2. ...
  (through 6)

CUTTING THE BOARD (server-side, deterministic)
1. Save the full board as boardImage.
2. Divide the board into an exact 2 x 3 grid of equal sections.
3. In each section, find the bounding box of non-white pixels (any channel
   below 245), ignoring a 2% strip at the section's edges.
4. Pad the box by 8% on every side, crop, and save as that story's img.
5. Flag the card for regeneration if the box touches the section edge (the
   illustration spilled over), covers under 4% of the section (empty), or
   the section contains two separate large shapes (two illustrations).

REGENERATING ONE CARD
"Regenerate" on a single card makes one new illustration (not a new board):
send the board image as a reference with the instruction "Draw a new
illustration for this story in exactly the same style, palette and scale as
the attached board", plus that story's brief and key figure. Output a
landscape image on pure white; trim it with the same bounding-box rule.
For a card with an attached real photo (the existing personImage flow), use
OpenAI's image edit endpoint with the photo as an input image.

NEWSROOM EDITOR (Headlines)
Under the "Paste today's six stories" panel from Message 1, show the six
cards. For each card:
| Field                | Control                          | Rule |
| Title                | text, live counter (max 45)      | 3 to 6 words |
| Caption              | textarea, live counter (max 160) | one sentence |
| Category             | dropdown of the 17 categories    | sets icon and colour |
| Key figure           | text                             | warning if not found in the pasted text |
| Source institution   | text                             | feeds the Sources line |
| Illustration brief   | textarea                         | used for generation |
| Illustration         | preview + Regenerate / Upload / Approve | |
| Source text / link   | read-only, collapsible           | what the editor pasted |
Buttons above the cards: "Generate board" (all six, one image) and "View
board" (shows the uncut board with the cut lines drawn on it).

TODAY'S DESK AND PUBLISH GUARD
Publish stays blocked while the visible Headlines page has: fewer than six
stories, a title or caption over its limit, a caption that overflows three
lines, a card without an approved image, or an unresolved key-figure
warning. Add matching Today's Desk items ("Headlines: paste today's six
stories", "Headlines: approve 4 illustrations", "Headlines: caption 3 is
too long").

QA
- Generate a board for today, show me the uncut board with cut lines, then
  the finished page next to one attached sample at the same size.
- No em dashes in any title, caption or Sources line.
- Every number on the page appears in the pasted text or was typed by the
  editor.
- Regenerate the Headlines share thumbnail after the redesign.
```
