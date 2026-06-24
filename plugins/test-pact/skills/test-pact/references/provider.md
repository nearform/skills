# Provider Tests Guidance

Apply this guidance when writing, reviewing or maintaining Pact provider verification tests.

## Test Design

- Verify pact artifacts against a running local instance of the provider

```javascript
const opts = {
  providerBaseUrl: "http://localhost:8080",
  pactUrls: ["./pacts/consumer-product-service.json"],
};

await new Verifier(opts).verifyProvider();
```

## Provider states

- Use provider states to prepare provider-side data for specific test cases

```javascript
const opts = {
  // ...
  stateHandlers: {
    "a product with ID exists": async (parameters) => {
      await productRepository.insert({ id: parameters.id, name: "Product 123" });
    },
  },
};
```

## Best Practices

- Stub external calls to downstream systems
