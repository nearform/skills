# Postman Collections and Environments

Apply this guidance when structuring Postman collections and environments.

## Collection layout

- A collection is JSON containing folders and requests. Organise folders by resource or feature.
- Name each request for the behaviour it verifies (e.g. `create order returns 201`), not by verb.
- Export the collection to the repository (`collection.json`) so requests and tests are diffed and reviewed like code.

## Variables and scoping

Postman resolves variables by scope, narrowest wins. Use each scope intentionally:

- **Environment** — per-target config (`baseUrl`, `apiVersion`), swapped between local/dev/staging.
- **Collection** — defaults shared across all requests in the collection.
- **Local (`pm.variables`)** — request-run temporary values; do not persist.
- **Globals** — avoid; they are workspace-wide and hard to reason about.

Reference variables as `{{baseUrl}}` in URLs, headers, and bodies.

## Environments and secrets

- Define one environment per target with the same variable keys, different values.
- Mark credentials as **secret**-typed variables so they are masked; prefer Postman Vault for local secrets.
- Never store real tokens in the environment's _initial_ values (those export to JSON). Keep real values in _current_ values or CI secrets only.
- Commit an example environment with placeholder values; `.gitignore` anything holding real secrets.

## Version control

- Export collection and environment JSON and commit them; review changes in pull requests.
- Minimise noisy diffs: avoid reordering requests in the UI purely for cosmetics.

## Anti-patterns

- Real tokens saved as environment initial values and exported to the repo.
- Overusing globals for values that belong to an environment or collection.
- Keeping the authoritative tests only in a personal workspace with no export.
- Folder structure organised by HTTP verb rather than resource/feature.
