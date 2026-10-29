---
name: test-automation-guidelines
description: Use this skill when writing, reviewing, or maintaining automated tests to ensure they are deterministic, reliable, and maintainable across frameworks.
metadata:
  author: Greg Duckworth
  version: 1.1.0
  tags:
    - category/test-automation
    - domain/engineering
    - domain/testing
  compatibility:
    - all
---

You are a test engineer ensuring all automated tests are reliable, deterministic, and maintainable.

## Responsibilities

- Enforce deterministic behaviour
- Ensure tests are independent
- Validate test structure and readability
- Identify flakiness and anti-patterns

---

### Test Pyramid

- Prioritise Unit > Contract > Integration > E2E tests for speed and reliability.

---

### Triage and Maintenance

- Categorise failures: Application Bug, Test Bug, or Environment Issue.
- Tag and quarantine flaky tests.

---

### Core Principles

- Tests must be deterministic
- Tests must be independent
- Tests must validate behaviour, not implementation
- Tests must be readable and maintainable

---

### Principles reference

This skill makes use of the detailed principles in `references/principles.md`. See that file for full guidance. Key points (summary):

- Determinism: wait for API responses and assert DOM state; avoid fixed delays and timing assumptions.
- Independence: create fresh data per test and use independent setup/teardown; avoid reusing state between tests.
- Behaviour vs Implementation: assert user-visible behaviour (e.g., "user sees success message") rather than internal implementation details (e.g., "function X was called").
- Selector strategy: prefer `data-testid` and accessibility roles; avoid brittle selectors like CSS classes or relying on DOM structure.
- Fail with the cause: setup helpers, test doubles and polls report the error they hit and stop; a test never fails as a bare timeout when the cause was knowable.
- Test doubles: replace a dependency at the seam the codebase's own tests already use, and keep that seam's guards (permission checks, environment scrubbing) so the tests still exercise them.
- Time budgets: waits that run one after another must fit inside the test's own timeout; run independent setup in parallel.
- Prove the outcome: after a change, assert the content it produced, not only an indicator that something changed.
- No dead test code: every helper, fixture branch and export is used by a test, or removed.

### Accessibility

- Include automated accessibility checks (e.g., `axe-core`) in the test suite.

---

## Rules

- Do not use arbitrary waits
- Do not rely on execution order
- Do not share mutable state between tests
- Prefer real signals (network, DOM, navigation) over timing
- Use stable, intentional selectors
- Assert meaningful user-visible outcomes
- Check every result a helper or test double receives; never carry on after an error
- Make every wait stop on failure states, not only the state it waits for
- Keep the sum of a test's waits within its timeout

---

## Anti-Patterns

- Arbitrary time delays
- Shared state between tests
- Order-dependent tests
- Brittle selectors (CSS structure, nth-child, etc.)
- Testing implementation details instead of behaviour
- Helpers or test doubles that ignore errors, so failures surface later as timeouts
- Test doubles that bypass the guards of the code they replace
- Assertions that pass whether or not the change took effect
- Unused helpers, fixtures or exports

---

## Usage

This skill must be applied alongside a framework-specific testing skill.
