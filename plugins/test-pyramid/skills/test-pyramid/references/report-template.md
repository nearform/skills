# Test pyramid report — <system name>

## 1. Scope & method

| Repository | Suites | Frameworks | Read in full / sampled |
|---|---|---|---|
| `web-app` | unit, component | Vitest | full |
| `qa-e2e` | e2e | Playwright | full |
| `api` | unit, integration | Jest + supertest | sampled (40 of 312 files) |

- **Evidence used:** e.g. JUnit XML from CI run #1234 (timings), lcov from `web-app`.
- **Excluded / unreachable:** e.g. `mobile-tests` (no access) — the pyramid below omits it.
- **Behaviour source:** e.g. API routes + UI journeys.

## 2. Pyramid

| Layer | Tests | Share | CI runtime | Notes |
|---|---|---|---|---|
| E2E | 420 | 28% | 38 min | 31 quarantined as flaky |
| Integration / API | 160 | 11% | 6 min | |
| Contract | 0 | 0% | — | |
| Component | 90 | 6% | 1 min | |
| Unit | 830 | 55% | 2 min | |

**Shape:** ice-cream cone at the top — E2E holds 28% of tests but 81% of runtime.

**Beside the pyramid:** k6 load tests (4 scenarios), axe accessibility checks in 12 Playwright specs.

## 3. Coverage matrix (top behaviours)

| Behaviour | Unit | Component | Contract | Integration / API | E2E |
|---|---|---|---|---|---|
| Checkout rejects expired card | `api:src/card.test.ts:41` | — | — | `api:test/checkout.int.ts:88` | `qa-e2e:checkout.spec.ts:120`, `:144`, `:171` |
| Password reset email sent | — | — | — | — | `qa-e2e:auth.spec.ts:60` |

## 4. Duplication

| Behaviour | Tests | Type | Recommendation |
|---|---|---|---|
| Checkout rejects expired card | Unit `api:src/card.test.ts:41`; Integration `api:test/checkout.int.ts:88`; E2E `qa-e2e:checkout.spec.ts:120`, `:144`, `:171` | Redundant | Keep unit + integration; remove E2E `:144` and `:171` (proven by `api:src/card.test.ts:41`); keep `:120` as the happy-path checkout |
| Login form renders error | Component `web-app:src/LoginForm.test.tsx:33`; E2E `qa-e2e:auth.spec.ts:22` | Complementary | Keep both — E2E proves server error is wired to UI |

## 5. Push-down candidates

| Test(s) | From → To | What stays | Expected saving |
|---|---|---|---|
| `qa-e2e:pricing.spec.ts` (24 tests, discount maths) | E2E → Unit | 1 E2E: discount applied at checkout | ~6 min CI, removes 3 flaky tests |
| `qa-e2e:orders.spec.ts:12-90` (UI only to reach `POST /orders`) | E2E → API | none | ~2 min (estimate) |

## 6. Gaps

- **Untested:** refund partial amounts (`api/src/refunds.ts`).
- **E2E-only:** password reset — add an API test for token expiry.
- **Thin high-risk:** auth role checks — 2 tests for 14 protected routes.

## 7. Prioritised recommendations

1. Remove 23 redundant pricing E2E tests (keep 1 checkout path) after confirming unit coverage (`test-refactor`, `test-unit-guidelines` + `test-vitest`) — quick win, ~6 min.
2. Add provider contract tests between `web-app` and `api` (`test-pact`) — replaces 18 integration shape checks.
3. Cover refund partial amounts at unit + API level (`test-unit-guidelines`, `test-api`).

_All runtimes from CI run #1234 unless marked estimate. Items marked UNVERIFIED were not checked._
