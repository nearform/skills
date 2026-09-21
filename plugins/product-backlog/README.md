# product-backlog

Client-neutral product skills for scoping a block of work into an epic and
turning it into stories a team can build.

## Status

Initial iteration. These skills are tuned to the cases they were written for: a
story map built from cards on a single Miro frame, an epic and its stories
pasted into Jira, governing rules in Confluence, designs in Figma. They are
deliberately narrow rather than general.

Where your case breaks one of them — a board laid out differently, a tracker
that does not behave like Jira, a step that assumes something your team does not
do — the fix is to add that capability rather than to work around it. Open an
issue saying what broke and what your setup looks like.

`product-story-map-read` has been run against one real board — a sticky map of
34 items on a single frame — and corrected against what came back. One board is
not coverage. A map built from cards, one with swim lanes, one with a genuine
epic row, and one large enough to hit the read limits are all untested, and the
colour and level confirmation in Step 3 exists precisely because no two teams
lay a story map out the same way. If it reads your frame wrongly, the layout you
used is the useful half of the issue.

## What it does

Four skills, in the order they are usually used:

| Skill | Use it when |
|---|---|
| `product-story-map-read` | The story map is on a Miro board. Pulls the cards and their content through the Miro connector. |
| `product-story-map-extract-from-image` | The story map only exists as an exported image or PDF, or the board is somewhere the API reader cannot reach. |
| `product-epic-writer` | Scoping a block of work, writing or rewriting an epic, or splitting one that has grown too big. Produces a markdown epic to review and paste into the tracker. |
| `product-story-writer` | Turning an epic into stories, one at a time, each testable by someone who was not in the room. Produces one markdown file per story. |

## How it works

The skills do not write to the tracker. The two writers each produce a markdown
file for a human to review and paste; the readers hand their hierarchy to the
writers in conversation rather than writing one.

That is how the skills are written, not a restriction on the connector. The
Atlassian connector registered here can create, edit, comment on and transition
issues. A team that wants an agent doing some of that can say so and override the
instruction — nothing in the plugin prevents it, the default is simply not to.

A vision or PRD is an input, not an output. The epic writer asks whether one
exists and reads it where there is one, rather than producing it.

Long-lived content stays where it belongs. Business rules, the risk log and the
technical documents live in the documentation space and are linked, never
restated. Designs live in the design tool and are linked, never described screen
by screen. The story map, on a page or a board, is the source of truth for
slicing.

Stories are proposed one at a time and agreed in conversation before the next
one is drafted.

Gaps are marked `(!)` as normal text, which renders as a warning icon on a Jira
ticket. Nothing plausible is ever written in place of an answer.

## Tools

The skills name Jira as the tracker, Confluence as the documentation space,
Figma as the design tool and Miro as the whiteboard, because that is the
commonest stack. None of it is required: where a team uses something else, the
skills ask once which tool plays each role and work with those names instead.

Both connectors are optional in the sense that the skills ask for what they
cannot reach rather than failing. Without Atlassian you paste the relevant pages
in yourself.

| Connector | Used for |
|---|---|
| Atlassian | Reading Confluence rules, technical documents and the risk log, and reading Jira issues. The skills only read; the connector itself can write, so a team that wants that can ask for it. |
| Miro | Reading a story map or story cards where the team works on a board. Optional, depending on the workstream. |

## House conventions

`product-epic-writer` carries a house conventions block with two groups of
values to fill in. The first names the tools: tracker, documentation space,
design tool, whiteboard. The second is per engagement:

- The tracker project key
- The space holding the governing rules
- Where the story map lives, a documentation page or a board
- Where the risk log lives
- The standing constraints that apply to most epics
- Any convention for epic titles

Fill them once and they stop being questions on every run. Until they are
filled, the skills ask rather than assuming.

## Versions

- `product-epic-writer` v1.1.0
- `product-story-writer` v1.0.0
- `product-story-map-read` v1.2.0
- `product-story-map-extract-from-image` v1.0.0
