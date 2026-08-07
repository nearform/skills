# Request Design Guidance

Apply this guidance when structuring API requests in tests.

## Structure

- Give each request a clear, behaviour-describing name (e.g. `create order returns 201`, `get order with unknown id returns 404`).
- Group related requests into collections/folders by resource or feature, not by HTTP verb.
- Keep the base URL, path, headers, query params, and body as distinct, readable parts — do not concatenate a full URL string with embedded variables.

## Parameterisation

- Resolve the base URL from an environment variable (e.g. `{{baseUrl}}`), never a literal.
- Use variables for path segments that change between environments or runs (IDs, tenant, version).
- Keep request bodies in a readable, formatted shape; prefer fixtures/example files for large payloads.

## Headers and content

- Set `Content-Type` and `Accept` explicitly where the API requires them.
- Send auth via headers/tokens resolved from the environment, never inline literals (see `environments-and-data.md`).
- Only send the headers the test needs; avoid copying a full browser header set.

## Ordering and dependencies

- Prefer independent requests. When a test genuinely needs setup (e.g. create then fetch), make the dependency explicit and set up the precondition within the same test flow.
- Capture only the specific values a later step needs (e.g. the created `id`) and pass them explicitly, rather than relying on shared global state that other tests can mutate.

## Timeouts

- Set an explicit per-request timeout so a hung endpoint fails fast instead of stalling the whole suite.
- Choose a timeout that reflects the endpoint's expected latency plus headroom, not an arbitrarily large value.
- Treat a timeout as a test failure with a clear message, not a silent retry.

## Anti-patterns

- Hardcoded full URLs like `https://staging.example.com/api/v1/orders`.
- Requests named `Request 1`, `Copy of ...`, or by verb only (`GET`).
- Large duplicated bodies pasted across many requests instead of a shared fixture/variable.
