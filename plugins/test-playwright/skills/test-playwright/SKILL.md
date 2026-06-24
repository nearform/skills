---
name: test-playwright
description: Use this skill when working with Playwright tests and applying best practices for reliable, parallel-safe test automation.
metadata:
  author: Greg Duckworth
  version: 1.0.0
  tags:
    - domain/engineering
  compatibility:
    - all
  frameworks:
    - playwright
---

Use this skill when working with Playwright tests.

## Requires

- test-automation-guidelines

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

---

### Anti-Patterns

- `waitForTimeout` usage
- Global shared state
- Overuse of `page.evaluate`
