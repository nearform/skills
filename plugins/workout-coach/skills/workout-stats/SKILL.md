---
name: workout-stats
description: Summarize the user's recent workout history — sessions this week, current streak, planned-vs-actual hit rate, muscle-group balance, and progression trends. Reads the per-day logs in ~/workout-coach/logs/ (plus profile.md for targets). Use when the user asks "how am I doing", "show my stats", "what's my streak", "weekly summary", "am I making progress", or similar.
user-invocable: true
allowed-tools:
  - Read
  - Bash(ls *)
  - Bash(cat *)
  - Bash(date *)
---

# /workout-stats — Summarize recent workout history

Give the user a short, motivating read on how their training is going, based on
the logs that `/workout-log` writes. Read-only — this skill never writes.

## Steps

1. **Gather the data.** Get today's date (`date +%F`). Then:
   - List the log files: `ls -t ~/workout-coach/logs/ 2>/dev/null`. If there
     are none, say so warmly and point them to `/workout` to get started —
     don't fabricate stats.
   - Read the recent ones (e.g. the last 7–14 days):
     `cat ~/workout-coach/logs/<file>.yaml` for each. Each log is **YAML** — a
     `sessions:` list of structured entries; an exercise's `met: true/false`
     field marks whether the planned target was hit.
   - Read `~/workout-coach/profile.md 2>/dev/null` for their target frequency,
     goals, and limitations (to judge "on track").

2. **Compute the summary.** From the logs, work out:
   - **This week** — number of sessions vs. the profile's target frequency.
   - **Streak** — consecutive days (or workdays) with at least one session, up
     to today.
   - **Volume / balance** — rough split across muscle groups or session types
     (e.g. mobility vs. upper vs. lower), flagging anything neglected.
   - **Progression** — for recurring movements, are reps/load/duration trending
     up? Use the planned-vs-actual `met` signal.
   - **Effort** — if RPE / effort notes exist, note the trend (cruising vs.
     grinding).

3. **Present it concisely.** A short scannable summary, e.g.:

   ```
   ### Last 7 days
   - Sessions: 5 / target ~4–8  ✅ on track
   - Streak: 3 days
   - Balance: 3× mobility, 1× upper, 1× lower — legs a bit light
   - Progress: push-ups 12 → 14 → 15 (trending up 💪)
   - Effort: mostly RPE 6–7, one tough one
   ```

   Add one or two honest, encouraging takeaways and, if useful, a concrete
   nudge (e.g. "due for a lower-body set — try `/workout`").

## Principles

- **Read-only and truthful** — summarize only what the logs actually show; if
  data is thin, say the picture is limited rather than inventing trends.
- Keep it short and motivating, not a spreadsheet dump.
- Respect the profile's goals and limitations when judging "on track" or
  suggesting what's next.
