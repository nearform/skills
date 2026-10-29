---
name: test-playwright
description: Use this skill when working with Playwright tests and applying best practices for reliable, parallel-safe test automation.
metadata:
  author: Greg Duckworth
  version: 1.1.0
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

Use this skill when working with Playwright tests.

## Requires

- test-automation-guidelines

## See Also

- `test-accessibility` — apply when the suite should include automated a11y scanning (`@axe-core/playwright`) or when reviewing/writing accessible selector strategy.

---

## Playwright Rules

### Async Model

- Use async/await consistently

---

### Selectors

Prefer stable, intentional selectors (attributes or accessibility hooks) as your primary choice. Text-based selectors are brittle — they can break with copy changes, translations, or small wording edits — so treat them as a fallback only when no stable attributes exist.

**Primary (preferred):**

```javascript
page.getByTestId("...");
page.getByRole("button", { name: "Submit" });
```

**Fallback (use only if no stable attributes are available):**

```javascript
page.getByText("Submit");
```

---

### Waiting

Do not use:

```javascript
await page.waitForTimeout(1000);
```

Use:

- locator auto-waiting
- expect(...)

---

### Assertions

```javascript
await expect(page.getByTestId("...")).toBeVisible();
```

---

### Authentication

- Use storageState
- Avoid UI login per test

```javascript
test.use({ storageState: "storageState.json" });
```

---

### Network Control

```javascript
await page.route('/api/...', route => route.fulfill(...));
```

---

### Mocking

```javascript
await page.route("/api/products", (route) =>
  route.fulfill({ path: "fixtures/products.json" }),
);
```

---

### Visual Testing

- Use `expect(page).toHaveScreenshot()` for UI regression testing

---

### Debugging

- Use traces and video recordings in CI for easy triage

---

### Parallelism

- Tests must be parallel-safe
- Avoid shared state across tests
- When tests share a server with a concurrency limit, set `workers` no higher than that limit

---

### Timeouts

- Web-first assertions and `expect.poll` give up after `expect.timeout` (5 seconds by default). Pass a `timeout` that covers waits on server work, or raise `expect.timeout` in the config.
- The test `timeout` (30 seconds by default) covers the whole test, including setup in fixtures. Waits that run one after another must fit inside it together.

```javascript
test("order becomes ready", async () => {
  test.setTimeout(90_000);
  await expect
    .poll(() => getStatus(id), { timeout: 60_000 })
    .toMatch(/^(ready|failed)$/);
  expect(await getStatus(id)).toBe("ready");
});
```

---

### Test Servers (`webServer`)

- `webServer.env` is laid over `process.env` and replaces any variable it names. Add to `NODE_OPTIONS` instead of overwriting it:

```javascript
env: {
  NODE_OPTIONS: `${process.env.NODE_OPTIONS ?? ""} --conditions=source`.trim(),
},
```

- Take ports from environment variables with defaults, so a second copy of the suite or a developer's own server doesn't collide.
- Playwright stops `webServer` processes with SIGKILL unless `gracefulShutdown` is set. Set it (e.g. `{ signal: "SIGTERM", timeout: 5000 }`) when the server cleans up on a signal.

---

### Anti-Patterns

- `waitForTimeout` usage
- Global shared state
- Overuse of `page.evaluate`
- Overwriting `NODE_OPTIONS` (or similar variables) in `webServer.env`
- Chained waits whose timeouts add up to more than the test timeout
