# Metrics, Checks, and Thresholds Guidance

Apply this guidance when defining how a k6 test passes or fails and what it measures.

## Scope

- Focus on validating system behaviour, not on shaping load.
- Every performance test should have explicit pass/fail criteria.

## Checks vs. thresholds
- **Checks** validate individual responses (status code, body content). A failing check does not fail the test on its own.
- **Thresholds** define pass/fail criteria for the whole test. A breached threshold makes k6 exit with a non-zero code.
- Use checks to find *what* went wrong, and thresholds to decide *whether the test passed*.

### Checks

```javascript
import http from "k6/http";
import { check } from "k6";

export default function () {
  const res = http.get(`${__ENV.BASE_URL}/products`);
  check(res, {
    "status is 200": (r) => r.status === 200,
    "response under 500ms": (r) => r.timings.duration < 500,
  });
}
```

### Thresholds

```javascript
export const options = {
  thresholds: {
    http_req_failed: ["rate<0.01"],          // error rate under 1%
    http_req_duration: ["p(95)<500", "p(99)<1000"],
    checks: ["rate>0.99"],                    // 99% of checks must pass
  },
};
```

## Key built-in metrics

- `http_req_duration` — total request time.
- `http_req_failed` — request error rate; always include a threshold on this.
- `http_reqs` — total requests and throughput (requests/second).
- `vus` / `vus_max` — concurrency during the test.
- `iteration_duration` — full iteration time including think time.

## Custom metrics

Use custom metrics for business-level SLAs that the built-in metrics do not capture.

```javascript
import http from "k6/http";
import { Trend, Counter } from "k6/metrics";

const checkoutDuration = new Trend("checkout_duration", true); // true: A boolean indicating whether the values added to the metric are time values or just untyped values.
const failedCheckouts = new Counter("failed_checkouts");

export const options = {
  thresholds: {
    checkout_duration: ["p(95)<800"],
    failed_checkouts: ["count<10"],
  },
};

export default function () {
  const res = http.post(`${__ENV.BASE_URL}/checkout`);
  checkoutDuration.add(res.timings.duration); // record the duration metric
  if (res.status !== 200) failedCheckouts.add(1); // count a failure
}
```

A custom metric only contributes to its threshold if you `.add()` to it during the test. Declaring the metric and setting a threshold without recording any values leaves it empty, and the threshold passes against no data.

## Best practices

- Assert on percentiles rather than averages
- Always set a threshold on `http_req_failed`; response time alone is not enough.
- Set thresholds from agreed SLO/SLAs
- Use `abortOnFail` on critical thresholds to stop a hanging load when a test is already failing.
