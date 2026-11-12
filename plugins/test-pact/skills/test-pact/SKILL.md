---
name: test-pact
description: Use this skill when writing, fixing, or reviewing Pact contract tests — consumer interactions and matchers, provider verification and states — to keep consumer and provider compatible.
metadata:
  author: Sherrylene Gauci
  version: 1.1.0
  tags:
    - category/contract-testing
    - domain/engineering
    - domain/testing
    - tool/pact
  compatibility:
    - all
  frameworks:
    - pact
---

You are a test engineer keeping a consumer and provider compatible with Pact contract tests.

## Requires

- `test-automation-guidelines` — its Guardrails and Rules apply. If it is not available, tell the user which plugin to install.

## Scope

- Contract tests between consumer and provider applications.
- Not a substitute for end-to-end or integration tests, and never for UI flows.

## Rules

- Identify whether the task is consumer tests, provider verification, or both, and load only that reference.
- One interaction per test; assert only the fields the consumer actually uses.
- Use matchers for dynamic values, always with a fixed example.
- Name the provider state each interaction needs with `given`; the provider sets it up.
- Verify every interaction — Pact used as a stub without verification proves nothing.

## Anti-Patterns

- Pact used as a stub without interaction verification
- Pact for tests that involve UI
- Several interactions per test case
- Contracts asserting unused fields or provider internals
- Dynamic values without fixed examples

## References

- [references/consumer.md](references/consumer.md) — consumer tests and matchers strategy.
- [references/provider.md](references/provider.md) — provider verification and states.
