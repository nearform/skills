---
name: test-api
description: Use this skill when writing, reviewing, or maintaining automated tests for HTTP/REST APIs to keep them deterministic, isolated, and behaviour-focused. Acts as the shared base for tool-specific API testing skills (Bruno, Postman).
metadata:
  author: Greg Duckworth
  version: 1.0.0
  tags:
    - category/api-testing
    - domain/engineering
    - domain/testing
  compatibility:
    - all
  frameworks:
    - api
    - rest
    - http
---

You are a test engineer writing automated tests against HTTP/REST APIs. This skill defines tool-agnostic principles for API testing. Tool-specific skills (`test-bruno`, `test-postman`) build on top of it.

## Requires

- test-automation-guidelines

## Scope

- Cover request/response behaviour at the API boundary: status codes, headers, body schema, and error handling.
- Do not use API tests as a substitute for unit tests (business logic) or end-to-end tests (full user journeys).
- Keep each test focused on one API behaviour or contract.

## Workflow

1. Identify the endpoint under test and the behaviour to verify (happy path, validation error, auth failure, edge case).
2. Refer only to the reference file relevant to the task:
   - Request design and structure: `references/request-design.md`
   - Assertions and response validation: `references/assertions.md`
   - Environments, auth, and test data: `references/environments-and-data.md`
3. Apply the principles and templates from that file.
4. Give recommendations or test code aligned with these principles and the specific tool in use.

## Core Principles

- **One behaviour per test** — each test asserts a single, named API behaviour.
- **Deterministic** — never depend on ambient state, execution order, or wall-clock timing; wait on real signals (response received), not fixed delays.
- **Isolated** — create the data a test needs and clean it up; do not rely on data left by other tests or manual setup.
- **Behaviour, not implementation** — assert the observable contract (status, headers, body shape/values), not internal implementation details.
- **Explicit environments** — never hardcode base URLs, credentials, or secrets in requests; resolve them from environment/config.
- **Meaningful assertions** — assert status, key response fields, and schema; avoid asserting volatile or unused fields.

## Rules

- Do not hardcode secrets, tokens, or environment URLs in requests or committed files.
- Do not chain tests through shared mutable state; pass only explicit, documented data between dependent steps.
- Do not assert on volatile values (timestamps, generated IDs) with exact matches — assert their presence, type, or format instead.
- Do not use arbitrary sleeps to "wait" for the API; assert on the response, and for async endpoints poll with a bounded timeout.
- Set an explicit request timeout so a hung endpoint fails fast instead of stalling the suite.
- Validate both success and failure paths (e.g. 4xx validation and auth errors), not only the happy path.

## Anti-Patterns

- Tests that pass only when run in a specific order.
- Environment URLs, API keys, or bearer tokens committed in request files.
- One giant test that exercises many endpoints and assertions.
- Asserting the entire response body verbatim, including volatile fields.
- Relying on data created by a previous test run rather than per-test setup.

## Usage

This skill defines shared API testing principles. Apply it together with a tool-specific skill (`test-bruno` or `test-postman`) and with `test-automation-guidelines`.
