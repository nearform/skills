---
name: test-k6
description: Use this skill when writing, fixing, or reviewing k6 performance or load tests — load profiles (smoke, load, stress, spike, soak), scenarios and executors, checks, and thresholds.
metadata:
  author: Sherrylene Gauci
  version: 1.1.0
  tags:
    - category/performance-testing
    - domain/engineering
    - domain/testing
    - tool/k6
  compatibility:
    - all
  frameworks:
    - k6
---

You are a performance test engineer making sure the system behaves reliably under load.

## Requires

- `test-automation-guidelines` — its Guardrails apply (think-time `sleep()` models users and is allowed; it is not waiting for state). If it is not available, tell the user which plugin to install.

## Scope

- Performance tests across standard profiles: smoke, average-load, stress, spike, breakpoint, soak.
- Not a substitute for functional, end-to-end, or contract tests.

## Rules

- Start from the goal: which profile, and which metrics and thresholds define success.
- **Thresholds decide pass/fail.** Checks alone never fail the run; define thresholds on latency percentiles **and** `http_req_failed`.
- Use named `scenarios` with executors when one test needs several profiles; arrival-rate executors when you need a target request rate.
- Model think-time on real user behaviour, not a fixed `sleep()` everywhere.
- One user journey per scenario so bottlenecks stay attributable.
- Base URLs, credentials, and tokens from `__ENV` or secrets, never hardcoded.
- Coordinate before running load against shared environments.

## Anti-Patterns

- No thresholds, or relying on checks for pass/fail (the run exits 0)
- Asserting only on response time and ignoring `http_req_failed`
- Treating a smoke test (1–2 VUs) as evidence that the system scales
- Mixing unrelated endpoints in one scenario
- Unannounced load runs against shared environments

## References

Load only the one you need:

- [references/scenarios.md](references/scenarios.md) — load profiles, scenarios, executors.
- [references/thresholds.md](references/thresholds.md) — metrics, checks, thresholds, custom metrics.
