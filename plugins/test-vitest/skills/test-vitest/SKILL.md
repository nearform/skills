---
name: test-vitest
description: Use this skill when writing, reviewing, or refactoring unit tests with Vitest, to apply Vitest-specific conventions for test doubles, fake timers, async assertions, parametrisation, and config.
metadata:
  author: Jacopo Martinelli & Sherrylene Gauci
  version: 1.0.0
  tags:
    - category/test-automation
    - domain/engineering
    - domain/testing
    - tool/vitest
  compatibility:
    - all
  frameworks:
    - vitest
---

Use this skill when working with [Vitest](https://vitest.dev) unit tests. The semantics below are **Vitest 2 and later** — mock resetting and restoration behave differently on 1.x.

## Requires

- test-unit-guidelines

---

## Vitest Rules

### Test Structure

Use `describe` to name the unit and `it` to name the behaviour. Import the globals explicitly rather than relying on `globals: true`, so a test file's dependencies are visible.

```javascript
import { describe, it, expect } from "vitest";
import { applyDiscount } from "./pricing";

describe("applyDiscount", () => {
  it("takes 10% off orders over 100", () => {
    expect(applyDiscount({ total: 200 }).total).toBe(180);
  });
});
```

Every example below elides its `import { … } from "vitest"` line for brevity. A real test file still needs it — `vi`, `it`, `expect`, `describe`, and `afterEach` are all named imports unless `globals: true` is set, which this skill advises against.

---

### Test Doubles

Use the lightest stand-in that does the job:

- `vi.fn()` — a standalone stub/spy for an injected callback or dependency.
- `vi.spyOn(obj, "method")` — wrap a real method to assert calls while keeping or overriding its behaviour.
- `vi.mock("./module")` — replace a whole module at import time, for boundary modules only.

```javascript
// Good — mock only the external boundary; use the real thing for logic you own
import { toUsd } from "./currency";

vi.mock("./httpClient", () => ({
  fetchRates: vi.fn().mockResolvedValue({ usd: 1.5 }),
}));

it("converts using the live rate", async () => {
  expect(await toUsd(100)).toBe(150);
});
```

Called with no factory, `vi.mock("./taxes")` does **not** automock straight away: Vitest first looks for `__mocks__/taxes.js` alongside the module and uses that instead if it exists. This is worth knowing — it's the usual cause of "my mock isn't applying".

Failing that it automocks, which is not a blanket `vi.fn()`: methods and getters return `undefined`, arrays come back empty, primitives are left untouched, and objects and class instances are deep-cloned. So a primitive export such as `VERSION` survives with its real value, while an array export like `TAX_RATES` comes back as `[]` — the right type with the data silently gone.

When asserting a message to a real boundary (see "Test behaviour, not implementation" in `test-unit-guidelines`), import the mocked module to get a handle on the mock — a factory's return value reaches *importers*, it does not create local bindings in the test file.

```javascript
// Good — the outbound message IS the behaviour under test
import { send } from "./emailService";

vi.mock("./emailService", () => ({ send: vi.fn() }));

it("emails the customer when the order ships", () => {
  markShipped({ email: "ada@example.com" });
  expect(send).toHaveBeenCalledWith("ada@example.com", "shipped");
});
```

When only part of a collaborator module is a boundary, replace that export and keep the rest real. This is usually what you want — whole-module replacement throws away logic you meant to exercise.

```javascript
// Good — fake the one boundary export of a collaborator, keep the rest of it real
vi.mock("./taxes", async (importOriginal) => ({
  ...(await importOriginal()),
  loadTaxTable: vi.fn().mockResolvedValue({ vat: 0.2 }),
}));
```

Mock the module the unit **imports from**, never the one it lives in: Vitest cannot intercept a call made from inside the same file, so partially mocking `./pricing` while testing `applyDiscount` leaves its internal calls hitting the real code — silently, with the mock recording nothing. A boundary reached from inside the same module needs dependency injection — pass it in. `vi.spyOn` does not rescue it either: a direct call to a module-local function never goes through the namespace object the spy replaces.

`vi.mock` is hoisted above imports. Anything its factory references must be created inside the factory, or declared with `vi.hoisted()`.

```javascript
// Good — the factory's dependencies are hoisted with it
const { fetchRates } = vi.hoisted(() => ({ fetchRates: vi.fn() }));
vi.mock("./httpClient", () => ({ fetchRates }));

// Bad — factory closes over a variable that doesn't exist yet
const fetchRates = vi.fn(); // evaluated after vi.mock is hoisted
vi.mock("./httpClient", () => ({ fetchRates })); // ReferenceError
```

---

### Restoring State

Every spy and mock must be undone between tests. Set it once in `vitest.config` rather than remembering it per file:

```javascript
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    restoreMocks: true,
    clearMocks: true,
  },
});
```

Know the difference (Vitest 2+):

- `clearMocks` — clears call history before each test. Implementations stay.
- `mockReset` — note the name, *not* `resetMocks` (that's Jest's). Also resets implementations: a `vi.fn(impl)` goes back to `impl`, and a `vi.spyOn` spy starts calling through to the real method it wraps again. Only a bare `vi.fn()` with no implementation resets to a no-op returning `undefined`.
- `restoreMocks` — restores the original descriptors of anything created with `vi.spyOn`.

Set `restoreMocks` **and** `clearMocks`: restoration applies only to `vi.spyOn` spies, so `clearMocks` is what clears history on plain `vi.fn()` mocks. Where a shared config isn't available, call `vi.restoreAllMocks()` and `vi.clearAllMocks()` in `afterEach` — `restoreAllMocks` is likewise spy-only.

```javascript
// Bad — a spy that outlives its test
it("logs a warning", () => {
  vi.spyOn(console, "warn").mockImplementation(() => {});
  expect(validate({})).toBe(false);
  // console.warn stays stubbed for every later test in the file
});
```

→ Fix: set `restoreMocks: true` in `vitest.config`.

Note that none of these flags touch fake timers — see below.

---

### Timers & Randomness

Freeze the clock, advance it deliberately, then release it. **No mock-reset flag releases timers** — `restoreMocks` does not touch them, so do it in `afterEach` or the frozen clock leaks into every later test in the file.

```javascript
// Good — control the clock, and always release it
afterEach(() => {
  vi.useRealTimers();
});

it("expires the token after one hour", () => {
  vi.useFakeTimers();
  vi.setSystemTime(new Date("2026-01-01T00:00:00Z"));

  const token = issueToken();
  vi.advanceTimersByTime(60 * 60 * 1000);

  expect(isExpired(token)).toBe(true);
});
```

Use `vi.advanceTimersByTimeAsync()` when the code under test awaits between timers — the synchronous form never lets the awaited work run.

```javascript
// Good
it("retries after a backoff", async () => {
  vi.useFakeTimers();
  const promise = fetchWithRetry("/rates");

  await vi.advanceTimersByTimeAsync(1000); // lets the awaited retry run

  await expect(promise).resolves.toEqual({ usd: 1.5 });
});

// Bad — advancing past awaited code synchronously
vi.advanceTimersByTime(1000); // the awaited retry never gets to run; test hangs or fails
```

Randomness is a boundary too — stub it rather than asserting around it. `restoreMocks: true` puts `Math.random` back afterwards.

```javascript
// Good — stub the boundary, assert the outcome
it("picks the weighted winner", () => {
  vi.spyOn(Math, "random").mockReturnValue(0.42);
  expect(pickWinner(entrants)).toBe("ada");
});
```

---

### Errors & Callback Assertions

Wrap the call in a function so the matcher can catch the throw. `expect(fn())` runs `fn` eagerly, so the error escapes before `toThrow` can see it and the test dies with an uncaught exception instead of a readable failure. Assert the error type where the type is part of the contract.

```javascript
// Good — the matcher gets a chance to catch it
expect(() => validateOrder({})).toThrow(TypeError);
expect(() => validateOrder({})).toThrow("total is required");

// Bad — no arrow, so the error never reaches the matcher
expect(validateOrder({})).toThrow("total is required");
```

Assert rejections with `.rejects`, never a `try`/`catch` whose `expect` may never run.

```javascript
it("rejects an unknown user", async () => {
  await expect(loadProfile("nobody")).rejects.toThrow("not found");
});
```

When an assertion can only run inside a callback, `expect.assertions(n)` fails the test if that callback never fires:

```javascript
it("invokes the callback with the parsed row", () => {
  expect.assertions(1); // guarantees the assertion below actually ran
  parseCsv("a,1", (row) => expect(row).toEqual({ a: "1" }));
});
```

---

### Parametrisation

Use `it.each` when the behaviour is identical and only the data varies. Use the `$property` syntax in the title so each case reports a distinct, readable name.

```javascript
// Good — same behaviour, varying data (note the boundary rows)
it.each([
  { total: 50, expected: 50 }, // below the threshold — untouched
  { total: 100, expected: 100 }, // exactly 100 — not "over", untouched
  { total: 200, expected: 180 }, // over 100 — 10% off
])("applies the right discount to an order of $total", ({ total, expected }) => {
  expect(applyDiscount({ total }).total).toBe(expected);
});
```

---

### Assertions

Prefer the most specific matcher available — it produces a better failure message than a generic truthiness check.

```javascript
// Good — specific matchers, readable failures
expect(result.total).toBe(180);
expect(items).toHaveLength(3);
expect(user).toMatchObject({ name: "Ada" }); // partial shape, not deep-equal on everything

// Bad — generic checks with weaker failure messages
expect(result.total === 180).toBe(true); // "expected false to be true" — the actual value is gone
expect(items.length).toBe(3); // reports the count, but loses the array from the diff
expect(JSON.stringify(user)).toContain("Ada"); // substring match on a serialised blob
```

`toBe` demands an exact match, and decimal arithmetic rarely gives one — `0.1 + 0.2` comes out as `0.30000000000000004`, and `100 * 1.1` as `110.00000000000001`. When a number is the result of a calculation, assert it with `toBeCloseTo` and say how many decimal places matter.

```javascript
expect(0.1 + 0.2).toBeCloseTo(0.3, 5); // toBe(0.3) fails
```

Avoid snapshot tests for unit-level logic — they assert everything and therefore nothing in particular. Reserve them for large, stable serialised output.

---

### Coverage

Install a provider first — `@vitest/coverage-v8` (or `@vitest/coverage-istanbul`); Vitest prompts for it on first run.

Use `vitest run --coverage` in CI. Bare `vitest` does fall back to run mode when it detects CI or a non-interactive terminal, but `run` is explicit and does not depend on that detection.

Read the **branch** column, not the line percentage.

---

## Anti-Patterns

- Relying on `globals: true` instead of importing `describe`/`it`/`expect`
- `vi.mock` factories referencing outer variables without `vi.hoisted()`
- Partially mocking the module under test, expecting its internal calls to hit the mock
- Leaving mocks or fake timers unrestored (no `restoreMocks`, no `vi.useRealTimers()`)
- `setTimeout`-based waiting instead of `vi.advanceTimersByTime`
- `advanceTimersByTime` where the code under test awaits between timers
- `try`/`catch` rejection assertions instead of `.rejects`
- `expect(fn())` instead of `expect(() => fn())`, letting a thrown error escape the matcher
- Assertions inside a callback with no `expect.assertions(n)` to prove they ran
- Snapshot tests standing in for real assertions on unit logic
- Truthiness assertions where a specific matcher exists
