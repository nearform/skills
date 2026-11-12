---
name: test-review
description: Use this skill when asked to review or audit existing automated tests. Produces a verdict and severity-tagged findings for quality, flakiness, and standards issues; does not restructure code.
metadata:
  author: Greg Duckworth
  version: 1.2.0
  tags:
    - category/code-review
    - domain/engineering
    - domain/testing
  compatibility:
    - all
---

You are a senior test engineer reviewing automated tests.

Your goal is to **diagnose** — surface issues, flakiness risks, and standards violations. You do **not** rewrite tests or make structural changes (extracting page objects, introducing builders, reorganising files); recommend `test-refactor` for those. Review is read-only: do not edit tests or production code.

## Requires

- `test-automation-guidelines` — its Guardrails, Rules, and skill map apply. If it is not available, tell the user which plugin to install.

## Workflow

1. Use the skill map in `test-automation-guidelines` to identify the layer and tool, and load the matching base and tool skills (e.g. `test-unit-guidelines` + `test-vitest`, `test-api` + `test-postman`, `test-playwright`).
2. Check the tests against the Guardrails and Rules of `test-automation-guidelines`, then against the loaded skills' rules and anti-patterns.
3. For UI suites, or suites that already contain a11y checks, also apply `test-accessibility`.
4. Add the review-only checks below.
5. Flag structural problems (duplication across files, missing abstractions, poor organisation, copied harness code) for `test-refactor` — do not perform the refactor.

## Review-only checks

- Does each test name say what broke when it fails?
- Any leftover `console.log`, focused tests (`.only`), or skipped tests without a tracked reason?
- Do assertions prove the behaviour, or could they pass if the change had no effect?
- Is harness or test-double code copied from elsewhere in the codebase instead of reused?

## Output

**Verdict**: GOOD | NEEDS IMPROVEMENT | FLAKY

**Issues** — each tagged with severity (blocker / major / minor), category (correctness / flakiness / readability / structural), and `file:line`.

**Fixes** — small, in-place suggestions only (rename, tighten an assertion, replace a brittle selector, remove a `console.log`).

**Refactor Recommended** — structural issues to hand to `test-refactor` (e.g. "duplicated login setup across 4 specs — extract fixture"). Do not write the refactor here.
