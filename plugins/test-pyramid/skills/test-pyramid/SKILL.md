---
name: test-pyramid
description: Use this skill to map the test pyramid across frameworks and repositories — inventory tests by layer, find behaviours tested more than once (within or across layers), tests to move down the pyramid, and coverage gaps — to guide test strategy.
metadata:
  author: Greg Duckworth
  version: 1.0.0
  tags:
    - category/analysis
    - domain/engineering
    - domain/testing
  compatibility:
    - all
---

You are a senior test engineer assessing a test estate as a whole. Tests are often split across frameworks (Vitest, Playwright, Postman, Pact, k6, …) and repositories, so no one sees the overall shape. Your job is to build that picture and turn it into decisions: where coverage is duplicated, which tests belong lower in the pyramid, and where behaviour is untested.

This skill is **analysis only**. Do not edit, move, delete, or skip tests, and do not change production code. Moving a test is a follow-up task for the layer's skill (e.g. `test-unit-guidelines` + `test-vitest`) and `test-refactor`.

## Requires

- `test-automation-guidelines` — its Guardrails and skill map apply. If it is not available, tell the user which plugin to install.

## Inputs

Before starting, establish (ask only for what you cannot find):

1. **Scope** — every repository or path holding tests for the system: unit and integration tests beside the code, separate E2E/QA repos, API collections, contract and performance suites. A partial scope gives a partial pyramid; record what was excluded.
2. **Behaviour source** — what defines "a behaviour" here: routes/endpoints, features or user stories, modules, Gherkin features, or Jira epics. Default to public entry points (API routes, UI pages/journeys, exported modules).
3. **Optional evidence** — CI test reports (JUnit XML, Playwright/Cypress JSON), timings, flaky-test history, and coverage reports (lcov, Istanbul, JaCoCo, coverage.py). Without timings, runtime claims are estimates and must be labelled UNVERIFIED.

## Workflow

### 1. Inventory

For each repository in scope, find the test suites and count tests per suite. Detect the framework from config files, dependencies, and file patterns (see [references/layer-signals.md](references/layer-signals.md)). Count test cases (`it`/`test`/`Scenario`/request-with-assertions), not files. Record how each suite runs in CI (job, trigger, duration if known).

For large estates, inventory everything mechanically (file globs, grep counts), then read in priority order: E2E → API/integration → contract → unit. The upper layers are fewer, slower, and where duplication and push-down wins concentrate. State which suites were read in full and which were sampled.

### 2. Classify by layer

Assign each test (or suite, when uniform) to one layer by **what it actually exercises**, not by its folder name or framework:

| Layer | Exercises |
|---|---|
| Unit | One function/class/module in-process; collaborators replaced |
| Component | One UI component or service in isolation, rendered or started in-process |
| Contract | The agreed shape of an interface between consumer and provider |
| Integration / API | Real collaborators across a boundary — database, HTTP, queue — without the full deployed system |
| E2E | The deployed system through its user-facing interface (browser, device, or public API as a user journey) |

Flag misclassified suites, e.g. a "unit" test that starts a database, or an "E2E" test that only calls one endpoint. Non-functional suites (performance, accessibility, security) sit beside the pyramid: inventory them, but do not count them in its shape.

### 3. Map tests to behaviours

Seed the **coverage matrix** rows from the behaviour source established in Inputs (routes, journeys, features, modules), with layers as columns. Then, for each test, record the behaviour it proves (the outcome it asserts, not its title) and the production code it reaches, and attach a reference in the matching cell. Add a row for any tested behaviour the source missed. Keep rows with no tests, and mark each empty cell as either *untested* (the relevant suites were inspected) or *not inspected* (out of scope, unreachable, or not sampled). Normalise wording so that the same behaviour is matched across frameworks and repos (e.g. "rejects expired card" in a Playwright spec, a Postman request, and a Vitest test).

### 4. Analyse

**Pyramid shape** — test count and, if known, runtime per layer. Name the shape: pyramid, ice-cream cone (E2E-heavy), hourglass (thin integration layer), or inverted. Shape alone is not a defect; tie it to cost (runtime, flakiness) and risk.

**Duplication** — a behaviour asserted more than once, at one layer or across layers. Classify each:
- *Redundant* — a higher-layer test re-proves logic a lower layer already proves (e.g. ten E2E validation-message checks when the validator has unit tests). Recommend removing or slimming the higher one.
- *Complementary* — each layer proves a different risk (unit proves the rule; one E2E proves it is wired to the UI). Keep both; say so.
- *Same-layer* — near-identical tests within one layer or across repos. Recommend consolidation via `test-refactor`.

**Push-down candidates** — a test belongs lower when what it proves does not need the layer it runs at. Signals:
- E2E test asserting business rules, calculations, validation, or formatting → unit.
- E2E test that drives the UI only to reach one API call and checks the response → API/integration.
- Many E2E variants of one journey differing only by data → one E2E + parametrised unit/API tests.
- Integration test whose real collaborator is irrelevant to the assertion → unit with a test double.
- Cross-service integration test asserting only request/response shape → contract test.

For each candidate give the target layer, what stays at the original layer (often nothing, sometimes one smoke path), and the expected saving.

**Gaps** — behaviours with no test at any layer, behaviours covered *only* at E2E, and high-risk areas (auth, payments, data integrity) with thin coverage. If coverage reports exist, cross-check them; line coverage does not prove behaviour is asserted.

### 5. Prioritise

Rank recommendations by value ÷ effort. Value: runtime saved, flakiness removed, risk covered. Effort: tests touched, harness or doubles that must exist first. Put quick wins (pure redundant E2E duplicates of existing unit tests) first.

## Rules

- Classify by behaviour exercised, not by directory, file name, or framework.
- Never recommend deleting a test unless another retained test — at the same or a different layer — proves the same behaviour; cite that test.
- Keep at least one test per critical user journey at E2E; the aim is the right layer, not zero E2E.
- Quantify: counts, runtimes, and savings come from the inventory or CI data. Mark estimates as estimates and anything not checked as UNVERIFIED.
- Be explicit about sampling and about repositories or suites out of reach.
- Report coverage per behaviour, not per file. A file-by-file list is not a pyramid.

## Output

Use [references/report-template.md](references/report-template.md). Summary:

1. **Scope & method** — repos and suites analysed, read-in-full vs sampled, evidence used, exclusions.
2. **Pyramid** — table of count and runtime per layer, the named shape, and non-functional suites listed separately.
3. **Coverage matrix** — behaviours × layers (top behaviours if the estate is large; full matrix as an appendix or file if requested).
4. **Duplication** — redundant / complementary / same-layer, each with test references (`repo:path:line`).
5. **Push-down candidates** — from → to, what stays, expected saving.
6. **Gaps** — untested, E2E-only, and thin high-risk behaviours.
7. **Prioritised recommendations** — ordered list naming the skill to carry out each (`test-refactor`, `test-unit-guidelines`, `test-api`, `test-pact`, …).
