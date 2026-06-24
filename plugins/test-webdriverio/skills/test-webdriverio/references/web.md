# Web Tests Guidance

Apply this guidance when writing, reviewing or maintaining WebdriverIO browser-based web tests.

## WebdriverIO Rules

### Async Model

- Use async/await

---

### Selectors

- Prefer stable, intentional selectors (attributes or accessibility hooks) as your primary choice. 
- Text-based selectors that match visible text are brittle — they can break with copy changes, translations, or small wording edits — so treat them as a fallback only when no stable attributes exist.

**Primary (preferred):**

```javascript
$('[data-testid="add-to-basket"]');
$('[role="button"][aria-label="Submit"]');
```

**Fallback (use only if no stable attributes are available):**

```javascript
$("button=Submit"); // text selector - brittle
```

---

### Waiting

Do not use:

```javascript
browser.pause(1000);
```

- Use `waitForDisplayed`, `waitForEnabled`, `waitForClickable` to wait on element state, or `browser.waitUntil` for custom conditions

```javascript
await $("#element").waitForDisplayed();
```

---

### Assertions

- Use `expect-webdriverio` matchers for test assertions

```javascript
await expect($('[data-testid="cart-count"]')).toHaveText("1");
await expect($('[data-testid="cart-count"]')).toBeDisplayed();
```

---

### Page Objects

- Use the Page Object Model for structure and reuse

**Example:**

```javascript
class ProductPage {
  // Prefer stable attributes or accessibility hooks
  get addButton() {
    return $('[data-testid="add-to-basket"]');
  }

  async addToBasket() {
    await this.addButton.click();
  }
}
```

---

### Network Control

- Use CDP (Chrome DevTools Protocol) for debugging and network control in Chromium-based environments
- Prefer mock services where available
