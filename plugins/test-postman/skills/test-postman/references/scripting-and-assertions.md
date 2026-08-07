# Postman Assertions, Scripting, and Newman

Apply this guidance when adding assertions, scripts, or running Postman tests in CI.

## Assertions (`pm.test`)

Wrap each assertion in a named `pm.test` so failures are self-describing:

```javascript
pm.test("returns 201", () => {
  pm.response.to.have.status(201);
});

pm.test("body has an id", () => {
  const body = pm.response.json();
  pm.expect(body).to.have.property("id");
});
```

- Assert the status, then the key fields that define the behaviour.
- For volatile values, assert presence/type/format, not exact matches:

```javascript
pm.expect(pm.response.json().id).to.be.a("string");
```

## Schema validation

Prefer schema validation over many manual field checks:

```javascript
const schema = {
  type: "object",
  required: ["id", "status"],
  properties: {
    id: { type: "string" },
    status: { type: "string" },
  },
};
pm.test("matches schema", () => {
  pm.response.to.have.jsonSchema(schema);
});
```

## Response time

Use a lightweight latency guard as a smoke check — not a substitute for a proper load test:

```javascript
pm.test("responds within 800ms", () => {
  pm.expect(pm.response.responseTime).to.be.below(800);
});
```

- Set the bound generously above expected latency so it flags gross regressions, not normal variance.
- Keep performance-sensitive thresholds in dedicated performance tests (e.g. k6), not scattered across functional tests.

## Scripting

- **Pre-request scripts:** fetch auth tokens, generate unique test data, set variables.
- **Test scripts:** assert responses and extract values for later requests.
- Persist only what later requests need, at the right scope:

```javascript
pm.environment.set("orderId", pm.response.json().id);
```

- Read secrets from environment/vault variables; never inline literals or `console.log` full tokens.
- Keep scripts deterministic — no arbitrary waits.

## Running with Newman (CI)

- Run headlessly with Newman: `newman run collection.json -e staging.postman_environment.json`.
- Supply secrets via `--env-var "token=$TOKEN"` or a CI-provided environment file, not committed values.
- Emit machine-readable output for CI:

```bash
newman run collection.json -e staging.json \
  --reporters cli,junit --reporter-junit-export results.xml
```

- Fail the build on any failed assertion (Newman exits non-zero) and publish the report as an artifact.
- Consider `--bail` to stop on first failure in fast-feedback pipelines.

## Anti-patterns

- Assertions outside `pm.test`, so a failure doesn't identify which check broke.
- Dozens of unrelated assertions crammed into one request.
- Passing secrets to Newman via committed environment JSON.
- Depending on collection run order for tests that should be independent.
