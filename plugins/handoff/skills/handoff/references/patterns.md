# Handoff patterns

## Side task without polluting the main session

During a focused session (feature work, grilling, planning), you notice out-of-scope work (refactor, issue filing, separate API split).

1. State the handoff purpose explicitly (reason + what the next session does).
2. Run handoff — this also **sharpens** the current session by marking that work out of scope.
3. Open a new session; paste or attach the handoff file path.
4. Continue the original session unchanged.

## Prototype during planning (round-trip)

When questions need code or UI to answer (unknown unknowns):

1. **Parent session** (e.g. grill/plan): hand off with focus on prototyping specific risky areas.
2. **Prototype session**: may run large; build spike; learn.
3. **Prototype session** → handoff **back** to parent: learnings not obvious from the code alone, decisions made, dead ends.
4. **Parent session**: resume planning/PRDs/issues with prototype + return handoff as pointers.

This mimics a sub-agent: one context window per task, compressed learnings in between.

## Cross-agent handoff

Because the artifact is plain markdown in a temp path:

- Session A can be any harness (Claude Code, Cursor, Codex, etc.).
- Session B can be a different tool — paste the file or its contents.
- Useful for adversarial review or second-opinion implementation.

## Context budget reminder

Very long sessions degrade quality before the model's max context is exhausted. Handoff early when spinning up **different** work; use compact when staying on the **same** thread.
