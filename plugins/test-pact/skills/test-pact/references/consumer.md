# Consumer Tests Guidance

Apply this guidance when writing, reviewing or maintaining Pact consumer tests.

## Scope

- Focus on how the consumer builds requests and processes provider responses
- Validate that the consumer can work with the provider's response structure, not the provider's internal behaviour

## Test design

- Keep each interaction focused on one API behaviour
- Add one interaction per test
- Use `given`, `uponReceiving`, and `willRespondWith` in BDD style language
- Specify the required state using the `given` method to tell the provider which data scenario to set up
- Use meaningful and unique test descriptions
- Use a consistent test template

```javascript
await pact
  .addInteraction()
  .given("a product with ID exists", { id: "123" })
  .uponReceiving("a request for product with ID 123")
  .withRequest({
    method: "GET",
    path: "/products/123",
  })
  .willRespondWith({
    status: 200,
    headers: { "Content-Type": "application/json" },
    body: {
      id: like("123"),
      name: like("Widget"),
    },
  });

await pact.executeTest(async (mockserver) => {
  const client = new ProductApiClient(mockserver.url);
  const response = await client.getProduct("123");
  expect(response.id).toBeDefined();
});
```

## Matchers strategy

- For the response, loose matchers are the preferred option
- Exact/loose matchers should be made on a field-by-field basis
- Consumer should mostly care about the type rather than the content of the field

Preferred:

```javascript
eachLike({ id: like("123"), name: like("Widget") });
```

Avoid:

```javascript
{ id: 123, name: "Widget", inStock: true }
```

## Assertions

- Assert only fields that can break the consumer if changed
- Avoid unrelated assertions (for example logging)

## Best practices

- Test the actual client API
- Do not bypass consumer logic with raw `fetch`/`axios` API calls
- Ensure consumer-generated pact files are deterministic i.e. avoid dynamic data
- Do not include sensitive data in tests (for example tokens, passwords)
