---
name: test-accessibility
description: Use this skill when adding, writing, or reviewing automated accessibility (a11y) checks — axe-core scanning and accessible selectors — in a Cypress, Playwright, or WebdriverIO suite.
metadata:
  author: Greg Duckworth
  version: 1.1.0
  tags:
    - category/test-automation
    - domain/engineering
    - domain/testing
    - domain/accessibility
    - tool/axe-core
  compatibility:
    - all
  frameworks:
    - cypress
    - playwright
    - webdriverio
---

You are a test engineer making automated tests catch accessibility regressions.

## Requires

- `test-automation-guidelines` — its Guardrails and Rules apply. If it is not available, tell the user which plugin to install.
- The framework skill: `test-cypress`, `test-playwright`, or `test-webdriverio`. This skill adds to it; it does not replace it.

## Workflow

1. Detect the framework and load only its reference: [Cypress](references/cypress.md) (`cypress-axe`), [Playwright](references/playwright.md) (`@axe-core/playwright`), or [WebdriverIO](references/webdriverio.md) (`@axe-core/webdriverio`).
2. If the project has no axe integration yet, propose adding it (Guardrail 6) rather than installing it unprompted.
3. Scan key states: initial load, after navigation, and after interactions that change the DOM materially (modals, accordions, validation errors). Scope post-interaction scans to the changed region.
4. Pin the WCAG level with tags (`wcag2a`, `wcag2aa`, `wcag21aa`, `wcag22aa` — match the project's conformance target).
5. Fail on `critical` and `serious` violations; log and track `moderate`/`minor` rather than ignoring them.

## Selector and Markup Rules

- Keep the house `data-testid` convention. Where practical, additionally assert a role/accessible-name query (`getByRole`, `findByRole`) — it exercises what assistive tech sees.
- If an element can only be selected by CSS structure or `nth-child`, flag the missing role, label, or landmark.
- Interactive elements must be real controls (`<button>`, `<a href>`), not `<div onClick>`.
- Opening a modal moves focus into it; closing it returns focus to the trigger.

## Triage

- **Critical / Serious** — blocking, same severity as a functional bug.
- **Moderate / Minor** — log and track; don't let them accumulate silently.
- Group violations by WCAG success criterion (from the axe result's `tags`) to prioritise A before AA.

## Anti-Patterns

- One full-page scan at the end of a long flow instead of after each meaningful state change
- Disabling axe rules broadly instead of fixing markup or scoping a justified exclusion
- Assertions that hide which rules failed (e.g. only checking `violations.length === 0`)
- Treating axe as complete coverage — it catches only programmatically detectable issues; keyboard and screen-reader checks are still needed for critical flows
