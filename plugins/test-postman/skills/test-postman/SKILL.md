---
name: test-postman
description: Use this skill when writing or reviewing Postman API tests to apply Postman-specific best practices for reliable, idiomatic tests using pm.* assertions, collections, environments, and Newman in CI.
metadata:
  author: Greg Duckworth
  version: 1.0.0
  tags:
    - domain/engineering
  compatibility:
    - all
  frameworks:
    - postman
    - newman
---

Use this skill when working with Postman API tests. Postman stores collections and environments as JSON that can be exported, version-controlled, and run headlessly with Newman.

## Requires

- test-automation-guidelines
- test-api

Apply the shared API principles from `test-api` first; the rules below are Postman-specific additions.

## Workflow

1. Confirm the collection and environment layout, and how they are exported/version-controlled.
2. Refer only to the reference file relevant to the task:
   - Collection and environment structure: `references/collections.md`
   - Assertions, scripting, and Newman/CI: `references/scripting-and-assertions.md`
3. Apply the principles and templates from that file.

## Postman Rules

### Collections and structure

- Export the collection and environment JSON into the repo and review changes in pull requests — do not keep tests only in a personal workspace.
- One request per behaviour, named for what it verifies; group into folders by resource/feature.
- Keep the exported JSON stable (avoid churn from unrelated UI reorders) so diffs stay reviewable.

### Variables and environments

- Reference the base URL and config as variables (`{{baseUrl}}`), defined per environment.
- Store credentials as **secret**-typed variables (or Postman Vault), never as committed initial values.
- Scope variables intentionally: environment for config, collection for shared defaults, and `pm.variables` for request-local temporary values.

### Assertions (`pm.test`)

- Assert behaviour inside `pm.test(...)` blocks with clear names.
- Use `pm.response.to.have.status(...)` and `pm.expect(...)` for status, fields, and schema.
- Use `pm.response.to.have.jsonSchema(schema)` for structural validation instead of many manual field checks.
- Assert presence/type/format for volatile values (IDs, timestamps), not exact matches.
- For a lightweight latency guard, assert `pm.response.responseTime` against a generous upper bound (a smoke check, not a load test).

### Scripting

- Use pre-request scripts to fetch tokens and generate unique data; use test scripts for assertions and extraction.
- Persist only the specific values later requests need via `pm.environment.set(...)` / `pm.collectionVariables.set(...)`.
- Keep scripts deterministic — no `setTimeout` waits; rely on the response.

## Anti-Patterns

- Committing environment JSON containing real tokens or API keys.
- Hardcoded full URLs instead of `{{baseUrl}}`.
- A single request with dozens of unrelated `pm.test` assertions.
- Tests that only pass when run in a fixed collection order.
- Tests kept solely in a personal workspace, never exported or reviewed.
