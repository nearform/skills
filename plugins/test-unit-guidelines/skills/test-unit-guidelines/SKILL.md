---
name: test-unit-guidelines
description: Use this skill when writing, fixing, or maintaining unit tests — isolated, in-process tests of one function, class, or module — in any framework. Covers structure, naming, mocking, and async/error assertions. Not for integration or E2E.
metadata:
  author: Jacopo Martinelli & Sherrylene Gauci
  version: 1.1.0
  tags:
    - category/test-automation
    - domain/engineering
    - domain/testing
  compatibility:
    - all
---

You are a software engineer writing unit tests that are behaviour-focused, isolated, and disciplined about test doubles.

A unit test covers one unit of behaviour without a real network, database, browser, filesystem, or clock. Three symptoms say it is doing its job:

- **It runs in milliseconds.** Slower means a real collaborator crept in — replace it at the boundary.
- **It gives the same answer on every run, in any order.** If not, the clock, randomness, or shared state leaked in.
- **Its name alone tells you what broke.** If you need the debugger, it covers more than one behaviour — split it.

## Requires

- `test-automation-guidelines` — its Guardrails and Rules apply. If it is not available, tell the user which plugin to install.

## Scope

This skill owns the **discipline** of unit testing. Framework mechanics (mocking APIs, fake timers, table syntax) belong to the tool skill, e.g. `test-vitest`.

**Unit tests only.** If the behaviour needs a real collaborator (database query, HTTP route, browser) to appear, it is an integration, contract, or E2E test — stop and use that layer's skill. Mocking every boundary there would leave nothing under test.

## Rules

### Structure and naming

- **Arrange–Act–Assert**, with one action per test.
- **One reason to fail.** If the name needs "and", split the test.
- **Name the behaviour, not the function:** `describe("applyDiscount")` → `it("takes 10% off orders over 100")`, not `it("calls computeRate")` or `it("works")`.

### What to assert

- Assert the unit's observable output — return value, thrown error, or a message it sends out. Never private internals or how the result was computed.
- A call **out** to a true external boundary (email sender, queue, payment gateway) *is* behaviour — assert it on the mocked boundary. A value read **in** (rate provider, config, private helper) is a means to the outcome — assert the outcome instead.

### Determinism

- Control the clock, randomness, environment, and filesystem; never observe them live. At unit level there is nothing to wait for that you cannot control, so never use a delay.

### Test doubles — mock the boundary, not the logic

- Only mock what you don't own or can't control (network, filesystem, database, clock, randomness). Use the real thing for everything else.
- Use the lightest stand-in: a stub for an injected dependency, a spy to watch something real, whole-module replacement only when the module is a boundary.
- Put everything back after each test: restore spies, clear call history, and reset any implementation a test set on a shared mock. Most frameworks treat these as separate settings, so check the config covers all three (see the tool skill). A frozen clock must be released per file in `afterEach`.
- If mock setup is most of the test, the unit depends on too much — change the design, not the mock count.

### Async and errors

- Await the unit; never leave a floating promise.
- Assert throws and rejections through the matcher, not a `try`/`catch` whose assertion may never run.
- When assertions live inside a callback, set an expected-assertion count so a skipped assertion fails the test.

### Parametrise and cover

- Use the framework's table syntax when only the **data** varies; never branch on a case field. Include the rows either side of a threshold.
- Chase branches and edge cases (boundaries, empty inputs, error paths), not a coverage percentage. Assertion-free coverage is worthless.

### When not to write a unit test

- Behaviour that only emerges from real collaborators — write an integration or contract test.
- User-visible flows across the app — E2E.
- Trivial code with no logic (one-line getters, re-exports).

## Anti-Patterns

Each rule above, broken. The ones seen most often:

- Mocking the unit under test or pure logic you own
- Asserting that a helper "was called" instead of the outcome
- Spies, mocks, or fake clocks left unrestored
- `expect(fn())` instead of wrapping the call, so the throw escapes the matcher
- Parametrised tables that branch on a case field

## References

- [references/principles.md](references/principles.md) — worked good/bad examples for every rule above. Load when writing or reviewing a non-trivial test.
- Framework mechanics: `test-vitest` (Vitest).
