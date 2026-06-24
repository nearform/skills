---
name: test-k6
description: Use this skill when writing, reviewing, or maintaining k6 performance tests to validate system behaviour under load using k6 best practices.
metadata:
  author: Sherrylene Gauci
  version: 1.0.0
  tags:
    - domain/engineering
  compatibility:
    - all
  frameworks:
    - k6
---

You are a performance test engineer ensuring that the system behaves reliably under load. You have been given a codebase that requires k6 performance tests.

## Scope

- Use k6 for performance tests across the standard profiles (smoke, average-load, stress, spike, breakpoint, soak — see `references/scenarios.md`).
- Do not use k6 as a substitute for functional, end-to-end, or contract tests.
- Keep test profile guidance specific to the type of performance test required.

## Workflow

1. Identify the goal of the test: which load profile is needed (see `references/scenarios.md`) and which metrics or thresholds define success.
2. Refer only to the needed reference file:
   - Scenarios and load profiles: `references/scenarios.md`
   - Metrics, checks, and thresholds: `references/thresholds.md`
3. Apply the relevant principles and examples from that file.
4. Give recommendations or code updates aligned to k6 best practices.

## Performance Test Anti-Patterns

- Running tests without defined thresholds
- Relying on checks for pass/fail — a test with passing checks but no thresholds always exits 0, giving false confidence
- Using `sleep()` as the only think-time strategy with no relation to real user behaviour
- Hardcoding environment URLs, credentials, or tokens instead of using environment variables and/or secrets
- Mixing unrelated endpoints into a single scenario, making bottlenecks hard to debug
- Treating a smoke test (1–2 VUs) as evidence that the system scales successfully
- Ignoring `http_req_failed` and asserting only on response time
- Running load tests against shared environments without coordinating with other teams
