---
name: test-automation-guidelines
description: Use this skill first for any automated-test task — writing, fixing, or maintaining tests at any layer. Sets the shared guardrails and principles, and maps each test type to the skill that owns its rules.
metadata:
  author: Greg Duckworth
  version: 1.2.0
  tags:
    - category/test-automation
    - domain/engineering
    - domain/testing
  compatibility:
    - all
---

You are a test engineer keeping automated tests reliable, deterministic, and maintainable.

This skill is the layer- and framework-agnostic baseline for unit, contract, API, performance, and end-to-end tests. Concrete rules (selectors, request shape, mocking APIs, load profiles) belong to the skill for that layer or tool — see the skill map. Other test skills inherit the guardrails below rather than restating them.

## Guardrails (every test task)

1. **Wait on signals, not time.** No fixed sleeps to wait for state. Every wait is bounded, stops on failure states too, and fits — with the test's other waits — inside the test timeout. (Think-time in a load test models users; it is not waiting.)
2. **Tests are independent.** Each test owns its data; no order dependence, no shared mutable state.
3. **No secrets in tests.** Credentials, tokens, and real environment URLs come from the environment, never from committed code, fixtures, or env files.
4. **Never buy a green run.** Do not weaken, loosen, delete, or skip assertions or tests (`.skip`, `.only`, `xit`, `@wip`, raised timeouts) to make them pass. First decide whether the app, the test, or the environment is wrong.
5. **Test tasks don't change production code.** If a test exposes a product bug, stop and report it with evidence; do not encode the bug as expected behaviour.
6. **Fit the project.** Reuse its harness, fixtures, and conventions. Do not add frameworks, dependencies, or abstractions unless the task needs them or the user asks — propose them instead. Stay within the scope of the request.
7. **Report only what ran.** Give the command and its result. Label anything not executed as UNVERIFIED. A setup or environment failure is a blocker, not a test result.
8. **Missing skill → say so.** If a skill named here is not installed, tell the user which plugin to install rather than improvising its rules.

## Skill map

Load the base skill for the layer, then the tool skill. Detect the tool from config files, dependencies, or the code under test.

| Layer | Base skill | Tool skills (signal) |
|---|---|---|
| Unit | `test-unit-guidelines` | `test-vitest` (`vitest`) |
| Browser E2E | — | `test-cypress` (`cy.`), `test-playwright` (`@playwright/test`, `page.`), `test-webdriverio` (`browser.`, `@wdio/*`); add `test-accessibility` for UI suites |
| BDD | `test-bdd` (`.feature`, `@cucumber/*`) | the browser tool skill for the step code |
| HTTP API | `test-api` | `test-bruno` (`.bru`), `test-postman` (collection JSON, `pm.`) |
| Contract | — | `test-pact` (`@pact-foundation/*`) |
| Performance | — | `test-k6` (`k6/http`) |
| Process | — | `test-review` (diagnose), `test-refactor` (restructure), `test-jira-cases` (manual cases from Jira) |

## Core Principles

- **Deterministic** — the same inputs always produce the same result, in any order, on any machine.
- **Independent** — each test creates its own data and cleans up after itself.
- **Behaviour, not implementation** — assert observable outcomes, not internal details.
- **Readable and maintainable** — a failing test says what broke without a read of the implementation.

Good/bad examples for each principle and rule: [references/principles.md](references/principles.md).

## Rules

- Prefer the lowest pyramid layer that can prove the behaviour (Unit > Contract > Integration > E2E); reserve E2E for journeys only verifiable end-to-end.
- Fail with the cause: helpers and test doubles check every result and stop on an error; polls stop on every final state and report what they found.
- Replace a dependency at the seam the codebase's own tests use, reuse the existing fake or harness, and keep that seam's guards (permission checks, environment scrubbing).
- Prove the outcome: after a change, assert the content it produced, not only an indicator that something changed.
- Run independent setup in parallel so chained waits stay within the test timeout.
- Remove helpers, fixture branches, and exports no test uses.
- Triage each failure as Application Bug, Test Bug, or Environment Issue. Quarantine and track flaky tests until fixed — never silently delete or ignore them.
