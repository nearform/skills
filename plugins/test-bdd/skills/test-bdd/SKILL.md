---
name: test-bdd
description: Use this skill when writing or reviewing BDD/Cucumber feature files and step definitions. It keeps Gherkin declarative and behaviour-focused, keeps step definitions thin, and delegates all framework mechanics (selectors, waiting, network, assertions) to the matching framework skill.
metadata:
  author: Greg Duckworth
  version: 1.0.0
  tags:
    - domain/engineering
    - domain/testing
    - methodology/bdd
  compatibility:
    - all
  frameworks:
    - cucumber
    - cypress
    - playwright
    - webdriverio
---

Use this skill when writing, reviewing, or maintaining Behaviour-Driven Development (BDD) tests — Gherkin feature files and their step definitions.

This is a **methodology** skill. It owns the **specification layer** (how scenarios are written) and the **glue layer** (how step definitions are organised). It does **not** own framework mechanics. Selectors, waiting, network control, and assertions are delegated to whichever framework skill matches the project.

## Requires

- test-automation-guidelines

---

## Composes With (framework delegation)

BDD sits **above** the framework. Detect the framework from the step-definition code and apply its skill for everything that happens **inside** a step:

- If steps use `cy.` → use **test-cypress**
- If steps use `@playwright/test`, `page.`, or `playwright-bdd` → use **test-playwright**
- If steps use `browser.` or `@wdio/globals` → use **test-webdriverio**

```
test-automation-guidelines   (determinism, independence, stable selectors)
          ▲ requires
      test-bdd                (Gherkin quality + step-definition structure)
          ▲ delegates in-step mechanics to
test-cypress / test-playwright / test-webdriverio   (cy.* / page.* / browser.*)
```

**Rule of thumb:** BDD owns the `.feature` file and the _shape_ of each step. The framework skill owns the _code inside_ each step. Never restate framework rules here — defer to the framework skill so there is a single source of truth.

---

## Gherkin Rules

### Write declarative scenarios, not imperative scripts

Describe **what** the user is trying to achieve and **why**, never the mechanical **how**. Imperative steps leak UI detail into the specification, break on redesigns, and are unreadable to non-technical stakeholders — which defeats the entire purpose of BDD.

**Bad (imperative — UI mechanics in the feature file):**

```gherkin
Scenario: Login
  Given I open "/login"
  When I type "ada@example.com" into "#email"
  And I type "hunter2" into "#password"
  And I click "#submit"
  Then the URL should be "/dashboard"
```

**Good (declarative — behaviour and intent):**

```gherkin
Scenario: Registered customer signs in
  Given Ada is a registered customer
  When she signs in with valid credentials
  Then she lands on her dashboard
```

The selectors, typing, and URL assertions still exist — they live in the **step definitions**, governed by the framework skill, not in the feature file.

---

### One behaviour per scenario

A scenario asserts a single, named behaviour. If the name needs "and" to be accurate, split it. Multiple unrelated assertions in one scenario make failures ambiguous and violate the independence principle from `test-automation-guidelines`.

---

### Use `Background` for shared context only

`Background` is for **Given** steps that establish common context every scenario needs. It must not contain `When`/`Then`, actions, or assertions. If a `Background` grows beyond a few lines, the scenarios are probably coupled — prefer explicit setup or a tag-scoped hook.

```gherkin
Background:
  Given the store has the "Standard" catalogue
  And Ada is signed in
```

---

### Use `Scenario Outline` for one behaviour with varying data

Reach for `Scenario Outline` + `Examples` only when the **behaviour is identical** and only the **data** changes. Do not use it to bolt together different behaviours.

```gherkin
Scenario Outline: Discounts apply at qualifying basket totals
  Given Ada has a basket worth <total>
  When she views the basket
  Then the discount shown is <discount>

  Examples:
    | total | discount |
    | 50    | 0%       |
    | 100   | 5%       |
    | 250   | 10%      |
```

---

### Avoid conjunction steps

A step that contains "and" is doing two things and cannot be reused cleanly. Split `Given the user is logged in and has an empty basket` into two `Given` steps.

---

### Keep step phrasing reusable and unambiguous

- Use business vocabulary (ubiquitous language), not UI labels.
- Prefer the third person and a consistent tense across the suite.
- Parameterise nouns (`"<role>"`, `<amount>`) so one step definition serves many scenarios.
- Two steps that read differently must not do the same thing; two steps that read the same must not do different things.

---

### Tag strategy

Tags are for **selection and lifecycle**, not narration. Keep a small, documented vocabulary, e.g. `@smoke`, `@regression`, `@wip`, `@flaky`. Wire suite-wide concerns (auth, seeding) to **tagged hooks** rather than repeating setup in every scenario. Never ship `@wip`/`@flaky` to a blocking pipeline — quarantine, per `test-automation-guidelines`.

---

## Step Definition Rules

### Keep steps thin

A step definition translates one line of business language into one behavioural action, then hands the mechanics to the framework. Business logic, loops, and conditionals do not belong in step code — push them into helpers, page objects, or fixtures (see `test-refactor`).

### Share state through the World, never module globals

Within a scenario, share state via the Cucumber **World** (`this`). Across scenarios, share **nothing** — a fresh World per scenario is what keeps tests independent and parallel-safe. Module-level `let` variables create shared mutable state across scenarios and violate `test-automation-guidelines`.

```javascript
// Good — per-scenario state on the World
Given("Ada has a basket worth {int}", function (total) {
  this.basket = createBasketWorth(total); // isolated to this scenario
});

// Bad — module global leaks across scenarios
let basket; // shared mutable state
```

### Map Gherkin to behaviour, delegate mechanics to the framework

The same feature file is satisfied by framework-specific glue. The step **signature** is identical; only the body changes, and the body obeys the framework skill.

**Cypress glue (obeys test-cypress — chaining, `cy.session`, intercept aliases):**

```javascript
import { When, Then } from "@badeball/cypress-cucumber-preprocessor";

When("she signs in with valid credentials", function () {
  cy.session("ada", () => cy.login(this.user)); // programmatic login
});

Then("she lands on her dashboard", () => {
  cy.get('[data-testid="dashboard"]').should("be.visible");
});
```

**Playwright glue (obeys test-playwright — async/await, auto-waiting, `storageState`):**

```javascript
import { expect } from "@playwright/test";
import { When, Then } from "playwright-bdd/decorators";

When("she signs in with valid credentials", async function ({ page }) {
  await loginViaApi(page, this.user); // programmatic login
});

Then("she lands on her dashboard", async ({ page }) => {
  await expect(page.getByTestId("dashboard")).toBeVisible();
});
```

Note what this skill does **not** dictate: whether to chain or await, how to wait, how to stub the network. Those are the framework skill's job.

---

## When NOT to use BDD

BDD earns its overhead when non-technical stakeholders read or write scenarios, or when living documentation has real value. Be honest about when it is just ceremony:

- **Engineer-only suites** with no business audience — plain framework tests are clearer and cheaper.
- **Unit and contract tests** — Gherkin adds indirection with no readership benefit. Keep BDD for behaviour-level (E2E/acceptance) tests, consistent with the test pyramid in `test-automation-guidelines`.
- **Highly technical assertions** (headers, status codes, schema) that no stakeholder will read.

If you reach for BDD, commit to the declarative discipline above. Imperative Gherkin is worse than no Gherkin: it carries all the cost and none of the readability.

---

## Anti-Patterns

- Imperative, UI-coupled steps (clicks, selectors, URLs in the feature file)
- Conjunction steps (`Given … and …`)
- Scenarios that assert several unrelated behaviours
- `Scenario Outline` used to merge different behaviours
- Logic, loops, or conditionals inside step definitions
- Shared mutable state via module globals instead of the World
- Restating framework rules in this skill instead of delegating
- Tags used as narration, or `@wip`/`@flaky` left in a blocking pipeline

---

## References

For the full declarative-Gherkin style guide with worked before/after rewrites, see `references/gherkin-style.md`.
