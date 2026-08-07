---
name: test-bruno
description: Use this skill when writing or reviewing Bruno API tests (.bru files) to apply Bruno-specific best practices for reliable, version-controlled, idiomatic tests.
metadata:
  author: Greg Duckworth
  version: 1.0.0
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

Use this skill when working with Bruno API tests. Bruno stores collections as plain-text `.bru` files in the repo, so tests are reviewed and versioned like code.

## Requires

- test-automation-guidelines
- test-api

Apply the shared API principles from `test-api` first; the rules below are Bruno-specific additions.

## Workflow

1. Confirm the collection layout and environments (`bruno.json`, `environments/`).
2. Refer only to the reference file relevant to the task:
   - Collection and environment structure: `references/collections.md`
   - Assertions, scripting, and running: `references/scripting-and-assertions.md`
3. Apply the principles and templates from that file.

## Bruno Rules

### Files and structure

- Keep `.bru` files in version control and review them like code — never treat the collection as a throwaway export.
- One request per `.bru` file, named for the behaviour it verifies.
- Organise requests into folders by resource/feature (folders are plain directories on disk).

### Variables and environments

- Reference the base URL and other config as variables (`{{baseUrl}}`), defined per environment under `environments/`.
- Store secrets as secret variables (`.env` / secret vars), never committed literals. Commit an example env with placeholders only.
- Use `{{process.env.VAR}}` or Bruno secret vars for credentials so they stay out of the committed `.bru`.

### Assertions

- Prefer the declarative `assert` block for status and simple field checks.
- Use the `tests` block with `expect` for schema/shape and richer logic.
- Assert status and key fields; avoid asserting volatile values (see `test-api` assertions guidance).

### Scripting

- Use `pre-request` scripts to obtain tokens or set up data, and `post-response` scripts for extraction/cleanup.
- Capture only the specific values later requests need (e.g. `bru.setVar('orderId', res.body.id)`); avoid leaking broad state.
- Keep scripts small and deterministic; do not add arbitrary waits.

## Anti-Patterns

- Committing `.bru` files or environment files that contain real tokens or API keys.
- Hardcoded full URLs instead of `{{baseUrl}}`.
- Bloated `tests` scripts re-implementing assertions that `assert` handles declaratively.
- One `.bru` file exercising many endpoints.
- Relying on collection-runner order for tests that should be independent.
