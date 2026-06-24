---
name: test-cypress
description: Use this skill when writing or reviewing Cypress tests to apply Cypress-specific best practices for reliable, idiomatic test code.
metadata:
  author: Greg Duckworth
  version: 1.0.0
  tags:
    - domain/engineering
  compatibility:
    - all
  frameworks:
    - cypress
---

Use this skill when working with Cypress tests.

## Requires

- test-automation-guidelines

---

## Cypress Rules

### Command Model

- Use Cypress command chaining
- Do not use async/await
- Avoid excessive use of `cy.then()`

---

### Selectors

Prefer stable, intentional selectors (attributes or accessibility hooks) as your primary choice. Text-based selectors are brittle — they can break with copy changes, translations, or small wording edits — so treat them as a fallback only when no stable attributes exist.

**Primary (preferred):**

```javascript
cy.get('[data-testid="..."]');
cy.get("#elementId");
```

**Fallback (use only if no stable attributes are available):**

```javascript
cy.contains("Submit"); // text selector - brittle
```

---

### Waiting

Do not use:

```javascript
cy.wait(1000);
```

Use:

- `cy.wait('@alias')`
- `.should(...)`
- DOM assertions

---

### Network Control

- Use `cy.intercept()` for spying and stubbing
- Prefer stubbing for deterministic tests
- Always use aliases for intercepted requests

```javascript
cy.intercept("GET", "/api/...", { fixture: "user.json" }).as("getUsers");
cy.wait("@getUsers");
```

---

### Authentication

- Prefer programmatic login
- Use `cy.session()`
- Avoid UI login in every test

```javascript
cy.session("user", () => {
  cy.login();
});
```

---

### State Management

- Use `Cypress.env` for configuration and static data
- Do not use `Cypress.env` to share mutable state between tests
- Reset cookies/localStorage when required

---

### Assertions

```javascript
cy.get('[data-testid="cart-count"]').should("have.text", "1");
```

---

### Custom Commands & Fixtures

- Use custom commands sparingly for repetitive actions
- Use `cy.fixture()` to manage large data sets

---

### API Waiting Pattern

```javascript
cy.intercept("POST", "/checkout").as("checkout");
cy.wait("@checkout");
```

---

### Anti-Patterns

- Mixing async/await with Cypress
- Overusing `cy.then()`
- UI login in every test
