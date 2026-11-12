---
name: test-webdriverio
description: Use this skill when writing, fixing, or reviewing WebdriverIO tests — browser e2e, or iOS/Android mobile via Appium — including selectors, waiting, page objects, and wdio.conf.
metadata:
  author: Greg Duckworth, Sherrylene Gauci
  version: 1.1.0
  tags:
    - category/test-automation
    - domain/engineering
    - domain/testing
    - tool/webdriverio
    - tool/appium
  compatibility:
    - all
  frameworks:
    - webdriverio
    - appium
---

WebdriverIO-specific rules for web and mobile tests. Base principles are not restated here.

## Requires

- `test-automation-guidelines` — its Guardrails and Rules apply. If it is not available, tell the user which plugin to install.
- For web UI suites, also `test-accessibility` (`@axe-core/webdriverio`).

## Rules

### Async model

- `await` every `$`, `$$`, `browser`, and `expect` call.

### Selectors

- Web: `$('[data-testid="…"]')` or a role/label attribute selector. Mobile: accessibility id (`$("~login-button")`).
- Avoid XPath and DOM-structure selectors; text selectors (`$("button=Submit")`) are a fallback only.

### Waiting and timeouts

- Never `browser.pause(ms)`. Use `waitForDisplayed`/`waitForClickable` or WebdriverIO's auto-retrying `expect(...)` matchers.
- Give waits on server work an explicit `timeout` rather than raising `waitforTimeout` globally; chained waits must fit within the framework's test timeout (`mochaOpts.timeout` or equivalent).
- Use `browser.waitUntil` with a `timeoutMsg` that states the condition, and stop on failure states too.

### Structure

- Page objects expose elements as getters and actions as methods; query an element once per action, not repeatedly.

## Anti-Patterns

- `browser.pause` usage
- XPath or other brittle selectors
- Excessive or repeated element queries
- Tight coupling between tests and implementation details

## References

Load only the one you need:

- [references/web.md](references/web.md) — browser selectors, waiting, assertions, page objects.
- [references/mobile.md](references/mobile.md) — Appium selectors (iOS predicate/class chain, Android UiAutomator), gestures, mobile page objects.
