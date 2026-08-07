# Environments, Auth, and Test Data

Apply this guidance when handling environments, authentication, and test data in API tests.

## Environments

- Define one environment per target (local, dev, staging) with the same variable names and different values.
- Reference values through variables (`{{baseUrl}}`, `{{apiVersion}}`) so a suite runs unchanged across environments.
- Never commit environment files that contain real secrets. Commit a template/example with placeholder values and keep real values in local/CI secrets.

## Authentication

- Resolve tokens and API keys from environment/secret variables, never inline literals in requests.
- Obtain short-lived tokens at run time (e.g. a pre-request auth step) rather than pasting a static token that will expire or leak.
- Store credentials as secret-typed variables where the tool supports it, so they are masked in logs and exports.
- Never print full tokens in test output or assertions.

## Test data

- Create the data a test needs as part of its own setup, and remove it in teardown.
- Prefer API calls for setup/teardown over relying on pre-seeded or manually created records.
- Use unique, generated identifiers (e.g. a random suffix) to avoid collisions when tests run in parallel or repeatedly.
- Do not depend on data created by a previous run; each run must be able to start from a clean baseline.

## Secrets handling

- Keep secrets out of the repository: use `.gitignore`d local files or the CI secret store.
- Rotate any credential that is accidentally committed and scrub it from history.

## Anti-patterns

- A committed environment file containing a real bearer token or API key.
- A single shared long-lived token pasted into many requests.
- Tests that only pass because a specific record already exists in the target environment.
- Leaving created test data behind, so re-runs fail on duplicates.
