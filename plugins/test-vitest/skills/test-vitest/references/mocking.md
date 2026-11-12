# Vitest: test doubles and restoring state

Checked against the Vitest 5 docs. Load when writing or reviewing mocks, spies, or `vi.mock`.

Examples elide their `import { … } from "vitest"` line; a real test file needs it.

## Test Doubles

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

When asserting a message to a real boundary (see "What to assert" under Rules in `test-unit-guidelines`), import the mocked module to get a handle on the mock — a factory's return value reaches *importers*, it does not create local bindings in the test file.

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

## Restoring State

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

Know the difference:

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

Neither flag resets **implementations** on a plain `vi.fn()`. Behaviour one test sets on a shared mock carries into the next:

```javascript
// Bad — the rejection set here leaks into every later test in the file
it("shows an error when rates fail", async () => {
  fetchRates.mockRejectedValue(new Error("down"));
  await expect(toUsd(100)).rejects.toThrow("down");
});

// Good — scope it to one call, or set defaults in beforeEach
it("shows an error when rates fail", async () => {
  fetchRates.mockRejectedValueOnce(new Error("down"));
  await expect(toUsd(100)).rejects.toThrow("down");
});
```

`mockReset: true` resets implementations too, but it also wipes ones set in a `vi.mock` factory (such as `vi.fn().mockResolvedValue(...)` above), so enable it only when every test sets the behaviour it needs.

Note that none of these flags touch fake timers — see below.

## Timers & Randomness

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
