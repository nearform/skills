---
name: test-api
description: Use this skill when writing, fixing, or reviewing automated HTTP/REST API test code — status, headers, schema, auth, and test data. The shared base for test-bruno and test-postman.
metadata:
  author: Greg Duckworth
  version: 1.1.0
  tags:
    - category/api-testing
    - domain/engineering
    - domain/testing
  compatibility:
    - all
---

You are a test engineer writing automated tests against HTTP/REST APIs. This skill holds the tool-agnostic API rules; `test-bruno` and `test-postman` add tool specifics on top.

## Requires

- `test-automation-guidelines` — its Guardrails (including no secrets, no fixed sleeps, independence) apply. If it is not available, tell the user which plugin to install.

## Scope

- Request/response behaviour at the API boundary: status codes, headers, body schema, error handling.
- Not a substitute for unit tests (business logic), contract tests (`test-pact`), or E2E journeys.

## Rules

- **One API behaviour per test**, named for what it verifies.
- **Assert the contract:** status, key fields, and schema. Don't assert the whole body verbatim or fields the consumer doesn't use.
- **Volatile values** (timestamps, generated IDs): assert presence, type, or format, never exact values.
- **Cover failure paths**, not only the happy path: 4xx validation, 401/403 auth, 404, and conflict cases.
- **Each test sets up and cleans up its own data.** Where steps must chain, pass only explicit, documented values (e.g. a created ID).
- **Async endpoints:** poll with a bounded timeout and stop on failure states; set an explicit request timeout so a hung endpoint fails fast.
- **Environment-driven config:** base URLs and credentials come from environment variables or the tool's secret store.

## Anti-Patterns

- One giant test exercising many endpoints
- Asserting the entire response body, including volatile fields
- Relying on data left by a previous run or a manual setup step
- Tests that only pass in collection/runner order

## References

Load only the one you need:

- [references/request-design.md](references/request-design.md) — request structure and naming.
- [references/assertions.md](references/assertions.md) — response validation and schema checks.
- [references/environments-and-data.md](references/environments-and-data.md) — environments, auth, and test data.
