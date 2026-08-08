---
name: workout-trainer
description: Pick or create the user's workout "trainer" — a coaching persona (e.g. Bruce Lee, Saitama, Arnold, a Yoga Instructor, a Physio/Mobility Coach, or a custom one) that flavors how workouts are designed and described. Saves the choice to ~/workout-coach/trainer.md, which workout reads alongside the profile. Use when the user wants to choose, change, or create a trainer/coach persona, says "pick a trainer", "switch coach", "I want Arnold to train me", "make my own trainer", or similar.
user-invocable: true
allowed-tools:
  - Read
  - Write
  - Edit
  - AskUserQuestion
  - Bash(mkdir -p *)
  - Bash(ls *)
  - Bash(cat *)
  - Bash(date *)
---

# /workout-trainer — Choose or create a trainer

This skill picks the coaching **persona** that flavors the user's workouts. It
writes `~/workout-coach/trainer.md`; `workout` reads that file together
with `profile.md` so the *same* profile yields differently-styled workouts
depending on the active trainer.

A trainer changes **tone, exercise selection bias, and how a set is framed** —
it never overrides safety. The user's profile (level, equipment, time, and
especially **injuries/limitations**) always wins over a persona's style.

## The 5 default trainers

1. **Bruce Lee** — functional strength, speed, and mobility with a mind-body
   philosophy. Lean, efficient, precise movements; flow between strength and
   stretch. Motivational, philosophical tone.
2. **Saitama (One Punch Man)** — brutally simple, high-rep calisthenics
   (push-ups, sit-ups, squats, a run). Minimal equipment, maximal consistency.
   Deadpan, "just do it every day" tone.
3. **Arnold** — classic bodybuilding / hypertrophy with the dumbbells and band:
   moderate-heavy sets, mind-muscle focus, upper/lower splits. Big, encouraging,
   "come on, one more rep" tone.
4. **Yoga Instructor** — breath-led flexibility, balance, and flow; mobility and
   calm over intensity. Soothing, cue-rich, body-awareness tone.
5. **Physio / Mobility Coach** — longevity and desk-recovery focus: joint-friendly
   mobility, postural correction, gentle progressive strengthening. Careful,
   educational, "protect the joint" tone. (Great default for desk workers and
   anyone managing a niggle.)

## Steps

1. **Check for an existing trainer.** Run `cat ~/workout-coach/trainer.md`
   (it may not exist). If it exists, show the current trainer and ask whether
   they want to switch, tweak it, or create a new one.

2. **Let the user choose.** Offer the 5 defaults above (use AskUserQuestion),
   or a **"create my own"** option. Briefly remind them what each persona
   emphasizes so the choice is informed.

3. **If creating a custom trainer,** ask 2–4 short questions to capture:
   - **Name** of the trainer/persona.
   - **Training philosophy / focus** (e.g. strength, conditioning, mobility,
     calisthenics, hypertrophy).
   - **Style & tone** (e.g. intense drill-sergeant, calm and technical,
     playful, philosophical).
   - **Signature moves or biases** (anything they want emphasized or avoided).
   Use sensible defaults and confirm; don't interrogate.

4. **Write the trainer** to `~/workout-coach/trainer.md` using the template
   below (run `mkdir -p ~/workout-coach` first if needed). For a default
   trainer, fill it from the description above. For a custom one, fill from the
   interview.

5. **Confirm** what you saved and tell the user they can now run
   `/workout` to get a workout in this trainer's style, or
   `/workout-trainer` again any time to switch.

## Trainer file template

```markdown
# Workout Trainer

_Last updated: <YYYY-MM-DD>_

## Trainer
<persona name>  (<default | custom>)

## Philosophy & focus
<one or two lines on what this trainer emphasizes>

## Style & tone
<how the trainer talks and frames a session>

## Exercise biases
- <moves / styles this trainer leans toward>
- <anything this trainer tends to avoid>

## Notes
<anything else worth remembering, e.g. catchphrases>
```

Get the current date from the shell with `date +%F` for the `_Last updated_`
field (don't trust a remembered date).

## Principles

- A trainer flavors **style and selection**, never safety. Profile limits and
  injuries always override the persona.
- Keep personas fun but the workouts real and doable in a working-day break.
- One active trainer at a time — this file is the single source of truth.
