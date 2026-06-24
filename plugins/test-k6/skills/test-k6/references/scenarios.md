# Scenarios and Load Profiles Guidance

Apply this guidance when designing the shape of a k6 performance test: how many virtual users (VUs), for how long, and which executor to use.

## Scope

- Focus on how load is generated over time, not on what is being measured.
- Choose the load profile that matches the performance test you want to run.

## Test types

| Type | Question it answers | Typical shape |
|---|---|---|
| Smoke | Does the script work and the system respond at minimal load? | 1–2 VUs, short |
| Average-load | How does the system behave at expected/average load? | Ramp to target VUs, hold, ramp down |
| Stress | Where does the system start to degrade? | Ramp beyond expected load in steps |
| Spike | How does the system handle a sudden surge? | Sharp ramp up, brief hold, sharp ramp down |
| Breakpoint | At what point does the system break? | Ramp up with no plateau until thresholds fail |
| Soak | Are there leaks or degradation over time? | Moderate load held for a long duration |

## Structuring a test

- Keep each scenario focused on a single user journey or endpoint group.
- Use `scenarios` with named executors when you need different profiles in one test.
- Prefer `ramping-vus` or `ramping-arrival-rate` over a fixed VU count so load builds and tapers gradually, which models real traffic more accurately.
- Use arrival-rate executors (`constant-arrival-rate`, `ramping-arrival-rate`) when you need a target request rate independent of system response time.

### Stages-based load test

```javascript
import http from "k6/http";
import { sleep } from "k6";

export const options = {
  stages: [
    { duration: "2m", target: 100 }, // ramp up
    { duration: "5m", target: 100 }, // steady state
    { duration: "2m", target: 0 },   // ramp down
  ],
};

export default function () {
  http.get(`${__ENV.BASE_URL}/products`);
  sleep(1); // think time
}
```

### Multiple scenarios with executors

```javascript
export const options = {
  scenarios: {
    average_load: {
      executor: "ramping-vus",
      exec: "browse", // run the `browse` function, not `default`
      startVUs: 0,
      stages: [
        { duration: "2m", target: 100 },
        { duration: "5m", target: 100 },
        { duration: "2m", target: 0 },
      ],
    },
    spike: {
      executor: "ramping-vus",
      exec: "checkout", // a different user journey
      startVUs: 0,
      startTime: "9m", // run after the average_load scenario
      stages: [
        { duration: "30s", target: 500 },
        { duration: "1m", target: 500 },
        { duration: "30s", target: 0 },
      ],
    },
  },
};

export function browse() {
  http.get(`${__ENV.BASE_URL}/products`);
  sleep(1);
}

export function checkout() {
  http.post(`${__ENV.BASE_URL}/checkout`);
  sleep(1);
}
```

## Best practices

- Use environment variables for environment parameters, for example `__ENV.BASE_URL` - never hardcoded values.
- Model realistic think time with `sleep()` based on expected user behaviour, not arbitrary values.
- Use `group()` to label logical steps of a journey so results are easy to read and trace.
- Keep setup/teardown logic in `setup()` and `teardown()` rather than inside the iteration.
- Tag requests so dynamic URLs are aggregated correctly in metrics. Use the `http.url` tagged template for path params, or an explicit `name` tag otherwise:

```javascript
http.get(http.url`${__ENV.BASE_URL}/users/${userId}`);          // aggregated as /users/${}
http.get(`${__ENV.BASE_URL}/products/${id}`, { tags: { name: "GET /products/:id" } });
```

- Set `gracefulStop` (or `gracefulRampDown` for `ramping-vus`) deliberately. These control how long in progress iterations are allowed to finish before being cut off at the end of a stage.
