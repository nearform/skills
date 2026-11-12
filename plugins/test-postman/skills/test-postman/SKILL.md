---
name: test-postman
description: Use this skill when writing, fixing, or reviewing Postman test scripts — pm.test/pm.expect assertions, collection and environment structure, variables and secrets, and running collections in CI with the Postman CLI or Newman.
metadata:
  author: Greg Duckworth
  version: 1.1.0
  tags:
    - category/api-testing
    - domain/engineering
    - domain/testing
    - tool/postman
    - tool/newman
  compatibility:
    - all
  frameworks:
    - postman
    - newman
---

Postman-specific rules for test scripts and collections.

## Requires

- `test-api` (which requires `test-automation-guidelines`) — its API rules and the base Guardrails apply. If either is not available, tell the user which plugin to install.

## Rules

### Collections in the repo

- Keep collections and environments in version control and review changes in PRs, never only in a personal workspace. Follow the repo's existing format: Postman's Git-native collection files if present, otherwise exported collection JSON.
- One request per behaviour, named for what it verifies; folders by resource or feature. Keep exports stable (no unrelated reorder churn) so diffs stay reviewable.

### Variables and environments

- Config as variables (`{{baseUrl}}`) per environment. Credentials as **secret**-typed variables or Postman Vault, never as committed initial values.
- Scope intentionally: environment for config, collection for shared defaults, `pm.variables` for request-local values.

### Assertions

- Assert inside named `pm.test(...)` blocks with `pm.response.to.have.status(...)` and `pm.expect(...)`.
- Prefer `pm.response.to.have.jsonSchema(schema)` over many manual field checks.
- `pm.response.responseTime` against a generous bound is a smoke check only, not a load test.

### Scripting

- Pre-request scripts fetch tokens and generate unique data; test scripts assert and extract.
- Persist only values later requests need (`pm.collectionVariables.set(...)`). No `setTimeout` waits.

### Running in CI

- Use the runner the project already uses — the Postman CLI (`postman collection run`) or Newman — with the environment selected explicitly.

## Anti-Patterns

- Committed environment files containing real tokens or keys
- Hardcoded full URLs instead of `{{baseUrl}}`
- One request with dozens of unrelated `pm.test` assertions
- Tests that only pass in a fixed collection order

## References

Load only the one you need:

- [references/collections.md](references/collections.md) — collection and environment structure.
- [references/scripting-and-assertions.md](references/scripting-and-assertions.md) — assertions, scripting, and CI runs.
