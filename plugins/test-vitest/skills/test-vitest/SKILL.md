---
name: test-vitest
description: Use this skill when writing, fixing, or reviewing unit tests that use Vitest — test doubles (vi.fn, vi.spyOn, vi.mock), fake timers, async and error assertions, it.each, and vitest.config.
metadata:
  author: Jacopo Martinelli & Sherrylene Gauci
  version: 1.1.0
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

Vitest-specific mechanics for unit tests. Written against the Vitest 5 docs; mock reset and restore semantics differ in older majors, so check the project's version before relying on them.

## Requires

- `test-unit-guidelines` (which requires `test-automation-guidelines`) — its discipline and the base Guardrails apply. If it is not available, tell the user which plugin to install.

## Rules

### Structure

- `describe` names the unit, `it` names the behaviour. Import `describe`/`it`/`expect`/`vi` explicitly rather than relying on `globals: true`.

### Test doubles

- `vi.fn()` for an injected stub/spy; `vi.spyOn(obj, "m")` to watch a real method; `vi.mock("./mod")` only for boundary modules.
- To fake one export of a collaborator, spread `await importOriginal()` and override that export only.
- Mock the module the unit **imports from**, never the one it lives in — calls inside the same file can't be intercepted, by `vi.mock` or `vi.spyOn`. Inject the dependency instead.
- `vi.mock` is hoisted: anything its factory uses must be created inside it or with `vi.hoisted()`.
- `vi.mock(path)` with no factory uses `__mocks__/` if present, then automocks (arrays become `[]`, primitives keep real values) — the usual cause of "my mock isn't applying".
- To assert a call to a mocked boundary, import the mocked module to get the handle.

### Restoring state

- Set `restoreMocks: true` **and** `clearMocks: true` in `vitest.config`. `restoreMocks` restores only `vi.spyOn` spies; `clearMocks` clears call history on every mock. `mockReset` (not Jest's `resetMocks`) also resets implementations.
- Neither `restoreMocks` nor `clearMocks` resets implementations: a `mockReturnValue`/`mockImplementation` one test sets on a shared `vi.fn()` leaks into later tests. Set per-test behaviour in `beforeEach` or with the `*Once` variants; add `mockReset: true` only if no implementation set in a `vi.mock` factory needs to survive between tests.
- No mock flag releases fake timers: call `vi.useRealTimers()` in `afterEach`.

### Timers and randomness

- Freeze with `vi.useFakeTimers()` / `vi.setSystemTime()` and advance deliberately. Use `vi.advanceTimersByTimeAsync()` when the code awaits between timers.
- Stub `Math.random` with `vi.spyOn` rather than asserting around it.

### Assertions

- `expect(() => fn()).toThrow(Type)` — wrap the call. Rejections: `await expect(p).rejects.toThrow(...)`. Callback assertions: `expect.assertions(n)`.
- `it.each` with `$property` in the title when only data varies.
- Use the most specific matcher (`toHaveLength`, `toMatchObject`); `toBeCloseTo(x, digits)` for computed decimals. Avoid snapshots for unit logic.

### Coverage

- Install `@vitest/coverage-v8` (or `-istanbul`), run `vitest run --coverage` in CI, and read the **branch** column.

## Anti-Patterns

- `vi.mock` factories closing over outer variables without `vi.hoisted()`
- Partially mocking the module under test and expecting its internal calls to hit the mock
- No `restoreMocks`/`clearMocks`, or fake timers never released
- `setTimeout`-based waiting, or sync `advanceTimersByTime` past awaited code
- `try`/`catch` rejection assertions; `expect(fn())` without the arrow
- Truthiness checks (`expect(a === b).toBe(true)`) where a specific matcher exists

## References

- [references/mocking.md](references/mocking.md) — worked examples for doubles, `vi.mock`/`vi.hoisted`, restoring state, timers. Load when writing or reviewing mocks.
- [references/assertions.md](references/assertions.md) — worked examples for errors, rejections, `it.each`, matchers. Load when writing assertions.
