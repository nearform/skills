---
name: test-bruno
description: Use this skill when writing, fixing, or reviewing Bruno API tests (.bru files) — collection layout, environments and secrets, assert/tests blocks, scripting, and bru run in CI.
metadata:
  author: Greg Duckworth
  version: 1.1.0
  tags:
    - category/api-testing
    - domain/engineering
    - domain/testing
    - tool/bruno
  compatibility:
    - all
  frameworks:
    - bruno
---

Bruno-specific rules. Bruno stores collections as plain-text `.bru` files in the repo, so tests are reviewed and versioned like code.

## Requires

- `test-api` (which requires `test-automation-guidelines`) — its API rules and the base Guardrails apply. If either is not available, tell the user which plugin to install.

## Rules

### Files and structure

- Keep `.bru` files in version control and review them like code.
- One request per `.bru` file, named for the behaviour it verifies; folders by resource or feature.

### Variables and environments

- Reference config as variables (`{{baseUrl}}`), defined per environment under `environments/`.
- Credentials via Bruno secret variables or `{{process.env.VAR}}`; commit only an example env with placeholders.

### Assertions

- Declarative `assert` block for status and simple field checks; the `tests` block with `expect` for schema and richer logic.

### Scripting

- `pre-request` scripts for tokens and setup data; `post-response` scripts for extraction and cleanup.
- Capture only the values later requests need (`bru.setVar("orderId", res.body.id)`).

### Running

- `bru run <folder> --env <name>` in CI, with the environment chosen explicitly.

## Anti-Patterns

- Committed `.bru` or environment files containing real tokens or keys
- Hardcoded full URLs instead of `{{baseUrl}}`
- `tests` scripts re-implementing what `assert` does declaratively
- Relying on collection-runner order for tests that should be independent

## References

Load only the one you need:

- [references/collections.md](references/collections.md) — collection and environment structure.
- [references/scripting-and-assertions.md](references/scripting-and-assertions.md) — assertions, scripting, and `bru run`.
