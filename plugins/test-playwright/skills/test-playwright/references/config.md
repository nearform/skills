# Playwright: config, timeouts, and test servers

Load when editing `playwright.config.*`, tuning timeouts, or configuring `webServer`. Defaults below are from the Playwright docs at the time of writing; confirm against the project's version.

## Timeouts

- Web-first assertions and `expect.poll` give up after `expect.timeout` (5 seconds by default). Pass a `timeout` that covers waits on server work, or raise `expect.timeout` in the config.
- The test `timeout` (30 seconds by default) covers the whole test, including fixture setup. Waits that run one after another must fit inside it together.
- Poll for every final state, then assert the one you expect, so a failure reports itself instead of timing out:

```javascript
test("order becomes ready", async () => {
  test.setTimeout(90_000);
  await expect
    .poll(() => getStatus(id), { timeout: 60_000 })
    .toMatch(/^(ready|failed)$/);
  expect(await getStatus(id)).toBe("ready");
});
```

## Parallelism

- Tests must be parallel-safe. When tests share a server with a concurrency limit, set `workers` no higher than that limit.

## Test servers (`webServer`)

- `webServer.env` is laid over `process.env` and replaces any variable it names. Add to `NODE_OPTIONS` instead of overwriting it:

```javascript
env: {
  NODE_OPTIONS: `${process.env.NODE_OPTIONS ?? ""} --conditions=source`.trim(),
},
```

- Take ports from environment variables with defaults, so a second copy of the suite or a developer's own server doesn't collide.
- Playwright stops `webServer` processes with SIGKILL unless `gracefulShutdown` is set. Set it (e.g. `{ signal: "SIGTERM", timeout: 5000 }`) when the server cleans up on a signal.

## Debugging

- Record traces (`trace: "on-first-retry"`) and video in CI for triage.
