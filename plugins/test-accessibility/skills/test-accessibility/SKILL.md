---
name: test-accessibility
description: Use this skill when writing, reviewing, or maintaining accessibility (a11y) checks in automated tests, or when adding automated accessibility scanning (axe-core) to an existing Cypress, Playwright, or WebdriverIO suite.
metadata:
  author: Greg Duckworth
  version: 1.0.1
  tags:
    - category/test-automation
    - domain/engineering
    - domain/testing
    - domain/accessibility
    - tool/axe-core
  compatibility:
    - all
---

You are a test engineer ensuring automated tests catch accessibility regressions and encourage accessible selector strategy.

## Requires

- test-automation-guidelines

## Scope

- Apply alongside a framework-specific testing skill (`test-cypress`, `test-playwright`, `test-webdriverio`) — this skill does not replace them.
- Covers two complementary areas:
  1. **Automated scanning** — running `axe-core` against pages/components to catch WCAG violations.
  2. **Accessible-first selectors** — using accessibility roles and labels as the primary test selector strategy, which doubles as a manual accessibility signal.

---

## Workflow

1. Detect framework in the file under test/review:
   - `cy.` → use the Cypress section below (`cypress-axe`)
   - `@playwright/test` → use the Playwright section below (`@axe-core/playwright`)
   - `browser.` / `@wdio/globals` → use the WebdriverIO section below (`@axe-core/webdriverio`)
2. Ensure automated scans run on key states: initial page load, after navigation, and after interactions that change the DOM materially (modals, accordions, forms with validation errors).
3. Fail builds on `critical` and `serious` violations by default; track `moderate`/`minor` separately rather than silently ignoring them.
4. Additionally assert accessible names/roles where practical (see `plugins/test-automation-guidelines/skills/test-automation-guidelines/references/principles.md`) — this forces pages to expose proper accessible names, which is itself an accessibility check.

---

### Cypress (`cypress-axe`)

```javascript
import "cypress-axe";

beforeEach(() => {
  cy.visit("/checkout");
  cy.injectAxe();
});

it("has no detectable accessibility violations on load", () => {
  cy.checkA11y(null, {
    includedImpacts: ["critical", "serious"],
  });
});

it("has no violations after opening the modal", () => {
  cy.get('[data-testid="open-modal"]').click();
  cy.checkA11y('[role="dialog"]', {
    includedImpacts: ["critical", "serious"],
  });
});
```

- Call `cy.injectAxe()` after each `cy.visit()` / full-page reload — the injected script does not survive navigation.
- Scope `cy.checkA11y(selector)` to the changed region after an interaction instead of re-scanning the whole page.

---

### Playwright (`@axe-core/playwright`)

```javascript
import { test, expect } from "@playwright/test";
import AxeBuilder from "@axe-core/playwright";

test("checkout page has no critical/serious a11y violations", async ({
  page,
}) => {
  await page.goto("/checkout");

  const results = await new AxeBuilder({ page })
    .withTags(["wcag2a", "wcag2aa"])
    .analyze();

  const blocking = results.violations.filter((v) =>
    ["critical", "serious"].includes(v.impact),
  );
  expect(blocking).toEqual([]);
});
```

- Use `.include(selector)` / `.exclude(selector)` to scope a scan to a changed region instead of the full page.
- Use `.withTags([...])` to pin the WCAG level under test (e.g. `wcag2a`, `wcag2aa`) rather than scanning every rule unconditionally.
- Filter `results.violations` by `impact` before asserting (per the default in step 3) — asserting the raw array is empty fails the build on `moderate`/`minor` issues too. Log the filtered-out violations instead of discarding them.
- Snapshot the failing violations in assertion failures — they carry the rule id, impact, and offending selectors needed for triage.

---

### WebdriverIO (`@axe-core/webdriverio`)

```javascript
import AxeBuilder from "@axe-core/webdriverio";

it("has no accessibility violations", async () => {
  await browser.url("/checkout");

  const results = await new AxeBuilder({ client: browser })
    .withTags(["wcag2a", "wcag2aa"])
    .analyze();

  const blocking = results.violations.filter((v) =>
    ["critical", "serious"].includes(v.impact),
  );
  expect(blocking).toEqual([]);
});
```

- Same scoping/tagging/filtering approach as Playwright — `include`/`exclude`, `withTags`, and filtering by `impact` keep scans intentional and consistent with the step 3 default.

---

## Selector & Markup Rules

- `test-cypress` and `test-playwright` prefer `data-testid`/`getByTestId` as the house selector convention — keep using it. Where practical, _additionally_ assert an accessible-name/role selector alongside it (`page.getByRole("button", { name: "Submit" })`, `cy.findByRole(...)` via `@testing-library/cypress`) rather than replacing the test id — this exercises the same accessible name the tool exposes to assistive tech without abandoning the house convention.
- If a test can only select an element via CSS structure or `nth-child`, treat that as a signal the markup is missing a semantic role, label, or landmark — flag it rather than only reaching for a test id.
- Verify interactive elements are real controls (`<button>`, `<a href>`) rather than `<div onClick>` — this affects both selector stability and keyboard/screen-reader access.
- Verify focus management for dynamic UI: opening a modal should move focus into it; closing it should return focus to the trigger.

---

## Triage

- **Critical / Serious** — treat as blocking, same severity as a functional bug.
- **Moderate / Minor** — log and track; do not let them silently accumulate unaddressed.
- Categorise each violation by WCAG success criterion (from the axe result's `tags`) so fixes can be prioritised by conformance level (A vs AA).

---

## Anti-Patterns

- Running a full-page scan only once at the end of a long flow instead of after each meaningful state change.
- Suppressing or disabling axe rules broadly (e.g. disabling a whole rule set) instead of fixing the underlying markup or scoping the exclusion narrowly with justification.
- Asserting only `results.violations.length === 0` without surfacing which rules failed, making failures hard to triage.
- Relying solely on automated scanning — axe-core catches a subset of WCAG issues (roughly programmatically detectable ones); it does not replace keyboard-navigation and screen-reader checks for critical flows.
