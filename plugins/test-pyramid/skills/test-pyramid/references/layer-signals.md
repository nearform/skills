# Layer signals

Use these signals to find suites and make a first-pass classification. Always confirm by reading what a test exercises — a signal is a hint, not a verdict.

## Finding and counting tests

| Framework | Detect by | Count |
|---|---|---|
| Vitest / Jest / Mocha | `vitest`, `jest`, `mocha` in dependencies; `*.test.*`, `*.spec.*`, `__tests__/` | `it(`, `test(` and modified forms (`it.each`, `test.each`, `test.concurrent`, `it.only`); exclude `.skip`, `.todo` and note them separately |
| pytest / unittest | `pytest.ini`, `conftest.py`, `test_*.py` | `def test_` functions, parametrised cases expanded if reports exist |
| JUnit / TestNG | `src/test/java`, `@Test` | `@Test` methods; `@ParameterizedTest` / `@DataProvider` cases expanded if reports exist |
| Playwright | `@playwright/test`, `playwright.config.*` | `test(` and modified forms (`test.describe.parallel` children, `test.fixme` noted separately) in spec files |
| Cypress | `cypress.config.*`, `cypress/e2e/` | `it(` in spec files |
| WebdriverIO | `@wdio/*`, `wdio.conf.*` | `it(` in spec files |
| Cucumber / BDD | `.feature` files, `@cucumber/*` | `Scenario:` + rows in each `Examples:` table |
| Postman | collection JSON, `pm.test` | requests containing `pm.test` assertions |
| Bruno | `.bru` files | requests with `tests` / `assert` blocks |
| Pact | `@pact-foundation/*`, `pact/` dir | interactions per consumer–provider pair |
| k6 | `import http from 'k6/http'` | scenarios (non-functional — list separately) |

Prefer counts from CI reports (JUnit XML, JSON reporters) when available: they reflect parametrisation and what actually ran.

Static counts miss tests generated at runtime: `.each`/parametrised tables, tests declared inside loops (`for (const c of cases) test(...)`), and data-driven runners. Without CI reports, expand them by reading the data source where practical; otherwise count each generator once, flag it, and label that suite's count an **estimate**. Never let an unexpanded generator make a populated suite look empty or small.

## Layer signals inside the test

| Signal | Suggests |
|---|---|
| Collaborators replaced with mocks/stubs/fakes; no I/O | Unit |
| Renders one component (`render(`, `mount(`, Testing Library, Storybook interaction tests) | Component |
| Starts the app in-process (`supertest`, `TestClient`, `WebApplicationFactory`, `fastify.inject`) with a real or containerised DB | Integration / API |
| Testcontainers, docker-compose in test setup, real queues | Integration |
| HTTP calls to a deployed base URL with assertions on one endpoint | Integration / API (even if in an "e2e" folder) |
| Pact consumer/provider verification | Contract |
| Browser or device driving a deployed environment through several steps | E2E |
| Browser test that intercepts and stubs every backend call | Component / UI integration — not full E2E |

## Common misclassifications

- `e2e/` folder containing single-endpoint API checks → Integration / API.
- "Unit" tests that open a DB connection or hit the network → Integration (and often slow/flaky).
- Postman collections that chain a full user journey across services → E2E at the API level.
- Playwright tests using the `request` fixture only, no `page` → API level, but check the scope: an isolated endpoint check is Integration / API; a chained journey across services against a deployed system is E2E at the API level (as with Postman above).
