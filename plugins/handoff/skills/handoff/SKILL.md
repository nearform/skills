---
name: handoff
description: >-
  Compact the current conversation into a handoff markdown document for another
  agent or session to pick up. Use when handing off a side task, spawning a
  prototype session, passing work to a different agent (Cursor, Codex, GitHub Copilot CLI, Claude Code),
  or keeping the current session focused while continuing work elsewhere. Also use
  when the user says handoff, hand off, or wants a handoff document.
argument-hint: "What will the next session be used for?"
metadata:
  author: Rosario Terranova
  version: 1.0.0
  tags:
    - category/productivity
    - domain/engineering
---

# Handoff

Compress the current session into a **disposable** markdown file so a fresh agent can continue without inheriting a bloated context window.

## When to use handoff vs compact

| Situation                                                                | Prefer                                                                           |
| ------------------------------------------------------------------------ | -------------------------------------------------------------------------------- |
| Same session, same goal — context is huge but work continues in place    | Harness **compact** (summarize in-place)                                         |
| Side task, refactor, bug, or prototype **out of scope** for this session | **Handoff** to a new session                                                     |
| Different agent or tool for the next step                                | **Handoff** (markdown is portable)                                               |
| Prototype or spike, then return learnings to the planner session         | **Handoff** out and **handoff** back (see [patterns.md](references/patterns.md)) |

Handoff keeps the **current** session pure; compact keeps one **long** session going.

## Before writing

The user (or skill arguments) should state **why** you are handing off and **what the next session will do**. Without that focus, the document will be vague. If missing, ask briefly or infer from the latest user message.

## Instructions

1. Write a handoff document summarizing only what the **next** session needs.
2. **Save outside the workspace** — use the OS temporary directory (e.g. `mktemp` / `$TMPDIR`), **not** the project repo. Handoffs are disposable, not documentation.
3. Tell the user the full path to the file when done.
4. Include a **Suggested skills** section listing skills the next agent should invoke (by name), with one line each on why.
5. **Do not duplicate** content already in PRDs, plans, ADRs, issues, commits, or diffs — **reference by path or URL**.
6. **Redact** API keys, passwords, tokens, and PII.
7. If the user passed arguments (or a slash-command argument), treat them as the next session's focus and tailor the document accordingly.

## Document template

Use this structure (omit empty sections):

```markdown
# Handoff: [short title]

## Purpose

Why we are handing off and what the next session must accomplish.

## Context

Decisions, constraints, and background the next agent needs (compressed).

## Current state

What is done, in progress, or blocked in this session.

## Artifacts

Pointers only — paths, URLs, issue numbers, branch names.

## Open questions

Unresolved items for the next session to answer or validate.

## Suggested skills

- `skill-name` — why to invoke it

## Out of scope

What this handoff explicitly does **not** include (keeps the next session focused).
```

## Quality bar

- **Slice, don't dump** — only context relevant to the stated next-session purpose.
- **Actionable** — the next agent should know the first concrete step.
- **Portable** — readable without this chat; no "as discussed above."
- **Short** — prefer pointers over pasted prose.

## Common patterns

For parallel side tasks and cross-agent review, see [references/patterns.md](references/patterns.md).
