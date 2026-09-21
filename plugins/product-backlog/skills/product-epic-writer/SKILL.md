---
name: product-epic-writer
description: "Write an epic and its story titles as a markdown file to review and paste into the tracker. Use when scoping a block of work, writing or rewriting an epic, splitting one that has grown too big, or turning a PRD, story map or workshop notes into something a team can pick up."
metadata:
  author: rich-warner
  version: 1.1.0
  tags:
    - category/backlog-authoring
    - tool/jira
    - tool/confluence
    - domain/product
---

# Product Epic Writer

An epic is one meaningful block of work with one demo at the end of it, big
enough to be worth a sprint or two, small enough that a human can sit down, run
it, and say whether it works. Under it sit a handful of stories an agent can
build. That is the shape this skill produces.

The failure mode it exists to prevent is the epic that covers every possible
outcome: forty lines of restated business rules, screen-by-screen design prose,
edge cases nobody will build this quarter. That epic goes unread, so it does no
work. Length is not thoroughness.

---

## Tools this skill names

The examples name Jira as the tracker, Confluence as the documentation space,
Figma as the design tool and Miro as the whiteboard. None of it is required:
where a team uses something else, ask once which tool plays each role, record
the answers in House conventions at the bottom, and read the rest with those
names substituted. Ask rather than assuming.

Two things are genuinely Jira-specific, and noted where they appear: the (!)
marker renders as a warning icon there, and the output is plain markdown because
that is what survives a paste into the Jira editor.

---

## Where things live

Information has an appropriate home depending on its purpose. This removes
unnecessary bloat from epics and preserves information for future reference.

| Content | Home | In the epic |
|---|---|---|
| Long-lived rules, policy, definitions, calculations | The documentation space, usually Confluence | Link it. Never restate it. |
| Designs, screens, flows, prototypes | The design tool, usually Figma | Link the flow. No screen-by-screen prose. |
| The slicing: activities, tasks, releases | Story map, in the documentation space or on a board | Take the slices from it. Do not reinvent them. |
| Risks and their mitigations | The risk log, usually a page in the documentation space | Find it, cite the page and the row. Do not restate it or start a second one. |
| Work items, scope, sequencing | The tracker, usually Jira | This is the epic. |

If a rule is load-bearing for this epic and is not written down anywhere yet,
that is a finding. Record it as an assumption and name who needs to write it.
Do not smuggle the rule into the epic body as a substitute.

Stories carry their own sources. A page that bears on one story and not on the
whole block of work, an interface contract or a decision record, belongs on that
story rather than here. The epic carries the sources the whole block rests on,
and `product-story-writer` collects the rest per story.

---

## Stage 0: Read the sources first

Before drafting, spend one message getting the sources you have been told about
but not given. If someone mentions "the eligibility policy on Confluence" or
"Design have a flow in Figma", those documents are the substance of the epic:
drafting without them guarantees a rewrite, and worse, produces assumptions the
page already answers.

So: ask for any named-but-unlinked URL, then read what you have. Where the
documentation space is Confluence and the Atlassian tools are connected, fetch
the page when a link or page title is given, and search the space when you have
only a topic. Note the page title and version you read, so the epic cites
something specific rather than "per the rules page".

Ask only for what has not been provided. If a source cannot be reached, say so
and carry on. A marked gap is workable, a fabricated rule is not.

Useful inputs, in rough order of value: **the story map**; the design flow or
prototype; the governing rules; what the work is in the requester's own words;
the stakeholder concern that prompted it; a vision or PRD, if one exists, so ask
rather than assuming either way; the existing epic, if this is a rewrite or a
split.

The order is deliberate. The story map is the context: what this work is part
of, where it sits in a journey, and the slicing already done. The designs show
that context happening. The rules are the substance of the epic, and they mean
little read cold, so they come after a flow you can already picture.

**Finding the map.** It may be a page in the documentation space, a table or a
grid of activities, tasks and releases, or it may be a board. Where the House
conventions block says which, go there. Where it does not, ask, and look in the
same space as the rules before asking anyone to describe scope. When you find
one:

- Take the story titles from the map, in its wording. Where you add a slice the
  map does not have, or change its wording, mark that line so the difference is
  visible on review.
- Work out which activity, or which release slice, this epic covers, and say so
  in Sources of truth.

**An activity on the map often spans two actors**, a customer-facing row and an
ops-facing one. The activity boundary is not the epic boundary, and this is the
normal case rather than the exception.

This stays one epic. Name what each actor does in it, specifically: the customer
submits a request, and the support team see it arrive in the queue. Whatever
else that actor does waits for a later epic, and the out-of-scope line names it:
what the support team do with a request once it is in the queue.

Say in Sources of truth that you cut across the activity, and why.

For a map on a Miro board, invoke `product-story-map-read` and work from the
hierarchy it returns: activities, tasks and the cards under them, with their
content. For a map in any other whiteboard tool, or one that only exists as an
exported image or PDF, invoke `product-story-map-extract-from-image` and work
from the hierarchy that returns instead — a board the API reader cannot reach is
exported as an image first.

Where a process flow exists, read it alongside the map. It is usually harder to
read than a story map. `product-story-map-read` reads a whole frame, so pointing
it at the frame holding the flow returns its shapes and labels along with
everything else in there. Its hierarchy step only classifies cards, so read those
shapes directly rather than expecting them in the hierarchy it returns.

---

## Stage 1: Rough draft

Now write a strawman, fast, and put it in front of the human. A concrete draft
to argue with beats an interview about an abstraction.

The Language rules below apply from this first pass, not just to the final
version, and so does the list of what is never written.

Fill every section. Where the sources do not support a section, mark the gap
rather than writing plausible filler: (!) to confirm: [the specific question],
or record it under Outstanding assumptions with its consequence. Keep every
assumption in the assumptions list rather than scattered through the body, so
one read tells you what the epic is standing on.

The (!) marker is the house convention for a known gap. It shows up on the
ticket, rendering as a warning icon in Jira, so a reviewer sees at a glance what
is missing. Name the question after it: (!) on its own tells a reviewer nothing,
and neither does a section that reads smoothly while saying nothing. Name who
can settle it where there is someone. Never write a plausible answer in its
place: a marked gap gets closed, a confident guess does not. Stories use the
same marker as (!) Todo.

Write the marker as normal text. Code formatting around it stops Jira rendering
the warning icon.

Then say plainly which sections you had to guess at, and go to Stage 2.

---

## Stage 2: Challenge the draft

Now go after the draft section by section. Questions in prose where the answer
is open: a number, a name, three sentences about a demo. `AskUserQuestion` only
where the answer is genuinely a choice between named options, and then leave
room for *don't know, record it as an assumption*.

Three or four questions per pause, grouped by section, and stop when the
remaining gaps are ones only a meeting can close. Record those and move on.

Questions that earn their place:

- **Problem.** Who does what today, and what does it cost them? Give me the
  number, or tell me nobody measures it.
- **Assumptions.** Which of these, if wrong, changes what we build rather than
  just how long it takes? Kill the rest.
- **Dependencies.** Who or what outside this team has to move first? Name the
  team or system, not "the platform", and say where it is today.
- **Constraints.** Anything that changes what gets built rather than sitting in
  policy: accessibility, a regulatory or safety sign-off, information
  governance, a data protection impact assessment, something the service cannot
  be seen to do. What happens on the worst input someone could give this thing?
- **Slicing.** Where does the story map say this epic starts and stops? If the
  map and the conversation disagree, which one is out of date?
- **Journey.** Which journey is this, and what is it for? What is the reader
  holding when they arrive, and where do they go next? Happy path in steps. Which
  one alternate path matters enough to build now? What are we knowingly leaving
  out?
- **Success criteria.** What moves if this works? Name it in a line. If nobody
  could tell afterwards whether it moved, say that instead.
- **Demo.** Who sits down, what do they do, what do they see? If that takes
  more than three sentences, the epic is not testable yet.

A stakeholder's worry usually lands as a metric with a threshold ("incomplete
submissions no higher than today's 14%"), or a constraint, or both. A worry that
shapes the build and needs watching afterwards is entitled to appear in each,
once. If it is neither, an open risk with a mitigation and an owner, it belongs
in the risk log, referenced by ID, not restated here.

Do not ask about sprint length, team size, estimates, tooling or story points.
None of them change the epic.

After the challenge round, rewrite in place, same filename, and say what
changed and what is still open.

---

## Stage 3: Acceptance criteria and story titles

**Acceptance criteria** describe the state of the world when the epic is done,
not how any story gets there. Each one names the observation that would show it
false; if you cannot name one, it is a sentiment, not a criterion. Write the
falsifier down. A test the reviewer cannot see is a test nobody ran.

Five or six is the useful range. Past eight, either the epic is too big or the
criteria are doing a story's job. Given/When/Then only where state actually
matters; a plain checkable statement is usually clearer.

Edge cases belong here only when handling them changes what gets built.
Otherwise they are a rule on a documentation page or a later story.

**Metrics get named, not specified.** Three or four, one line each, and stop.
Baselines, targets, data sources and instrumentation do not go in the epic: they
are the measurement framework's job, or the job of the story that builds the
tracking, and they are the fastest-rotting content an epic can carry. Do not
tier them into launch metrics, guardrails and diagnostics either. A flat list a
reader takes in at a glance is the point. If a metric cannot be measured at all
today, that is a dependency; put it there and say what is missing.

**Story titles** are the slicing, not the story detail. Each is a vertical slice,
a thin path through every layer that can be demonstrated on its own the moment it
lands:

```
- S-1 [Title]. Demo: [what someone can watch working]
- S-2 [Title]. Demo: [...]  Blocked by: S-1
- S-3 [Title]. Demo: [...]  Blocked by: S-1  Waiting on: D-2
```

`Blocked by` takes story IDs. `Waiting on` points at a dependency by its ID, so
the outside blocker is named once above and referenced here rather than retyped.
Keep the `demo:` label on every line: it survives a paste into the tracker as
plain text, it is greppable by whatever reads this file next, and it is the test
each title has to pass.

Test every title with one question: what can I demo when this is done? A title
with no answer is a horizontal slice (a layer, a schema, "set up the service")
and needs reslicing or folding into another story. The first story must have no
blockers, so someone can start immediately. No estimates.

Where a story map exists the titles come from it (Stage 0). Where there is no
map, you are doing the slicing yourself: walk the happy path, cut where a person
could stop and still have watched something work, and check each cut against the
demo question before writing it down. Do this with the person rather than alone,
particularly where there is a prototype. Walk it with them and ask where a clear
slice of delivery falls, even where it is not the whole thing, since they will
know which parts are settled and which are still moving.

Story titles obey the plain English rule in the Language section below. This is
where it slips most often: "Store the draft immutably", "Handle the unassigned
booking" and "Upload the template" all name a change to the system rather than
to someone's day, and they read as build tasks. Write what becomes possible
instead.

Write titles for the human reading the backlog. The detail an agent needs,
per-story acceptance criteria, technical notes, design frames, what gets logged,
is out of scope here and belongs to `product-story-writer` and to a technical
story skill where the team uses one.

---

## Writing for whoever builds it next

The epic is the context someone loads before touching a story, increasingly a
fresh session with no memory of the conversation that produced it. That reader
cannot ask you what you meant, so ambiguity a colleague would shrug off becomes a
wrong build. Two habits do most of the work, and neither costs a human reader
anything:

- **Name things as the system names them.** "The order record in Salesforce",
  not "the record", using whatever the team's systems are actually called. Where
  a term differs between the policy, the designs and the code, record both forms
  in Vocabulary and pick one for the epic. This is the cheapest defect
  prevention in the document.
- **Write the boundary as a boundary.** Anything capable will build the thing you
  left out, and `Out of scope for now` is the only line that stops it. State what
  will not happen, not what might happen later.
- **State the requirement; do not argue for it.** The reader is looking something
  up, not being persuaded. Contrastive framing, such as *this is not a preference, it is
  a control*, *it isn't about X, it's about Y*, *not a nice-to-have but a
  requirement*, spends a sentence arguing with an objection nobody raised, and
  invents a sceptical reader who is not in the room. Write the requirement, then
  write what depends on it.

  Avoid: *Nothing is issued unsigned. This is not a preference about quality: it
  is the human-in-the-loop control our compliance position depends on.*

  Write: *Nothing is issued unsigned. The signature is the human-in-the-loop
  control our compliance position rests on.*

  The test: delete the emphasis and read the sentence again. If it needed the
  contrast to feel important, either it was not important, or the important part
  has not been written yet. The same goes for *importantly*, *critically*, *it is
  worth noting that*, and bolding a claim to make it feel load-bearing.

- **Bullets for anything enumerable; narrative only where it earns it.** Prose
  hides items mid-sentence behind a semicolon, and a reader checking whether
  their obligation is covered needs lines they can point at. Bullet the lists: assumptions, dependencies, constraints,
  metrics, criteria, stories, and any set of rules, fields or conditions. Keep
  narrative for Problem and context and the Demo, where the connective tissue
  between sentences is the content. Prefer a table when items share the same
  attributes, and a numbered list when the order carries meaning.
- **Promotion needs provenance.** Content moving from a ticket into a permanent
  page must bring its confidence with it, not just its wording. In a ticket, "the
  Stage 2 doc, to be revised, follow up with the author" reads as a note with
  visible doubt; the same filename in a table on a rules page reads as
  established fact. Either give it a first-hand source and a link, or mark it as
  unverified and name the secondary source it came from. Any filename, document
  name or system name carries a link or a visible `link needed`, because a bare
  name is a claim with no way to test it.

Argument is usually a symptom. Where a section starts building a case,
pre-empting objections or insisting, the decision behind it has probably not been
taken. Write the open decision as an assumption with an owner, and let the
argument happen in the meeting where it belongs. An epic records decisions; it
does not win them.

IDs exist for the same reason: an assumption referenced as A-2 from a metric is
said once, and every reference survives a rewrite. Allocate an ID once and never
reuse it. A settled assumption stays in its place marked `settled`, or is
removed and its number retired. Renumbering breaks every reference to it, in this
file, in the tracker, and in whatever reads it next.

None of this is a licence to write more. It is a reason to write exactly.

### Language

These apply to the epic, not to this file.

- **No em-dashes.** Use a comma, a colon, a semicolon, brackets or a full stop.
- **Words to avoid**: seamlessly, robust, leverage, ensure, simply,
  comprehensive, it is worth noting, importantly, critically.
- **Do not open every criterion or bullet with the same verb.** It reads as
  generated even when the content is right.
- **Bold labels a field, nothing else.** No bolded fragments used for emphasis.
- **Plain English in the title and the problem statement.** No table, column,
  endpoint, class or service names, no pattern words (saga, state machine,
  idempotency, enum), no build verbs (implement, refactor, wire up, configure,
  integrate, migrate). Names the business actually says are fine: the name of
  the CRM, the customer portal, the order reference. Technical naming belongs in
  the stories, not the epic title. `product-story-writer` carries the paired
  examples.
- British English.

---

## What is assumed and never written

Covered by the design system and the team's standards. Writing them into an epic
adds length and teaches readers to skim.

- Accessibility.
- Tested code.
- Established patterns from the design system.
- Security and data handling rules that apply to everything.

Write one of these into an epic only where this epic changes the rule or breaks
the pattern. Then say which, and why.

---

## Output format

Write to `{epic-slug}-epic.md` in the working folder, and rewrite that same file
on each pass so there is one current version. Plain markdown, headings and
lists, so it survives a paste into the tracker's editor.

Omit the Constraints section entirely when there is nothing in it. An empty
heading invites padding.

```markdown
# [Epic title: the outcome, not the component]

Owner: [who signs this off] · Author: [who wrote it] · Actors: [who is involved] · Parent: [initiative or PRD link]

## Problem and context
[Who is affected, what it costs them today with a number or a named source, and
why now. Two or three short paragraphs. Context is only what a reader cannot get
from the sources below.]

## Sources of truth
- Rules: [documentation page, version]. Sections this epic depends on: [names]
- Designs: [flow or prototype link]. Frames: [names]
- Story map: [link]. This epic covers: [activity or release slice, and any cut
  across it]

Vocabulary: [term used here] = [term used in the system], where they differ.

## Outstanding assumptions
- A-1 [Assumption]. If wrong: [what changes]. Owner: [who can settle it]

## Known dependencies
- D-1 [Team or system]: [what we need, and where it is today]

## Constraints
- C-1 [Constraint that changes what gets built, and what it forces]

## Journey
[One line: which journey this is, and what it is for.]

Arrives from: [what the reader is holding, and which epic they come from]

1. [Step. Name the actor and the system]

Leads to: [what happens next, and which epic picks it up]

Also handled: [alternate paths in scope]
Out of scope for now: [the boundary: what this epic will not do]

## Success criteria and metrics
- M-1 [What moves if this works, in one line]

## Demo
[Who sits down, what they do, what they see, and where they look to confirm it
landed. Three sentences.]

## Acceptance criteria
- AC-1 [Actor] can [action] and [observable result], observed in [where].
  False if [the observation that would disprove it].

## Stories
- S-1 [Title]. Demo: [...]
- S-2 [Title]. Demo: [...]  Blocked by: S-1  Waiting on: D-1
```

---

## Length, and when to split

Keep Problem and context, Journey and Demo together under 450 words. The
structured lists (sources, assumptions, dependencies, constraints, metrics,
criteria) cost what they cost, because a version number, an owner and a
falsifier are the parts that make the epic usable. Expect the whole page around
800 words and treat 1,000 as the point to start cutting.

The commonest way to blow this is to specify where you should be naming: a
dependency becomes an integration design, a constraint becomes a policy summary.
Name the thing, point at where it is specified, move on.

Cutting means cutting, not splitting. Length is not a split trigger: a
well-scoped epic with six criteria and three constraints is simply longer than a
vague one, and splitting it to hit a word count makes two worse epics.

Test each sentence before keeping it: delete it, and ask whether anyone would
build or decide differently. If not, it was padding. That test does more work
than any word count.

Recommend a split, and say which trigger fired, when:

- No single demo can show the whole thing working.
- The demo needs more than one sitting, or two people who could not be the same
  person in a test environment.
- Half the stories wait on a dependency the other half never touch.
- The story list runs past eight or nine slices.

Name the proposed epics, give each a one-line demo, say what sequences them, and
let the human decide. Do not split unilaterally, and do not quietly return two
epics when asked for one.

---

## Quality checks

Run these before handing over, and say which ones you could not satisfy:

- Every claim traces to a source you read or something the human said. Nothing
  invented to fill a section.
- The problem is stated as a number or a named observation, or is explicitly
  marked as unmeasured.
- Every source names the specific version, section or frame this epic depends
  on, not just the document.
- No rule restated from the documentation space; no screen-by-screen description
  of the designs; no rule, dependency, constraint or metric stated twice.
  (Journey and Demo may cover the same ground; the journey gives the steps, the
  demo names who runs it and what they see.)
- Every acceptance criterion names an actor, an observable result and where it is
  observed, and carries its falsifier in writing.
- Every story title answers "what can I demo when this is done?" with behaviour,
  and the first has no blockers. Where a story map exists, the titles came from
  it.
- IDs are unique, and nothing references an ID that does not exist.
- Each story is finishable by someone starting cold with this epic and its links.
- Metrics are named in a line each, four at most, with no baselines, targets or
  data sources carried in the epic.
- No sentence argues for a decision already taken. Nothing is framed as "not X
  but Y", and no emphasis word is doing work a plain statement should do.
- No em-dashes, no words from the avoid list, and the title reads in plain
  English with no technical naming.
- No accessibility, testing or design system boilerplate.
- Every enumerable set is a list or a table, not a paragraph.
- Every gap carries the (!) marker with the question named after it, written as
  normal text rather than code formatting, and nothing plausible has been written
  in its place.
- Every filename, document or system named in the epic carries a link or a
  visible `link needed`, and anything taken second-hand says so.
- The demo is something a human can actually run.

---

## What this skill does not do

- Does not write story detail. That is `product-story-writer`, and a technical
  story skill where the team uses one.
- Does not create or update tracker issues. It produces a file for the human to
  review and paste; the human owns what lands in the tracker.
- Does not reslice an existing story map, or lift its card detail into the epic.
  Where a map exists, on a page or a board, it is the source of truth for
  slicing, and a disagreement with it is a finding to raise.
- Does not keep the risk register. Risks with mitigations live in the risk log.
  Find it, reference the row, and raise anything missing rather than starting a
  second list.
- Does not invent metrics, owners or dependencies to make a section look
  complete. A gap is not a zero.

---

## House conventions

Fill these in once for the team and they stop being questions every run. Until
they are filled in, ask rather than assume.

Tools:

- Tracker: Jira, or (!) to confirm
- Documentation space: Confluence, or (!) to confirm
- Design tool: Figma, or (!) to confirm
- Whiteboard, where the story map or process flows live there: Miro, or (!) to
  confirm

This engagement:

- Tracker project key: (!) to confirm
- Space or area holding the governing rules: (!) to confirm
- Where the story map lives: (!) to confirm: a documentation page or a board?
- Where the risk log lives: (!) to confirm
- Standing constraints that apply to most epics here: (!) to confirm, e.g.
  WCAG 2.2 AA, a security review before release, a data protection impact
  assessment before go-live
- House convention for epic titles: (!) to confirm

If a standing constraint applies to every epic, it belongs in the documentation
space with a link from the epic, not retyped each time.

---

## Changelog

- **v1.1 (2026-09-14).** Trimmed. The description halved to the trigger
  conditions, Language folded into "Writing for whoever builds it next" as a
  subsection, and the restatements cut from Stage 1, the story map bullets in
  Stage 0, the Tools section and the Length section. No rule removed.
- **v1.0 (2026-09-14).** First version. The worked examples are domain-neutral,
  the tool names sit in a Tools section and the House conventions block so a
  team can substitute their own, and the story map readers are referenced under
  their `product-` names. The technical story skill is named as a role rather
  than a specific skill, since none ships in this set.
