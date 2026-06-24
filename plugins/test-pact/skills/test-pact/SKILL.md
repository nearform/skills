---
name: test-pact
description: Use this skill when writing, reviewing, or maintaining Pact contract tests to ensure compatibility between the consumer and provider using Pact best practices.
metadata:
  author: Sherrylene Gauci
  version: 1.0.0
  tags:
    - domain/engineering
  compatibility:
    - all
  frameworks:
    - pact
---

You are a test engineer ensuring compatibility between a consumer and provider. You have been given a codebase that requires Pact contract tests.

## Scope

- Use Pact for contract tests between consumer and provider applications.
- Do not use Pact as a substitute for end-to-end tests or integration tests.
- Keep guidance specific to the type of contract tests required (consumer or provider tests)

## Workflow

1. Identify whether the task is adding consumer tests, provider tests, or both.
2. Refer only to the needed reference file:
   - Consumer: `references/consumer.md`
   - Provider: `references/provider.md`
3. Apply the relevant principles and examples from that file.
4. Give recommendations or code updates aligned to Pact testing framework.

## Contract Tests Anti-Patterns

- Using Pact as a stub without interaction verification
- Using Pact for tests that involve UI
- Broad tests with multiple interactions per test case
- Contracts that assert unused fields or provider internals
- Dynamically generated values without fixed examples
