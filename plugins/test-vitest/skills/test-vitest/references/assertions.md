# Vitest: errors, parametrisation, and matchers

Load when writing or reviewing assertions, `it.each` tables, or async/error tests.

Examples elide their `import { … } from "vitest"` line; a real test file needs it.

## Errors & Callback Assertions

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

## Parametrisation

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

## Assertions

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
