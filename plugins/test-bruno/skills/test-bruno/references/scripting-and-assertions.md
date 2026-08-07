# Bruno Assertions, Scripting, and Running

Apply this guidance when adding assertions, scripts, or running Bruno tests.

## Declarative assertions (`assert`)

Prefer the `assert` block for status and straightforward field checks:

```
assert {
  res.status: eq 200
  res.body.id: isDefined
  res.body.status: eq "active"
  res.headers["content-type"]: contains "application/json"
}
```

## Script assertions (`tests`)

Use the `tests` block with `expect` for schema/shape and richer logic:

```
tests {
  const body = res.getBody();
  expect(res.getStatus()).to.equal(200);
  expect(body).to.be.an("array");
  expect(body[0]).to.have.property("id");
}
```

- Use `tests` for structure, arrays, and conditional logic; keep simple checks in `assert`.
- Assert presence/type/format for volatile fields (IDs, timestamps) rather than exact values.

## Scripting (pre-request / post-response)

- `pre-request` scripts: obtain auth tokens, generate unique test data, set variables.
- `post-response` scripts: extract values and clean up created data.
- Capture only what later steps need:

```
script:post-response {
  bru.setVar("orderId", res.getBody().id);
}
```

- Read secrets from the environment (`bru.getEnvVar` / `process.env`), never inline literals.
- Keep scripts deterministic — no arbitrary `setTimeout`/waits.

## Running

- Run locally in the Bruno app for authoring, and via the Bruno CLI (`bru run`) in CI.
- Select the environment explicitly: `bru run <folder> --env staging`.
- Provide secrets to the CLI via environment variables or `--env-var`, not committed files.
- Fail the CI job on any failed assertion; publish the run report as a build artifact where possible.

## Anti-patterns

- Re-implementing simple status/field checks in `tests` that `assert` already covers.
- Printing full tokens in `console.log` inside scripts.
- Depending on run order instead of setting up each request's own preconditions.
- Committing secrets to pass them to `bru run`.
