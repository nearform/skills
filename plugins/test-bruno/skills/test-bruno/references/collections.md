# Bruno Collections and Environments

Apply this guidance when structuring a Bruno collection.

## Collection layout

- A Bruno collection is a folder containing `bruno.json` plus `.bru` request files, organised into subfolders by resource or feature.
- Keep the whole collection in the repository so requests are diffed and reviewed like source code.
- Name folders and requests for behaviour, not HTTP verb (e.g. `orders/create order returns 201.bru`).

## `.bru` request anatomy

A request file separates concerns into named blocks:

```
meta {
  name: create order returns 201
  type: http
}

post {
  url: {{baseUrl}}/orders
  body: json
}

headers {
  Content-Type: application/json
}

body:json {
  {
    "sku": "{{sku}}",
    "quantity": 1
  }
}

assert {
  res.status: eq 201
  res.body.id: isDefined
}
```

## Environments

- Define environments under `environments/` (e.g. `local.bru`, `staging.bru`) with the same variable names, different values.
- Reference config via variables (`{{baseUrl}}`, `{{apiVersion}}`) so the same requests run across environments.
- Store secrets as secret variables or via `{{process.env.TOKEN}}`; commit only an example environment with placeholders.

## Version control

- Commit `.bru` files and a placeholder environment; `.gitignore` any file holding real secrets.
- Review request and assertion changes in pull requests like any other code change.

## Anti-patterns

- Treating the collection as an exported artifact that is regenerated instead of edited.
- Environment files with real credentials committed to the repo.
- Deeply nested folders that obscure which behaviour a request covers.
