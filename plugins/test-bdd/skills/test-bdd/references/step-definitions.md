# Step Definition Glue by Framework

Load when writing or reviewing step definitions. The step **text** is identical across frameworks; only the body changes, and the body obeys the framework skill.

Feature under test:

```gherkin
Scenario: Registered customer signs in
  Given Ada is a registered customer
  When she signs in with valid credentials
  Then she lands on her dashboard
```

## Cypress (`@badeball/cypress-cucumber-preprocessor`) — obeys `test-cypress`

Command chaining, `cy.session` for programmatic login, per-scenario state on the Cucumber World (`this`, so use `function`, not arrows).

```javascript
import { When, Then } from "@badeball/cypress-cucumber-preprocessor";

When("she signs in with valid credentials", function () {
  cy.session("ada", () => cy.login(this.user));
});

Then("she lands on her dashboard", () => {
  cy.get('[data-testid="dashboard"]').should("be.visible");
});
```

## Playwright (`playwright-bdd`) — obeys `test-playwright`

Playwright-style steps from `createBdd()`: arrow functions that receive fixtures as the first argument and do **not** use `this`. Share per-scenario state through a custom fixture, not the World or a module global.

```javascript
import { expect } from "@playwright/test";
import { createBdd } from "playwright-bdd";

const { When, Then } = createBdd();

When("she signs in with valid credentials", async ({ page }) => {
  await loginViaApi(page); // programmatic login, e.g. via storageState
});

Then("she lands on her dashboard", async ({ page }) => {
  await expect(page.getByTestId("dashboard")).toBeVisible();
});
```

`playwright-bdd/decorators` is a separate API for class-based page objects (`@When(...)` on methods); don't import plain step functions from it.

## WebdriverIO (`@wdio/cucumber-framework`) — obeys `test-webdriverio`

```javascript
import { When, Then } from "@wdio/cucumber-framework";

When("she signs in with valid credentials", async function () {
  await loginViaApi(this.user);
});

Then("she lands on her dashboard", async () => {
  await expect($('[data-testid="dashboard"]')).toBeDisplayed();
});
```

## What this layer does not dictate

Whether to chain or await, how to wait, how to stub the network — those belong to the framework skill.
