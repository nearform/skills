---
name: workout-profile
description: Gather and save a remote-worker's fitness profile — preferred exercise types, how often they want to train, current fitness level, available equipment/space, time per session, goals, and any injuries. Use when the user wants to set up, create, or update their workout profile, or says things like "set up my workout plan", "I want to start exercising at my desk", or before proposing/logging workouts if no profile exists yet.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - Bash(mkdir -p *)
  - Bash(ls *)
  - Bash(cat *)
  - Bash(date *)
---

# /workout-profile — Build the user's fitness profile

This skill interviews the user and writes a profile to
`~/workout-coach/profile.md`. The other workout skills (`workout`,
`workout-log`) read this file, so keep it accurate and well-structured.

## Context

The user works remotely and wants **short workouts that fit inside a working
day** — typically 5–15 minute sets done between meetings, with limited space
and equipment. Frame every question with that in mind.

## Steps

1. **Check for an existing profile.** Run `cat ~/workout-coach/profile.md`
   (it may not exist). If it exists, this is an *update* — show the current
   values and ask what they want to change rather than starting from scratch.

2. **Interview the user.** Ask conversationally, a few related questions at a
   time — don't dump all of these at once. Cover:
   - **Exercise types they enjoy / want** — e.g. bodyweight strength,
     resistance bands, dumbbells/kettlebell, mobility & stretching, desk-based
     movement, light cardio. Multiple is fine.
   - **Frequency** — how many short sessions per day and/or days per week.
   - **Current fitness level** — beginner / intermediate / advanced, plus a
     sentence on what they can currently do (e.g. "can do 10 push-ups").
   - **Equipment & space** — what they have at their desk/home and how much
     room (e.g. "just a chair and a yoga mat", "adjustable dumbbells").
   - **Time per session** — minutes they realistically have per break.
   - **Goals** — e.g. counter sitting, build strength, mobility, energy.
     Also ask explicitly whether they have any **specific PB / measurable
     targets** they're chasing (e.g. "10 pull-ups in a row", "sub-6:00 mile",
     "half marathon in 2:10", "100 push-ups"). Capture each with a current
     best where known, so `/workout` can program toward them and progress is
     trackable. "None yet" is a fine answer.
   - **Injuries / limitations** — distinguish **permanent** things to avoid
     (e.g. a chronic bad back) from **temporary** niggles that should clear up
     (e.g. "neck stiff this week"). The temporary ones shouldn't suppress
     exercises forever.

   Use sensible defaults if the user is vague, and confirm them. Don't
   interrogate — 2–4 short exchanges is plenty.

3. **Write the profile** to `~/workout-coach/profile.md` using the template
   below. Run `mkdir -p ~/workout-coach` first if needed. Fill in only what
   you learned; leave honest "not specified" for anything skipped.

4. **Confirm** what you saved and point to the next steps: they can pick a
   coaching persona with `/workout-trainer` (optional), then run `/workout` to
   get a session.

## Profile template

```markdown
# Workout Profile

_Last updated: <YYYY-MM-DD>_

## Fitness level
<beginner | intermediate | advanced> — <one line on current baseline>

## Preferred exercise types
- <type 1>
- <type 2>

## Frequency
<e.g. 2 short sessions per workday, ~4 days/week>

## Time per session
<e.g. 10 minutes>

## Equipment & space
<e.g. yoga mat, resistance band, ~2m of floor space, a sturdy chair>

## Goals
<e.g. break up sitting, build upper-body strength>

### PB targets
<specific measurable goals with current best where known, e.g.
"Pull-ups: 10 in a row (current best: 8)", "1 mile run: sub-6:00". "none yet" if so>

## Permanent limitations
<lasting things to always avoid, e.g. sensitive lower back — avoid heavy spinal loading. "none" if so>

## Current / temporary niggles
<short-term issues to work around for now, e.g. stiff neck this week. Clear these once resolved. "none" if so>

## Notes
<anything else worth remembering>
```

Get the current date from the shell with `date +%F` for the `_Last updated_`
field (don't trust a remembered date). Keep the file tidy — it's the single
source of truth.
