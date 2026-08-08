---
name: workout
description: Propose a short, working-day-friendly workout tailored to the user's saved fitness profile (exercise types, level, equipment, time, goals, injuries) AND their chosen trainer persona. Use when the user asks for a workout, "what should I do today", "give me a quick set", "suggest exercises for my break", or similar. Reads ~/workout-coach/profile.md and ~/workout-coach/trainer.md.
user-invocable: true
allowed-tools:
  - Read
  - Edit
  - Bash(ls *)
  - Bash(cat *)
  - Bash(date *)
  - Bash(head *)
---

# /workout — Propose a short workout

Generate a concise workout the user can do during a working-day break,
tailored to their profile.

## Steps

1. **Load the profile, trainer, and plan.** Run
   `cat ~/workout-coach/profile.md`, `cat ~/workout-coach/trainer.md`, and
   `cat ~/workout-coach/plan.md` (the plan may not exist yet).
   - If the **profile** doesn't exist, tell the user to run `/workout-profile`
     first (or offer to capture a quick profile inline by asking 2–3
     questions). Don't guess a full profile silently.
   - If the **trainer** doesn't exist, proceed with a neutral, balanced coach,
     and mention they can run `/workout-trainer` to pick a persona (e.g. Bruce
     Lee, Saitama, Arnold, Yoga Instructor, Physio/Mobility Coach) for
     style-flavored workouts.

2. **Pick the session source & account for context.** Get today's date from the
   shell with `date +%F` (don't trust a remembered date), then factor in:
   - **Default to the plan.** If `~/workout-coach/plan.md` exists, it is the
     default source: work out which session the plan schedules next — match
     today's weekday in its weekly schedule, skip any slots already logged
     today, honor scheduling rules baked into the profile/plan (e.g. pull-ups
     only after 18:00 and only with a run), and prescribe the plan's current
     **progression targets** for those movements. Build *that* session.
     **Unless the user specifies otherwise** — if they ask for a different
     focus, a different duration, or to ignore the plan, honor that and tell
     them the session is *off-plan* (so `/workout-log` won't tick it against the
     plan). If no plan exists, build from the profile as usual and mention they
     can run `/workout-plan` to get a structured program.
   - **The trainer persona**, if set — let it shape the *style*, exercise
     selection bias, and tone of the workout (e.g. Saitama → simple high-rep
     calisthenics; Yoga Instructor → breath-led mobility flow). The persona
     flavors the set; it never overrides the profile's limits.
   - How much time they have right now (ask if unclear — default to the
     profile's "time per session").
   - **What they've already done today** — read today's log
     (`cat ~/workout-coach/logs/$(date +%F).yaml 2>/dev/null`; today's file is
     YAML). If sessions exist, avoid hammering the same muscle groups twice,
     nudge toward variety/balance, and note which session number this is vs.
     their target frequency.
   - **Recent history & progression** — list and skim the last few daily logs
     (`ls -t ~/workout-coach/logs/ 2>/dev/null | head -5`, then `cat` them).
     Logs are **YAML** (`<date>.yaml`, a `sessions:` list). Use the
     planned-vs-actual signal to scale: if they've consistently **met or beat**
     targets (`met: true`) on a movement, nudge reps/load/duration up a notch;
     if they've **fallen short** repeatedly, ease it back. This is how the
     log's data feeds back into the suggestion — close the loop. When a plan
     exists, its **progression targets** are the prescribed loads — use the logs
     to fine-tune within them, and let `/workout-log` ratchet the plan itself.
   - Their stated level, equipment, goals, and **injuries to avoid**.
   - **Today's condition** — the profile's limitations may include *temporary*
     niggles. Briefly ask if anything's bothering them today (e.g. "neck still
     stiff?") and adjust, so a healed niggle doesn't suppress work forever.
   - **Keep the profile current.** If the user volunteers *durable*
     health/fitness info — a niggle that has resolved or newly appeared, a
     change in level, equipment, or a PB/target — **propose updating
     `~/workout-coach/profile.md` and confirm before editing**, then apply the
     change with `Edit` and bump the `_Last updated_` date (`date +%F`). Only
     persist lasting changes this way; one-off day-of feelings belong in the
     log, not the profile. Never rewrite the profile silently.

3. **Design ONE workout**, scaled to their level and time budget. Keep it
   short (usually 3–6 working exercises, plus a warm-up). For each exercise
   decide: name, sets × reps (or a duration for holds/cardio), a one-line
   form/scaling cue, and a **short explanation** of what it does and why it's
   in today's set. If a trainer is set, write the set in that persona's voice
   and lean its exercise choices that way (without overdoing the roleplay —
   keep it useful first).

4. **Always open with a warm-up.** Put a warm-up section *first*, before the
   working sets. Scale it to the work ahead and to today's condition — a
   couple of minutes of mobility/activation for the muscle groups about to be
   loaded is the minimum; recommend more if the session is intense, the user
   is cold (e.g. first session of the day), or a niggle is present. Never skip
   it silently; if you genuinely judge none is needed, say so explicitly.

5. **Estimate the duration and check it against their preference.** Before
   presenting, do a quick time budget for the whole session and confirm it
   fits the user's stated time per session (or the time they gave for right
   now). For each exercise estimate:
   - **Work time:** reps × ~3s/rep (or the hold/cardio duration) × sets.
   - **Rest time:** rest between sets (typically 30–60s for strength,
     15–30s for mobility) × (sets − 1).

   Add the warm-up, sum it all, and compare to the budget:
   - If the total **overshoots**, trim — cut a set, drop an exercise, or
     shorten rests — until it fits, then re-check.
   - If it **undershoots** noticeably, you have room to add a set or an
     exercise.

   Show the user a one-line total (e.g. "≈12 min incl. rests — fits your
   15-min window") so the check is visible, not hidden.

6. **Take safety seriously.**
   - Order exercises sensibly: warm-up first, then heavier/more technical or
     explosive work while fresh, finishing with lighter/core/cooldown.
   - Default to movements appropriate for the user's level. **Avoid dangerous
     or high-injury-risk exercises** (heavy overhead/loaded spine work,
     ballistic/plyometric moves, deep loaded ranges, anything that stresses a
     listed injury). Only include such a movement if the profile shows the
     user is genuinely experienced with it **and** you flag the risk and the
     safety cues clearly — never slip it in unwarned.
   - Always honor injuries and temporary niggles; when in doubt, pick the
     safer regression.

7. **Present it cleanly**, exercises in order, warm-up first, each with its
   short explanation. For example:

   ```
   ### Today's set — ≈12 min (incl. rests, fits your 15-min window), upper body

   **Warm-up (~2 min)**
   - Arm circles + band pull-aparts — 1 min
     Wakes up the shoulders and upper back so the pressing work is safe.

   **Working sets**
   1. Push-ups — 3 × 12   (drop to knees if form breaks)
      Main chest/triceps builder; the staple horizontal press.
   2. Chair dips — 3 × 10  (keep elbows tracking back)
      Hits triceps and front delts — uses just your chair.
   3. Band rows — 3 × 15
      Balances all the pushing and counters desk-hunched posture.
   4. Plank — 3 × 30s
      Bracing core finisher; keep hips level, don't sag.
   ```

8. **Offer to log it.** End by telling the user that once they've done it,
   they can run `/workout-log` and report how many reps they actually
   completed — you'll record it against today's date.

## Principles

- Respect the remote-work constraint: quiet, low-space, minimal-equipment,
  finishable in one break. No 45-minute programs.
- Match difficulty to the profile — don't prescribe 50 push-ups to a beginner.
- Always honor the injuries/limitations section.
- **Safety first.** Warm up before loading, order exercises from
  technical/explosive to light, and never prescribe a dangerous movement
  unless the user is clearly experienced with it and explicitly warned. When
  unsure, choose the safer regression.
- **Always fit the clock.** Estimate the real duration including rest between
  sets, compare it to the user's time budget, and adjust until it fits — then
  show the total so they can trust it.
- **Explain each exercise** briefly so the user knows what it's for and how to
  do it safely — extra detail the first time they meet a movement.
- The trainer flavors **style and selection**, never safety — the profile's
  limits and injuries always override the persona.
- **Follow the plan by default.** When `plan.md` exists, pull today's session
  from the next thing it schedules; go off-plan only when the user asks, and say
  so when you do (so the log knows not to credit it against the plan).
- Keep it actionable, not a lecture. One workout, ready to start.
