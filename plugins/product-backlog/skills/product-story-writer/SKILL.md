---
name: product-story-writer
description: "Write stories under an epic: a one line goal naming the actor, blocking open questions at the top, short detail, test setup and data, a demo, filled acceptance criteria, and any implementation decisions in a separate section at the bottom. Each story is valuable on its own and testable by a human who was not in the room. Stories are proposed one at a time for review with the business analyst, and the skill reads the tracker without writing to it. Pairs with whichever skill wrote the epic. Use when turning an epic into stories, writing or rewriting a single story, or bringing a thin story up to something a tester and a build agent can both work from."
metadata:
  author: rich-warner
  version: 1.0.0
  tags:
    - category/backlog-authoring
    - tool/jira
    - domain/product
---

# Product Story Writer

A story is one thing a person can do that they could not do before, small enough
to build in a sitting and clear enough that a tester who was not in the room can
pick it up and say whether it works.

Two failures this exists to prevent. The story that reads well and cannot be
tested, because nobody said what state to start from. And the story that hides a
requirement inside a technical note, so it never gets tested at all.

---

## Tools this skill names

The examples name Jira as the tracker, Confluence as the documentation space and
Figma as the design tool, because that is the commonest stack. None of it is
required.

Where a team uses something else, ask once which tool plays each role and read
the rest of these instructions with those names substituted. Where
`product-epic-writer` has already filled its House conventions block for this
engagement, take the answers from there rather than asking again.

Two things are genuinely Jira-specific and noted where they appear: the (!)
marker renders as a warning icon on a Jira ticket, and the output is plain
markdown because that is what survives a paste into the Jira editor. Both are
harmless elsewhere.

---

## How this runs

Stories are proposed, not filed. Reading the tracker is fine: the epic, an
existing story, a linked ticket. Writing to it is not, so no new issues, no
edits, no comments. The output is a file for a human to paste.

Work one story at a time. Agree the breakdown with the business analyst in
conversation first, then draft that story, then take the next. Ten drafted in
one pass get reviewed as a batch, and that is where a duplicated slice or a
drifted name survives review.

---

## Before writing: check the sources

The epic links its sources. A story needs the ones bearing on this slice, and
should not go hunting mid-draft. Ask before starting, and read what you are
given:

- **Business rules.** Which page governs this behaviour, and which section? Note
  the page title and version so the story cites something specific.
- **Screen designs.** Is there a design frame or a prototype for this story, and
  which frames?
- **Interface detail.** Is there a page for the API, the payload or the
  integration contract?
- **Decisions already taken.** Is there an ADR that constrains how this is built?

Where the documentation space is Confluence and the Atlassian tools are
connected, fetch a page where a link or a title is given, and search the space
where you have only a topic. Ask only for what has not been provided. Where a
source cannot be reached, say so and carry on: a marked gap is workable, an
invented rule is not.

A source belongs on the story when it bears on this slice, whether or not the
epic carries it. The page describing a single record lookup is strong context
for the one story that reads it and noise in the epic. Link it on the story.

A rule that is load-bearing for this story and written down nowhere is a
finding. Mark it (!) Todo with the question and name who can settle it. Do not
restate a rule you found on a documentation page. Link it, and write the
behaviour a tester can check.

---

## The value test

Name what someone can do after this story that they could not do before. If the
honest answer is "nothing yet, the next story needs it", this is not a story. It
is part of another story, and it should be folded in.

That question does more work than any rule about slicing. Apply it before writing
anything else.

---

## Title and goal

The title is a summary someone reads to understand what is happening. It is not
a description of the change to the code or the database. The goal line follows
the same discipline.

Both describe what changes for a person, in the words that person would use.

**The test.** Could a subject matter expert, an operations manager or a tester
read the title and know roughly what they would see? If it only makes sense with
the code open, rewrite it.

Keep out of the title and goal:

- Table, column and field names.
- Endpoints, routes and payloads.
- Class, service, component and library names.
- Build verbs: implement, refactor, wire up, configure, integrate, expose, hook
  up, migrate.
- Patterns and architecture words: saga, state machine, idempotency, enum,
  middleware.

Business names are the exception. Where a system or artefact is the name people
actually say, use it: the CRM by its product name, the customer portal, the
order reference, the itinerary. The line is between a name the business uses and
a name that exists only in the code.

Examples of what to avoid, and what to write instead:

| Avoid | Write |
|---|---|
| Implement POST /api/requests with idempotency key handling | A customer can submit their request |
| Add request_source enum to orders table | Support can see which requests came in online |
| Refactor checkout to support atomic multi-item writes | A customer can order several items in one go |

The technical naming is not lost. It belongs in the implementation notes at the
bottom, where the person building it will look for it.

Conditions and qualifiers stay out of the title too. "A customer can submit their
request" is the title. That it may only happen once, and that the customer gets a
confirmation, are detail and acceptance criteria. A title carrying a condition is
two facts stapled together, and the second one gets lost.

---

## Language

Readers switch off when writing looks generated, so the rules are firm.

- **No em-dashes.** Use a comma, a colon, a semicolon, brackets or a full stop.
- **State the case, do not argue it.** No "this is not X, it is Y", no "it isn't
  about X, it's about Y". Write the requirement and stop.
- **Words to avoid**: seamlessly, robust, leverage, ensure, simply,
  comprehensive, it is worth noting, importantly, critically.
- **Do not open every criterion with the same verb.** It reads as generated even
  when the content is right.
- **No bolded fragments used for emphasis.** Bold labels a field, nothing else.
- Bullets for anything enumerable. Narrative only for Detail and Demo, where the
  connection between sentences carries meaning.
- British English.

---

## What is assumed and never written

These are covered by the design system and by the team's standards. Writing them
into a story adds length and teaches readers to skim.

- Accessibility.
- Tested code.
- Established patterns from the design system.
- Security and data handling rules that apply to everything.

Write one of these into a story only when this story changes the rule or breaks
the pattern. Then say which, and why.

---

## Structure

Order matters. Why it exists, what is still unresolved, then how to test it, then
how it was decided to build it. A tester reads the top and stops before the last
section.

**The story stands alone.** The epic link is there for completeness, so a
reviewer can see where the work sits. Nothing in the story may depend on someone
following it. Where a tester needs a fact that lives in the epic, put the fact in
the story in a few words rather than pointing at it. This is not licence to
restate the epic's problem and context, which stays where it is.

```markdown
# [Title: the behaviour, not the component]

**Goal.** [Actor] can [do the thing]. One sentence, naming the actor.

**Epic.** [link] · Serves [epic criterion, where the epic numbers them]

**Open questions**

- (!) Todo **Blocking.** [The decision that has to be made before this can be
  built, why it blocks, and who can settle it.]

**Detail**

[Two to four sentences. What a reader needs that the goal line does not carry:
what happens today, what changes, anything a tester would otherwise guess at.]

**Design.** [design frame or prototype link, n/a, or (!) Todo and the question]

**Rules.** [page and section governing this behaviour, or n/a]

**Decisions.** [ADR or decision record constraining this story, or n/a]

**Test setup**

- [What must already exist before a tester can start]
- [Which account, role or record to use]
- [Test data, named specifically]

**Demo**

[Who sits down and what they see first, in one or two sentences. Then the steps
as bullets, each naming what the tester does and where they look to confirm it
landed.]

**Acceptance criteria**

1. [Checkable statement]. Fails if [the observation that shows it false].
2. ...

**Out of scope**

[One line. What this story will not do.]

---

**Implementation notes**

[Decisions only. See the rule below.]
```

Omit Open questions when there are none. Do not leave the heading in place with
nothing under it, and do not put a reassuring line there instead.

---

## Output

Write each story to `{epic-slug}-S{n}-{story-slug}.md` in the working folder, and
rewrite that same file on each pass so there is one current version. One file per
story, since each one is pasted into its own ticket. Plain markdown, so it
survives a paste into the tracker's editor.

---

## Where we do not know

Mark it (!) Todo followed by the question. It shows up on the ticket, rendering
as a warning icon in Jira, and the human reviewer can see what is missing at a
glance.

Write the marker as normal text. Code formatting around it stops Jira rendering
the warning icon.

Two places take a marker, and the difference is what kind of gap it is.

**Open questions, at the top.** A decision nobody has made, which the build
cannot proceed without. It goes above Detail so the reader meets it before
anything else, marked Blocking, with the consequence stated and an owner named
where there is one.

```
Open questions
- (!) Todo Blocking. How long does the confirmation link stay valid, and does it
  work on a second visit? Nothing is written until confirm, so a link that stops
  working on first use locks the customer out with no order to show for it. Until
  this is settled a tester cannot tell whether a second visit is a pass or a fail.
```

**Test setup, in place.** A gap in the tester's shopping list: an account, a
record or a piece of data nobody has named yet. It sits on the line it belongs
to.

```
Test setup
- A customer who has reached the review screen with a plan selected
- (!) Todo: which test account has an open request in the queue?
```

Rules for using it:

- Name the question. (!) Todo on its own tells the reviewer nothing.
- Never write a plausible answer instead. A marked gap gets closed, a confident
  guess does not.
- Where someone can settle it, name them.
- A story is not ready to build when a tester could not start it, or could not
  tell what correct looks like. Judge it on that, not on which section the marker
  sits in: the same open question can be phrased as background or as a
  precondition, and readiness must not depend on the author's choice of section.
- Say which stories are not ready on handover, and what unblocks each.

---

## Test setup and demo

Test setup is the tester's shopping list and nothing else: what to have, who to
be, what data to use. It is what has to exist before someone can run the demo and
check the criteria, not what the team needs in order to build. It is the section
most often missing, and a tester without it spends twenty minutes building a
scenario and guessing whether it matches what you meant.

A fact a reader needs in order to understand the behaviour belongs in Detail even
when it is also a precondition. A booking with nobody assigned to it is the point
of the story and a state the tester must construct. Say it in both places and
accept the one line of overlap.

- Name the starting state, the role or account, and the test data.
- Where the state comes from another story, say which one.
- Say what is not needed where a reader would otherwise go and build it.
- If test setup cannot be described, the story is not ready. Say so rather than
  writing a vague line.

The demo opens with a sentence or two putting a person in front of a screen, then
turns to bullets for the steps. It names the surface and the record. Half of
testing failure is a tester seeing the right behaviour on screen and not knowing
where to check the effect.

---

## Acceptance criteria

Every story has them. They are the contract with the tester.

- One criterion per behaviour, each independently checkable.
- Each carries the observation that would show it false. Write it down; a test
  the reviewer cannot see is a test nobody ran.
- Given/When/Then only where the starting state changes the outcome. A plain
  checkable sentence is usually clearer, and clarity wins.
- Include one criterion for the failure that matters: a validation, a permission,
  a rejection. Do not enumerate edge cases. A story whose criteria only describe
  success gets signed off by someone who never tried to break it.
- Four to six is the useful range. Past eight, the story is too big.

**Criteria must be checkable without reading the implementation notes.** If a
tester has to scroll to the bottom to learn what correct means, the separation
has failed and a requirement is sitting in the wrong section.

Where an Open question would change what a criterion means, write the criteria
that hold whichever way it is answered and leave the rest to the marker. Do not
write a criterion that quietly assumes one answer.

---

## Implementation notes

This section carries decisions: the approach chosen, the component or service to
use, a data shape, a library, a pattern to follow, a trap to avoid.

**The test: delete the section, and the story still builds and still tests.** It
holds decisions, not obligations. Anything that would break the story if ignored
is a requirement, and requirements go in the acceptance criteria.

Watch for the requirement whose failure this story's demo cannot show. Versioning
that only matters on the second upload, a field name that only breaks when
another team reads it, a signature that must be a deliberate act rather than a
side effect of closing a record. These read like technical advice and are
obligations, so they still go in the criteria. Name where they are observed, even
where that is a later story or another team's check, rather than leaving them as
a note nobody has to satisfy.

Two other things do not belong here. An open question that blocks the work goes
in Open questions at the top, not in an architecture musing in the notes. A
disagreement with the epic's slicing goes on the ticket with a (!) marker and in
the handover, not in a section the reader is told they can drop.

---

## Writing a set of stories

Stories written one at a time drift. A tester reading ten in a sitting notices.

- Fix the vocabulary on the first story and hold it: the same name for the same
  field, screen, record and role throughout.
- Hold the criteria shape too. Do not mix Given/When/Then and plain statements
  across a set without reason.
- Name things as the system names them, not as the epic describes them.
- Where a story depends on another, say which by title or id, and keep the first
  story in the set free of blockers.
- Where two stories in the set are not independently demoable, say so on
  handover and name the pair, rather than letting a sprint pick up the half with
  nothing to show.

---

## Length

The story above the implementation notes reads in under a minute. Around 300
words for goal, detail, test setup, demo and criteria together.

Criteria are what cost. Each one carrying its failure observation runs 20 to 30
words, so six criteria are 150 before anything else is written. A simple UI story
with four criteria lands near 200. A records or signature story lands near 350
and that is not padding. Do not cut a failure observation or a line of test setup
to hit a number.

Brevity comes from leaving out what belongs elsewhere, not from thinning the
sections a tester needs. Delete a sentence and ask whether anyone would build or
test differently. If not, it was padding.

Open questions and implementation notes sit outside that budget. A blocking
question is worth the three or four lines it takes to say what is unresolved and
what breaks if it is guessed at.

---

## Quality checks

- The value test is answered: someone can do something new.
- The title and goal name what changes for a person, with no table, endpoint,
  class or pattern names, and no build verbs.
- The title carries no conditions or qualifiers. They sit in detail or the
  acceptance criteria.
- Every blocking decision sits in Open questions at the top, marked Blocking,
  with its consequence stated.
- Test setup names a starting state, a role and test data.
- The demo names where the tester confirms the effect.
- Every criterion carries its failure observation.
- One criterion covers the failure that matters.
- No criterion assumes an answer to an Open question.
- Criteria can be checked without reading the implementation notes.
- Deleting the implementation notes leaves a story that still builds and tests.
- No accessibility, testing or design system boilerplate.
- No em-dashes, no banned words, nothing argued rather than stated.
- Out of scope is one line and is present.
- A tester who never opens the epic can still run the story.
- Every source named on the story carries a link, or a visible (!) Todo where
  the link is missing.
- Every gap is (!) Todo with the question named, written as normal text rather
  than code formatting, and nothing plausible has been written in place of an
  answer.
- Any (!) Todo in Open questions, Test setup or the acceptance criteria is
  called out on handover as not ready to build.

---

## What this does not cover

This skill keeps a story readable, valuable and testable by a person. It does not
carry the engineering a developer needs settled before starting. Where it is used
alongside a technical story skill, that skill owns:

- Which system changes, and which systems it talks to.
- What triggers the behaviour.
- What happens on failure, timeout and retry, and whether a retry is safe to run
  twice.
- Concurrency: two people in the same record, a double submit.
- Throughput, retention and cost at expected volume.
- What is emitted so the epic's metrics can be measured.
- Where data goes, and which environment and endpoint.

A story with no stated trigger and no named system is not finished, it is half
finished. Say which half is missing rather than letting it look complete.

---

## What this does not do

- Does not write the epic, or restate its problem and context. The story links up.
- Does not create or update tracker issues, and does not comment on them. It
  produces a file for the human to review and paste; the human owns what lands in
  the tracker.
- Does not estimate or assign.
- Does not reslice. Where a story map or epic set the slicing, that is the
  source of truth, and a disagreement is raised rather than resolved quietly.
- Does not invent test setup, test data or design links to fill a section. A gap
  is marked (!) Todo with the question, and given an owner where there is one.

---

## Changelog

- **v1.0 (2026-09-14).** First version. The worked examples are domain-neutral,
  the tool names sit in a Tools section so a team can substitute their own, and
  the technical story skill is named as a role rather than a specific skill,
  since none ships in this set.
