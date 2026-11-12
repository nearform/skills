---
name: test-playwright
description: Use this skill when writing, fixing, or reviewing Playwright tests — locators, web-first assertions, page.route network control, storageState auth, timeouts, parallelism, and webServer config.
metadata:
  author: Greg Duckworth
  version: 1.2.0
  tags:
    - category/test-automation
    - domain/engineering
    - domain/testing
    - tool/playwright
  compatibility:
    - all
  frameworks:
    - playwright
---

Playwright-specific rules. Base principles are not restated here.

## Requires

- `test-automation-guidelines` — its Guardrails and Rules apply. If it is not available, tell the user which plugin to install.
- For UI suites, also `test-accessibility` (`@axe-core/playwright`).

## Rules

### Async model

- `await` every Playwright call and assertion.

### Selectors

- Primary: `page.getByTestId("…")` or `page.getByRole("button", { name: "Submit" })`.
- `page.getByText(...)` is a fallback only — copy changes break it. Avoid CSS structure and `nth()` positions.

### Waiting and timeouts

- Never `page.waitForTimeout(ms)`. Rely on locator auto-waiting and web-first assertions: `await expect(locator).toBeVisible()`.
- `expect.timeout` (assertions, `expect.poll`) and the test `timeout` (whole test, including fixtures) are separate. Give waits on server work an explicit `timeout`, and keep chained waits within the test timeout. When polling, stop on every final state — see [references/config.md](references/config.md).

### Network and test doubles

- Stub or fulfil with `page.route`; prefer fixtures for bodies.

```javascript
await page.route("**/api/products", (route) =>
  route.fulfill({ path: "fixtures/products.json" }),
);
```

### Auth and state

- Log in once and reuse `storageState`; don't drive the login UI per test. Tests must be parallel-safe — no global shared state.

### Config and CI

- `expect(page).toHaveScreenshot()` for visual regression, with stable data.
- Traces and video in CI; `workers` within any shared server's concurrency limit; don't overwrite `NODE_OPTIONS` in `webServer.env` — details in [references/config.md](references/config.md).

## Anti-Patterns

- `waitForTimeout` usage
- Global shared state between tests
- Overuse of `page.evaluate` instead of locators
- Overwriting `NODE_OPTIONS` (or similar variables) in `webServer.env`
- Chained waits whose timeouts add up to more than the test timeout

## References

- [references/config.md](references/config.md) — timeouts, polling, parallelism, `webServer`, traces. Load when editing config or diagnosing timeouts.
