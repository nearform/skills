---
name: test-cypress
description: Use this skill when writing, fixing, or reviewing Cypress e2e or component tests — command chaining, selectors, cy.intercept network control, cy.session auth, and waiting without cy.wait(ms).
metadata:
  author: Greg Duckworth
  version: 1.1.0
  tags:
    - category/test-automation
    - domain/engineering
    - domain/testing
    - tool/cypress
  compatibility:
    - all
  frameworks:
    - cypress
---

Cypress-specific rules. Base principles are not restated here.

## Requires

- `test-automation-guidelines` — its Guardrails and Rules apply. If it is not available, tell the user which plugin to install.
- For UI suites, also `test-accessibility` (`cypress-axe`).

## Rules

### Command model

- Chain Cypress commands; do not use `async`/`await`. Keep `cy.then()` for the rare case that needs a plain value.

### Selectors

- Primary: `cy.get('[data-testid="…"]')`, or a role/label query (`cy.findByRole` via `@testing-library/cypress`).
- Avoid CSS classes, `nth-child`, and DOM structure. `cy.contains("Submit")` is a fallback only — copy changes break it.

### Waiting and timeouts

- Never `cy.wait(1000)`. Wait on `cy.wait("@alias")`, `.should(...)` retries, or DOM state.
- Set `{ timeout }` on the specific command that waits for server work instead of raising `defaultCommandTimeout` globally. Chained alias waits must fit within the spec's budget (`requestTimeout` + `responseTimeout` per alias).
- When waiting on an alias, assert its status so a failed request fails with the cause: `cy.wait("@save").its("response.statusCode").should("eq", 200)`.

### Network and test doubles

- `cy.intercept()` for spying and stubbing; always alias it. Prefer stubs for determinism; use `{ fixture: "user.json" }` for bodies.

```javascript
cy.intercept("POST", "/api/checkout").as("checkout");
cy.get('[data-testid="pay"]').click();
cy.wait("@checkout").its("response.statusCode").should("eq", 201);
```

### Auth and state

- Log in programmatically and cache with `cy.session()`; don't drive the login UI in every test.
- `Cypress.env` is for configuration, never mutable cross-test state. Reset cookies/localStorage where needed.

### Config and CI

- Use custom commands sparingly, for genuinely repeated actions. `cy.fixture()` for large data sets.

## Anti-Patterns

- Mixing `async`/`await` with Cypress commands, or overusing `cy.then()`
- `cy.wait(<ms>)`
- Intercepts without aliases, or alias waits that never check the response
- UI login in every test
- Raising `defaultCommandTimeout` globally to hide a slow wait
