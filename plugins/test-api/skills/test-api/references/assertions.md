# Assertions and Response Validation

Apply this guidance when asserting on API responses.

## What to assert

- **Status code** — assert the exact expected status (`200`, `201`, `400`, `401`, `404`, ...).
- **Key body fields** — assert the values that define the behaviour under test, not the whole payload.
- **Schema/shape** — assert the response matches the expected structure (types and required fields), especially for lists and nested objects.
- **Headers** — assert headers that carry contract meaning (e.g. `Content-Type`, `Location`, pagination, rate-limit) when relevant.

## Volatile values

- Do not assert exact matches on timestamps, generated IDs, or ordering that the API does not guarantee.
- Instead assert presence, type, or format (e.g. `id` exists and is a UUID; `createdAt` is an ISO-8601 string).

## Error paths

- For every meaningful success case, cover the corresponding failure cases: validation errors (`4xx`), auth failures (`401`/`403`), and not-found (`404`).
- Assert the error status and the stable parts of the error body (e.g. an error `code`), not free-text messages that may change.

## Asynchronous responses

- For endpoints that accept work and complete it later (e.g. `202 Accepted` plus a status/location URL), assert the initial acknowledgement, then poll the status resource for the terminal state.
- Poll with a bounded number of attempts and a maximum overall timeout; never use a single fixed sleep to "wait long enough".
- Assert the final terminal state (e.g. `completed` / `failed`) and fail clearly if the timeout is reached before it settles.

## Schema validation

- Prefer validating against a JSON Schema (or the tool's schema/assertion helpers) over a long list of individual field assertions.
- Keep schemas versioned alongside the tests so contract changes are visible in review.

## Anti-patterns

- Deep-equality asserting the entire response body including volatile fields.
- Asserting only `status === 200` with no body/schema checks.
- Matching on human-readable error text that is not part of the contract.
- Ignoring failure paths and testing only the happy path.
