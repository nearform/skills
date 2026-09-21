---
name: product-story-map-extract-from-image
description: Extract a user story map from an image or PDF export and produce a confirmed hierarchy of epics, activities and stories ready for the epic and story writers. Use when a user uploads or shares a story map image or PDF export, or when the whiteboard connector is unavailable.
metadata:
  author: rich-warner
  version: 1.0.0
  tags:
    - category/story-mapping
    - domain/product
---

# Product Story Map Extract From Image

## Purpose

Extract a structured hierarchy of epics, activities, and stories from a story map
image or PDF export. The output is a confirmed, human-reviewed hierarchy ready to
pass to `product-epic-writer` and `product-story-writer`.

## When to Use

- A user uploads a story map image or PDF
- The whiteboard connector is unavailable, or the map was built somewhere the API
  reader cannot reach
- A user wants to turn a story map into epics and stories

Where the map is on a Miro board and the Miro connector is available, use
`product-story-map-read` instead. It returns full card descriptions and design
links; this skill sees titles only.

---

## Approach

### Step 1 — Confirm the export

Before reading the image, confirm:

> "Is this export just the story map, or does it include the whole board?
> If it includes the whole board, please re-export with only the story map
> selected. Whole-board exports are too small to read accurately."

Do not proceed if the image contains multiple unrelated artefacts. Whole-board
exports fail because text is too small and the story map cannot be reliably
distinguished from other content.

### Step 2 — Confirm the colour key

Ask:

> "Before I extract the map, please confirm what each sticky colour represents,
> for example: yellow = epic, teal = activity, purple = story, pink = metric. If
> your team hasn't used colour deliberately, just let me know and I'll rely on
> position only."

Record the colour key. If the user has not used colour deliberately, proceed on
position alone.

### Step 3 — Extract the hierarchy

Read the image and extract the hierarchy using the following rules:

**Position rules (always apply):**

- Row 1 (top) = Epics, the highest level grouping across the map
- Row 2 = Activities, grouped under the Epic in the same column
- Rows below = Stories, grouped vertically under the Activity in the same column
- Horizontal swim lanes = Release slices, capturing the release name where present

**Cluster rule:**
A sticky belongs to an activity column if it is visually within the same cluster
as other stories under that activity, not only if it is strictly vertically below
the activity header. Do not exclude items based on loose horizontal positioning
within a cluster.

**Ticket number rule:**
Any sticky containing a ticket reference (for example ABC-123) is an existing
backlog item. Include it in the output tagged as `[EXISTING]` so epic and feature
links can be confirmed, but flag it clearly so it is not re-imported as a new
item.

**Tech story rule:**
Stories that are clearly technical implementation tasks (often prefixed "Tech:"
or similar) should be included but tagged `[TECH]`.

**Metrics rule:**
Stickies that represent usage metrics, performance metrics, or audit metrics are
not stories. Tag them as `[METRIC]` and include them in a separate flagged
section. Do not include them in the story list.

**Ambiguous items:**
If a sticky's purpose is unclear (a technical note, a question, an annotation),
include it tagged as `[REVIEW]` rather than silently dropping it.

### Step 4 — Present hierarchy for confirmation

Output the full extracted hierarchy in the following format and ask the user to
confirm or correct it before proceeding:

```
Epic: <name> [EXISTING if applicable]

  Activity: <name>
  |-- <story name>
  |-- <story name> [EXISTING — confirm epic/feature link]
  |-- <story name> [TECH]
  |-- [REVIEW: <item> — note or story?]

  [METRIC: <metric name> — flag for review]
```

Say:

> "Here is what I've extracted. Please review and correct anything I've missed or
> misread before I proceed."

Do not proceed to the writers until the user has confirmed the hierarchy is
correct.

### Step 5 — Output the intermediate schema

Once confirmed, output the hierarchy as structured data:

```yaml
epics:
  - name: string
    existing: true|false
    activities:
      - name: string
        existing: true|false
        stories:
          - name: string
            existing: true|false
            tech: true|false
        releases:
          - name: string
            stories: [story names]

metrics_flagged:
  - name: string
    context: string (which activity or story it appeared near)

items_for_review:
  - name: string
    context: string
```

---

## Export guidance (to share with users)

When exporting from a whiteboard:

- Select only the story map frame, not the whole board
- Export as PDF or screenshot at readable zoom
- If the map is large, export in sections by epic

---

## Feeds into

`product-epic-writer`, which takes the slicing from the map and writes the epic,
and `product-story-writer`, which writes the stories under it.

This reader returns titles only. Where the writers need card detail, say so and
offer `product-story-map-read` if the map is on a Miro board the connector can
reach.

---

## Changelog

- **v1.0 (2026-09-14).** First version. Neutral examples, with the whiteboard
  named generically. Feeds `product-epic-writer` and `product-story-writer`.
