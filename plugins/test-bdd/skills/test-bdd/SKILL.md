---
name: test-bdd
description: Use this skill when writing or reviewing BDD/Cucumber feature files and step definitions. Keeps Gherkin declarative and steps thin, and delegates selectors, waiting, and network mechanics to the framework skill.
metadata:
  author: Greg Duckworth
  version: 1.1.0
  tags:
    - category/test-automation
    - domain/engineering
    - domain/testing
    - tool/cucumber
  compatibility:
    - all
  frameworks:
    - cucumber
    - cypress
    - playwright
    - webdriverio
---

A **methodology** skill. It owns the specification layer (how scenarios are written) and the glue layer (how step definitions are organised). It does not own framework mechanics.

## Requires

- `test-automation-guidelines` — its Guardrails and Rules apply. If it is not available, tell the user which plugin to install.
- The framework skill for the code inside each step, per the skill map: `test-cypress` (`cy.`), `test-playwright` (`playwright-bdd`, `page.`), or `test-webdriverio` (`browser.`, `@wdio/*`).

BDD owns the `.feature` file and the *shape* of each step; the framework skill owns the *code inside* it. Never restate framework rules here.

## Gherkin Rules

- **Declarative, not imperative.** Describe what the user achieves and why — `When she signs in with valid credentials`, not `When I type "ada@example.com" into "#email"`. Selectors, typing, and URLs live in step definitions.
- **One behaviour per scenario.** If the name needs "and", split it.
- **`Background` holds shared `Given` context only** — no `When`/`Then`. If it grows beyond a few lines, the scenarios are coupled.
- **`Scenario Outline` only when the behaviour is identical** and only the data varies.
- **No conjunction steps.** Split `Given the user is logged in and has an empty basket` into two steps.
- **Reusable, unambiguous phrasing.** Business vocabulary, consistent person and tense, parameterised nouns. Steps that read differently must not do the same thing, and vice versa.
- **Tags are for selection and lifecycle** (`@smoke`, `@regression`, `@wip`, `@flaky`), from a small documented vocabulary. Wire auth and seeding to tagged hooks. Never ship `@wip`/`@flaky` to a blocking pipeline — quarantine and track instead.

## Step Definition Rules

- **Keep steps thin** — one line of business language maps to one behavioural action. Move logic, loops, and conditionals into helpers, page objects, or fixtures.
- **Per-scenario state only.** Use the Cucumber World (`this`) in Cypress and WebdriverIO, or fixtures in playwright-bdd. Never module-level `let` variables; share nothing across scenarios.
- **Same step text, framework-specific body** — see [references/step-definitions.md](references/step-definitions.md).

## When not to use BDD

- Engineer-only suites with no business readership — plain framework tests are clearer.
- Unit and contract tests, or highly technical assertions (headers, status codes, schema) no stakeholder reads.

Imperative Gherkin is worse than no Gherkin: all the cost, none of the readability.

## Anti-Patterns

- Imperative, UI-coupled steps (clicks, selectors, URLs in the feature file)
- Conjunction steps, or scenarios asserting several behaviours
- `Scenario Outline` merging different behaviours
- Logic, loops, or conditionals inside step definitions
- Module globals instead of World/fixtures for state
- `@wip`/`@flaky` left in a blocking pipeline

## References

- [references/gherkin-style.md](references/gherkin-style.md) — worked good/bad Gherkin for every rule above. Load when writing or reviewing feature files.
- [references/step-definitions.md](references/step-definitions.md) — glue for Cypress, playwright-bdd, and WebdriverIO. Load when writing step definitions.
