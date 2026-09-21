---
name: product-story-map-read
description: Extract a user story map from a Miro frame through the Miro connector. Retrieves full card content including titles, descriptions, detail sections, and design links, then hands a confirmed hierarchy of epics, activities and stories to the epic and story writers. Use when the user provides a Miro frame URL and the Miro connector is available.
metadata:
  author: rich-warner
  version: 1.2.0
  tags:
    - category/story-mapping
    - tool/miro
    - domain/product
---

# Product Story Map Read (Miro)

## Purpose

Extract a structured hierarchy of epics, activities, and stories from a Miro
board using the Miro connector. Produces richer output than the image reader: full card
descriptions, detail sections, and design links are all available. The output is
a confirmed, human-reviewed hierarchy ready to pass to `product-epic-writer` and
`product-story-writer`.

## When to Use

- The user provides a Miro frame URL for the story map
- The Miro connector is available
- The map is built from stickies, from cards, or from a mix of the two

If the Miro connector is not available, fall back to
`product-story-map-extract-from-image` and ask the user to export the story map
as a PDF or screenshot.

Where the team keeps its story map somewhere other than Miro, say what this
skill can and cannot reach rather than guessing. A map held as a table on a
documentation page is read directly by `product-epic-writer`; a map in another
whiteboard tool is exported as an image and read by
`product-story-map-extract-from-image`.

---

## Approach

### Step 1 — Get the board URL

Ask the user for the Miro board or frame URL if not already provided. A frame URL
looks like:

```
https://miro.com/app/board/uXjVM12IvnU=/?moveToWidget=3458764673992133668
```

The frame ID (the `moveToWidget` value) scopes the extraction to just the story
map frame, and Step 2 needs it. Apply the same rule as the image reader:
**extract only the story map, not the whole board**. If the user provides a board
URL with no frame ID, ask them to navigate to the story map frame and share that
URL rather than reading the whole board.

### Step 2 — Read the frame

Call `canvas_read_as_svg` with:

- `miro_url`: the frame URL provided by the user
- `widget_ids`: the frame ID from that URL's `moveToWidget` value

Naming a frame returns the frame and everything nested inside it, so this is a
single call. There is no pagination to handle.

The response is a canvas-composer SVG document. The frame wraps its contents,
and every item inside is positioned relative to that wrapper:

```xml
<g data-miro-id="3458764683766463937" transform="translate(35243,29032)" data-frame="Frame 9">
  <rect data-type="frame" x="0" y="0" width="2740" height="4904" fill="#ffffff" />
  <rect data-miro-id="..." data-type="sticky" x="833" y="464" width="150" height="172"
        data-content="&amp;lt;p&amp;gt;&amp;lt;strong&amp;gt;Onboard&amp;lt;/strong&amp;gt;&amp;lt;/p&amp;gt;"
        data-color="blue" />
  <rect data-miro-id="..." data-type="custom-widget" data-widget-type="card"
        data-title="Hide the option for ineligible customers" data-description="..."
        data-color="#8950ff" x="240" y="880" width="320" height="88" />
</g>
```

Both item types appear on real maps and both are in scope. Read whichever the
frame contains, and a mix if it contains both.

Swim lanes, where a map has them, come back as further `<g data-frame="Release 1">`
wrappers nested inside. Many maps have none, which is not a problem: see Step 4.

**Check `item_count` against `skipped_count` in the response.** A read reports
both, and a skipped item is one this tool could not return. Items elsewhere on
the board are routinely skipped and do not matter — the scope is the frame. But
say the counts out loud when presenting the hierarchy, because a skipped item
*inside* the frame is a card missing from the map with nothing else to signal
it.

**If the read fails on permissions**, the connector is authorised as a Miro
account that cannot open the board. Say so and stop — the user either shares the
board with that account, reconnects as one that has access, or exports the frame
and uses `product-story-map-extract-from-image` instead. Do not retry.

**If the read fails on size, stop.** An area read fails above 500 widgets and the
document caps at 200,000 characters. Do not retry with a narrower scope and do
not hand on a partial map:

> "That frame is too large to read in one pass. A story map that big is hard to
> slice from in any case — split it into per-epic frames and point me at one, or
> export it as an image and I will use
> `product-story-map-extract-from-image` instead."

A partial hierarchy is worse than none here. The writers treat the story map as
the source of truth for slicing, so cards missing from the read become stories
missing from the backlog, with nothing to signal the gap.

### Step 3 — Confirm the colour key and the levels

Both item types carry `data-color`, but in different forms. A **sticky** returns
a named colour — `yellow`, `blue`, `light_blue`, `green`, `orange`. A **card**
returns a theme colour as a hex value. Read whichever you got; do not convert
between them and do not assume a palette.

There is no default mapping. Story map palettes vary by team and by template, so
propose one from what is actually on the frame — the y bands usually give it
away, since a colour used once across the top is a header and a colour repeated
down every column is a story — and have the user confirm before classifying:

> "This frame has 34 items in 3 colours, in 3 bands top to bottom:
>
> - `light_blue`, `yellow`, `orange`, `green` — 4 items in the top band
> - `blue` — 6 items in the next band
> - `yellow` — 24 items below that
>
> I read that as:
> - Top band = the map's title and legend, not part of the hierarchy
> - `blue` = Activities
> - `yellow` = Stories
> - No epic row on this map
>
> Have I got that right?"

Two things to get right when proposing it:

- **A map need not have three levels.** Activities and stories with no epic row
  above them is a normal shape, especially on a sticky map. Propose the levels
  you can actually see rather than forcing epic, activity and story onto the
  frame.
- **A title, a legend or a colour key is not the top level.** These sit above the
  first real band, often one item per colour, and their text describes the map
  rather than naming a step in it. Propose them for exclusion by name so the user
  can see what you are dropping.

Items whose colour or band the user does not settle go to `items_for_review`.

### Step 4 — Build the hierarchy

Once the colours and levels are confirmed, classify each item and build the
hierarchy from position.

Use each item's own `x` and `y`. These are relative to the frame's
`transform="translate(...)"`, and every item on the map shares that frame, so
they compare directly. Some items also carry `data-rendered-bounds`
(`x y width height`) in absolute board coordinates; where it is present prefer
it, and where it is absent the item's own `x`/`y`/`width`/`height` is its real
box. Do not mix the two in one comparison.

**Level by y-position:**

Take the bands the user confirmed in Step 3, top to bottom. Do not assume there
are three, and do not assume the top band is epics — on many maps it is
activities, with no epic row at all.

**Column grouping by x-position:**

- An Activity belongs to the Epic whose column it sits in
- A Story belongs to the Activity whose column it sits in

Judge the columns from the gaps between them, not from a fixed tolerance. An
activity's stories are often wider than one item and need not sit strictly below
the activity header, so do not exclude an item for loose horizontal alignment
inside a cluster. Where an item falls between two columns and the gaps do not
settle it, put it in `items_for_review` rather than guessing — the same handling
as an item of an unknown colour.

**Release slices by swim lane:**
Horizontal lanes come back as `<g data-frame="...">` wrappers nested in the story
map frame. Each lane's `data-frame` title is a release name, and a story belongs
to the lane whose bounds contain its rendered bounds. Fill `releases` from these.
Where the frame has no nested lanes there is no release slicing: leave `releases`
empty rather than inventing one, and say so when presenting the hierarchy.

**Text content:**
A sticky carries its text in `data-content`; a card carries `data-title` and
`data-description`. Both arrive as escaped HTML fragments, not plain strings —
`&lt;p&gt;&lt;strong&gt;Onboard&lt;/strong&gt;&lt;/p&gt;` is a sticky reading
"Onboard". Unescape, then take any `href` values out before stripping the tags,
then strip. Convert HTML lists (`<ol>`, `<li>`) to plain text bullet points.

**A sticky map carries titles and nothing else, and that is fine.**
Stickies have no description field, so `description`, `tech` and `designs` are
simply absent on a sticky map. Do not treat that as a gap to fill or flag: the
map is the slicing, and the detail comes later from the PRD, the documentation
space or a prototype. Hand over the titles and the hierarchy and let the writers
ask for the rest.

**Design links:**
Where a description exists, take the first URL in it, whether it appears inside
an `href` or as bare text. Where there is none, or where the map is stickies,
leave `designs` unset rather than guessing from a link label.

**Description parsing:**
Do not attempt to parse specific sections (Detail, Goal, Metrics, Designs) from
descriptions, since teams use freeform headings. Pass the full stripped
description through as-is. This preserves whatever structure the team chose to
use.

**Ticket numbers:**
If a card title contains a ticket reference (for example ABC-123), tag it as
`[EXISTING]`. Include it in the hierarchy to confirm epic and activity links, but
flag it so it is not re-imported.

**Metrics:**
Story-level cards whose descriptions contain metrics content (for example
"Journeys started", "Drop-out rate", "Straight through processing %") should be
flagged as `[METRIC]` and surfaced separately for review, not included in the
story list. An epic card's metrics are not flagged here: they belong in that
epic's own `metrics` field in the schema below.

### Step 5 — Present hierarchy for confirmation

Output the full extracted hierarchy with full card descriptions and ask the user
to confirm or correct before proceeding:

```
Epic: Order a document
  Goal: Get customers to find and order documents via the app
  Metrics: Journeys started

  Activity: Establish eligibility
  |-- Hide the option for ineligible customers
      Don't show the option when the customer holds one of the following
      product types:
      - Tracker product (types 396, 397)

  |-- Show the option
      Show the option per the design for eligible customers.
      Designs: https://figma.com/example-link
```

Say:

> "Here is what I've extracted from the board. Please review and correct anything
> before I proceed."

Do not proceed to the writers until the user has confirmed the hierarchy is
correct.

### Step 6 — Output the intermediate schema

Once confirmed, output the hierarchy as structured data. `description`, `goal`,
`metrics`, `tech` and `designs` come from item descriptions, so on a sticky map
they are absent throughout — emit the keys you have rather than empty ones:

```yaml
epics:
  - name: string
    description: string (if present)
    goal: string (if present in the epic's description)
    metrics: string (if present in the epic's description)
    existing: true|false
    activities:
      - name: string
        description: string (if present)
        existing: true|false
        stories:
          - name: string
            description: string (full description, HTML stripped, if present)
            existing: true|false
            tech: true|false
            designs: string (URL if present in description)
        releases:
          - name: string (if swim lane slicing is present)

metrics_flagged:
  - name: string
    context: string

items_for_review:
  - name: string
    context: string

excluded:
  - name: string
    reason: string
```

**Where the map has no epic row**, do not invent one. Emit `activities` at the
top level instead of nesting them under an epic, and say so when handing over:
the epic writer scopes the epic itself, and a fabricated parent would be a
decision the map never made.

`excluded` carries the title, legend and colour-key items dropped in Step 3, so
the user can see what was left out rather than having to notice its absence.

---

## Key differences from the image reader

| | Image reader | API reader |
|--|--|--|
| Content | Title only | Titles always; full descriptions where the map is built from cards rather than stickies |
| Colour classification | User must confirm upfront | Read from each item's data-color, then confirmed |
| Hierarchy | Inferred from position (fragile) | Confirmed from rendered bounds (reliable) |
| Missing items | Possible | None silently: an oversized frame fails rather than returning a partial map |
| Metrics detection | Visual and positional | Confirmed from description content |
| Design links | Not available | Extracted from description |

---

## Feeds into

`product-epic-writer`, which takes the slicing from the map and writes the epic,
and `product-story-writer`, which writes the stories under it.

Card detail belongs to the story, not the epic. Pass it through in full and let
`product-story-writer` use it; `product-epic-writer` takes the titles and the
slicing only.

---

## Changelog

- **v1.2 (2026-09-15).** Corrected against a live board, the first time this
  reader has been run against one. Sticky maps are now first-class: a sticky
  carries its text in `data-content` and has no description, so a map built from
  stickies yields titles and a hierarchy and nothing else, which is a legitimate
  starting point rather than a gap — detail arrives later from the PRD, the
  documentation space or a prototype. Colour is read as it comes: named on a
  sticky (`blue`, `yellow`), hex on a card. The fixed default palette is gone,
  because the observed board matched none of it. Levels are proposed from the
  bands actually on the frame and confirmed, rather than assumed to be three: a
  map with activities and stories and no epic row is normal, and a title or
  colour key sitting above the first band is furniture, not the top level.
  Position reads from the item's own frame-relative `x`/`y`, since
  `data-rendered-bounds` was absent on every item. Text arrives as escaped HTML
  inside the attribute, so unescaping comes before stripping. A read reports
  `item_count` and `skipped_count`, now surfaced when handing over. A permission
  failure is handled separately from a size failure.
- **v1.1 (2026-09-15).** Rewritten onto `canvas_read_as_svg`. The previous
  version called `board_list_items`, which Miro deprecated in favour of
  `canvas_search` and `canvas_read_as_svg` with a removal date of 14 September
  2026. Naming the frame in `widget_ids` returns it and everything nested inside
  it in one call, so the pagination step is gone. Card colour now reads from
  `data-color` and position from `data-rendered-bounds`, which is absolute and so
  compares across nested frames. Swim lanes arrive as nested `<g data-frame>`
  wrappers, which is what finally fills `releases`. Column grouping judges the
  gaps between columns rather than a fixed half-card tolerance, and an
  unresolvable card goes to `items_for_review`. An oversized frame now fails and
  says so instead of returning a partial map.
- **v1.0 (2026-09-14).** First version. Neutral examples, and a note on what to
  do when the map lives somewhere other than Miro. Feeds `product-epic-writer`
  and `product-story-writer`.
