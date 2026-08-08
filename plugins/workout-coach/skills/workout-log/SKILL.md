---
name: workout-log
description: Log a completed workout. The user reports how many reps/sets they actually did (usually against a workout that was just suggested), and this records it to a per-day YAML file at ~/workout-coach/logs/YYYY-MM-DD.yaml. Use when the user says "log my workout", "I did X reps", "done", "I finished my set", or wants to record exercise they completed.
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

# /workout-log — Record a completed workout

Append a short session to the user's daily log. One file per day; a working
day can hold several short sessions. Logs are stored as **YAML**
(`~/workout-coach/logs/<date>.yaml`) so they're machine-parseable yet still
readable by hand.

## Steps

1. **Determine today's date.** Run `date +%F` to get `YYYY-MM-DD`. Use this
   for the filename and run `date +%H:%M` for the session time. (Don't trust a
   remembered date — read it from the shell.)

2. **Figure out what was done.** Ideally this follows a `/workout`
   proposal — match the user's reported reps against the planned exercises so
   you can show planned-vs-actual. If there was no prior proposal, just ask
   what they did (exercise, sets × reps) and record that.

   Capture for each exercise: name, sets × actual reps, and (if known) the
   planned target. Note anything the user mentions (felt easy/hard, pain).
   Also capture, for the session as a whole:
   - **Trainer** — the active persona, if any
     (`cat ~/workout-coach/trainer.md 2>/dev/null` → the `## Trainer` line).
   - **Effort** — an optional perceived-exertion note (e.g. RPE 7/10, or
     "felt easy / felt brutal"). Ask briefly if the user didn't say.

3. **Write to `~/workout-coach/logs/<today>.yaml`.**
   - Run `mkdir -p ~/workout-coach/logs` first.
   - If the file **doesn't exist**, create it with `date: <today>` and a
     `sessions:` list containing this one session.
   - If it **already exists**, read it first, then **append a new entry to the
     `sessions:` list**; do NOT overwrite or drop earlier sessions from the
     same day.
   - Keep the YAML valid: quote any value containing `:` or `#` (e.g. times
     like `"13:27"` and planned strings like `"4x10-12s"`), and use the schema
     below. Omit fields that don't apply rather than leaving them blank.

4. **Confirm** what you logged and give one short, encouraging takeaway (e.g.
   progress vs. last session if visible). Offer another set later via
   `/workout` if appropriate.

5. **Keep the profile current.** If, while reporting the session, the user
   volunteers *durable* health/fitness info — a niggle that has resolved or
   newly appeared, a change in level/equipment, or a hit/new PB or target —
   **propose updating `~/workout-coach/profile.md` and confirm before
   editing**, then apply it with `Edit` and bump the `_Last updated_` date
   (`date +%F`). For PBs, also update the relevant "current best" under the
   profile's PB targets. Only persist lasting changes; one-off effort/feeling
   notes stay in the log, not the profile. Never rewrite the profile silently.

6. **Update the plan if the session came from it.** If `~/workout-coach/plan.md`
   exists (`cat` it), decide whether the session just logged was **taken from
   the plan** — i.e. it matches a session the plan schedules for today, or the
   `/workout` proposal / conversation indicates it was the planned one. A
   session the user explicitly did *off-plan* does **not** count.
   If it was taken from the plan:
   - **Tick adherence:** under the plan's `## This week (adherence)` section,
     record the completed session with today's date (append a line; if the week
     just rolled over, add a `Week of <this Monday's date>` heading). Compute
     that Monday from the shell rather than by hand —
     `date -v-$(($(date +%u)-1))d +%F` on macOS, or
     `date -d "-$(($(date +%u)-1)) days" +%F` on GNU/Linux — the same way
     today's date is read; don't trust a remembered date.
   - **Ratchet progression:** compare the actual reps/sets/time against the
     plan's `## Progression targets` for those movements. If the user **met or
     beat** the target per that movement's ratchet rule, bump the target up a
     notch; if they **fell short** repeatedly, ease it back. Record the change
     in the plan's `## Revision log` with today's date.
   - Bump the plan's `_Last updated_` date (`date +%F`).
   Make these as in-place `Edit`s — never rewrite the whole plan, and leave it
   untouched for off-plan sessions.

## Daily file format (YAML)

One file per day, `~/workout-coach/logs/<YYYY-MM-DD>.yaml`:

```yaml
date: 2026-06-15
sessions:
  - time: "13:27"                  # HH:MM, quoted
    label: Upper body & core       # short human label
    trainer: Bruce Lee             # active persona, or omit if none
    from_plan: Wed Slot 2          # plan slot it came from; omit if off-plan
    off_plan: false                # set true for an explicitly off-plan session
    effort: RPE 7/10               # perceived exertion, or omit
    exercises:
      - name: Push-ups
        sets: 4
        reps: [12, 12, 12, 12]     # per-set reps (a list), or a single int
        planned: "4x12"            # the prescribed target, quoted; omit if none
        met: true                  # true if met/beat target, false if under
      - name: L-sit hold
        sets: 4
        duration_s: [6, 5, 5, 6]   # timed holds, seconds per set
        planned: "4x10-12s"
        met: false
      - name: Run                  # cardio: use the fields that fit
        distance_km: 3
        duration: "17:45"
        pace: "5:55/km"
        met: true
    notes: nailed everything but L-sit — out of practice on it
  # append further same-day sessions as additional list items here
```

### Field guide
- **Per exercise**, always give `name`; then include only the fields that
  apply: `sets`, `reps` (int or per-set list), `duration_s` (timed holds),
  `distance_km` / `duration` / `pace` (cardio), `weight_kg` (loaded moves),
  `planned` (the prescribed target as a quoted string), `met` (bool), and an
  optional per-exercise `note`.
- **`met`** records whether the target was hit: `true` when the user met or
  beat the planned target, `false` when they came in under (no judgment — it's
  just a signal for future scaling). Omit it when there was no planned target.
- **`from_plan`** vs **`off_plan`**: set `from_plan` to the plan slot when the
  session was taken from `plan.md`; set `off_plan: true` for an explicitly
  off-plan session. (This is what step 6 keys off of.)

Keep entries terse. This file is the history that `/workout`, `/workout-stats`,
and `/workout-plan` read to balance future workouts and track progress.
