---
name: workout-plan
description: Generate (or revise) a structured, multi-session training plan from the user's profile and trainer persona, oriented toward their goals and PB targets. Writes ~/workout-coach/plan.md — a repeating weekly schedule plus progression targets that `workout` pulls today's session from and `workout-log` ratchets as sessions are completed. Use when the user says "make a plan", "build my training plan", "plan my week", "create a program", or wants a structured schedule rather than a single session.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash(mkdir -p *)
  - Bash(ls *)
  - Bash(cat *)
  - Bash(date *)
  - Bash(head *)
---

# /workout-plan — Build the user's training plan

This skill turns the profile + trainer into a structured program and writes it
to `~/workout-coach/plan.md`. The plan is a **living template**, not a fixed
calendar: a repeating weekly schedule plus a set of **progression targets** that
ratchet over time.

It sits at the center of the workout loop:
- **`workout`** reads `plan.md` and, by default, builds today's session from the
  next thing the plan schedules (unless the user asks for something else).
- **`workout-log`** updates `plan.md` when a logged session was taken from the
  plan — ticking adherence and ratcheting the progression targets.
- **`workout-profile`** / **`workout-trainer`** are the inputs; re-run this skill
  after they change so the plan stays aligned.

## Steps

1. **Load the inputs.**
   - `cat ~/workout-coach/profile.md` — **required.** If it doesn't exist, tell
     the user to run `/workout-profile` first; don't invent a profile.
   - `cat ~/workout-coach/trainer.md` — optional. If absent, use a neutral,
     balanced coach and mention `/workout-trainer`.
   - `cat ~/workout-coach/plan.md` — if it **exists**, this is a *revision*:
     summarize the current plan and ask what to change (progress the block,
     swap a focus, account for a profile change) rather than silently
     overwriting. Preserve the revision log.
   - Skim recent logs for current baselines
     (`ls -t ~/workout-coach/logs/ 2>/dev/null | head -5`, then `cat` a few).
     Logs are **YAML** (`<date>.yaml`, a `sessions:` list).
   - Get the date with `date +%F`. For the `## This week (adherence)` heading,
     also compute this week's Monday from the shell rather than by hand —
     `date -v-$(($(date +%u)-1))d +%F` on macOS, or
     `date -d "-$(($(date +%u)-1)) days" +%F` on GNU/Linux; don't trust a
     remembered date.

2. **Set the goal focus and prioritize.** Pull the goals and **PB targets** from
   the profile. Where goals compete for recovery (classic case: heavy leg work
   vs. running, or max-strength vs. endurance), say so plainly and **rank them**
   — the plan should clearly serve the top priorities and keep the rest in
   maintenance. Don't pretend five goals can all be peaked at once.

3. **Choose the methodology per goal**, scaled to the user's level:
   - Calisthenics rep goals (e.g. more push-ups/pull-ups) → frequent
     submaximal volume ("grease the groove"), which fits a many-short-sessions
     day especially well.
   - Running speed (e.g. a faster mile) → intervals + strides.
   - Running endurance (e.g. a half marathon) → a progressive weekly long run +
     tempo work.
   - Strength/size → progressive load within the available equipment.
   Let the **trainer persona** flavor *style and exercise selection* (e.g. Bruce
   Lee → crisp, never-to-failure GtG and mobility woven between sets), but never
   let it override the profile's limits or safety.

4. **Honor the profile's constraints — these are hard rules, not suggestions:**
   - **Frequency & time:** fit the user's sessions/day and minutes/session.
     Each planned session must be self-contained and inside the time budget.
   - **Equipment & location rules:** only program what they have, where they
     have it. Respect any **scheduling rules** in the profile (e.g. "pull-ups
     only after 18:00 and only when a run is involved" → schedule pull-ups as
     evening run-to-the-bar sessions, never as a daytime desk slot).
   - **Permanent limitations & current niggles:** work around them; pick the
     safer regression when unsure.
   - **Off-clock work:** if a goal needs a session that can't fit a normal slot
     (e.g. a 16 km long run), schedule it explicitly outside the working day and
     say so — don't pretend it fits a 15-minute break.

5. **Sequence and recover.** Order the week so competing systems aren't stacked
   (e.g. don't put a hard squat session the day before a hard run, or hammer the
   same muscle group two slots running). Build in at least one easy/mobility or
   rest provision per week, and a **deload** rule (e.g. ease ~30% every 4th
   week) so adaptation sticks.

6. **Set progression targets.** For each key movement, write the *current
   working number* the plan prescribes now, plus the **ratchet rule** that moves
   it (e.g. "push-up GtG: 4×22 — +2 reps/set once all sets hit target two
   sessions running"; "long run: 8 km — +1 km/week, hold on deload weeks").
   These are the numbers `workout` prescribes and `workout-log` updates.

7. **Write the plan** to `~/workout-coach/plan.md` using the template below
   (`mkdir -p ~/workout-coach` first if needed). Stamp the created/updated date
   and which profile/trainer it's based on. On a revision, append to the
   revision log rather than wiping history, and reset `## This week`.

8. **Confirm and hand off.** Summarize the plan in a few lines, name the top
   priority and any honest tradeoffs, then point the user on:
   - run `/workout` to get today's session pulled from the plan (or ask for
     something specific to go off-plan),
   - run `/workout-log` to record it — completed planned sessions ratchet the
     plan automatically,
   - re-run `/workout-plan` after a profile change, or every few weeks to
     progress the block.

## Plan template

```markdown
# Workout Plan

_Created: <YYYY-MM-DD> · Last updated: <YYYY-MM-DD>_
_Based on: profile.md (<profile's last-updated date>) · trainer: <persona or none>_
_Block: <e.g. 4-week progression — week 1 of 4>_

## Goal focus
Prioritized targets this plan drives toward:
1. <goal / PB> — <why it's first, the strategy in a line>
2. <goal / PB> — <...>
<note any goals kept in maintenance, and any tradeoff being made>

## Approach
<2–5 lines: the methodology — e.g. grease-the-groove for push/pull, 3 runs/week
(intervals / tempo / long), leg sequencing, deload every 4th week. Note the
trainer flavor.>

## Weekly schedule
Repeating week. <constraints recap, e.g. workday = 4 × ~15-min slots; pull-ups
only after 18:00 and only with a run; long run is off-clock on the weekend.>

### Monday — <theme>
- Slot 1 — <focus>: <movements @ targets>  (<one-line cue>)
- Slot 2 — <...>
- Slot 3 — <...>
- Slot 4 — <...>
### Tuesday — <theme>
- ...
### ... (through the week, including rest/mobility days)
### Sunday — <theme or rest>
- ...

## Progression targets
Current working numbers — `workout` prescribes these; `workout-log` ratchets them:
- <movement>: <current target>  — _ratchet:_ <rule>
- <movement>: <current target>  — _ratchet:_ <rule>
- ...

## This week (adherence)
Week of <Monday's date> — `workout-log` records sessions taken from the plan here;
`workout-plan` resets this on each revision.
- (none yet)

## Revision log
- <YYYY-MM-DD>: plan created from profile (<profile date>), trainer <persona>.
```

## Principles

- **The plan is a living template + progression state**, not a frozen calendar.
  It encodes *what to do* and *the numbers to do it at*; the numbers move as the
  log proves the user is ready.
- **The profile and safety always win.** The trainer flavors style and
  selection; it never overrides limits, injuries, equipment, or scheduling
  rules. When unsure, choose the safer regression.
- **Prioritize honestly.** When goals compete, rank them and keep the rest in
  maintenance — don't promise to peak everything at once.
- **Respect the clock and the calendar.** Every planned session fits the user's
  slot/time budget; anything that can't (e.g. a long run) is scheduled off-clock
  and labeled as such.
- **Sequence for recovery.** Don't stack competing systems; bake in easy days
  and a deload.
- **Keep `plan.md` tidy and skimmable** — `workout` and `workout-log` parse it,
  so keep the section headings and the progression-target format stable.
