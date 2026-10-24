# K6 lab results

## Part 0

- Step 0.1, http_req_duration: 260.21ms
- Step 0.2, requests, rate and p(95) and why the rate stays below 5: 80 requests, rate 3.78 req/s, p(95) = 278.22 ms. The request rate stays below 5 req/s because each iteration takes ~1.27s - 1.31s (HTTP request duration ~0.27s + sleep(1s)). Each VU completes ~0.76 iterations per second, so 5 VUs yield a total rate of ~3.78 req/s.
- Step 0.6, exit code of the normal run and of the run with P95=300: Normal run exit code = 0, P95=300 run exit code = 99
- Step 0.7, rules that passed, p(95), p(99), exit code after your change: All rules passed (p(95) = 381.81ms, p(99) = 386.54ms). Exit code after changing requirement (changed 'p(95)<1000' to 'p(95)<100') = 99
- Step 0.8, what was wrong, and how the checks showed it: The original script used lowercase 'pepperoni' and 'knife' instead of capitalized 'Pepperoni' and 'Knife', causing the API to ignore exclusions. A single manual request might accidentally return a compliant pizza by chance due to random recommendation logic, but testing under load with multiple iterations consistently exposes the failure rate when exclusions fail to apply.

## Part A results

| Step | Test type    | Users | Req/s | p(95)    | Max      | Failed | Checks  |
| ---- | ------------ | ----- | ----- | -------- | -------- | ------ | ------- |
| A1   | Smoke        | 1     | 0.54  | 335.91ms | 346.67ms | 0.00%  | 100.00% |
| A2   | Average load | 3     | 1.33  | 371.77ms | 387.61ms | 0.00%  | 100.00% |
| A3   | Stress       | 5     | 2.17  | 373.24ms | 396.90ms | 0.00%  | 100.00% |
| A4   | Spike        | 5     | 1.04  | 379.31ms | 393.02ms | 0.00%  | 100.00% |
| A5   | Soak         | 2     | 0.99  | 377.07ms | 393.96ms | 0.00%  | 100.00% |

## Part B

1. Filled in the Part A results table above.
2. In Step 0.2, each iteration took around 1.27s - 1.31s due to HTTP request latency plus the explicit `sleep(1)`. Therefore, 1 VU executes ~0.76 req/s, resulting in a total rate of ~3.78 req/s across 5 VUs.
3. The response time remained virtually identical between average load (p(95) = 371.77ms) and stress test (p(95) = 373.24ms). This indicates that 5 virtual users are still well inside QuickPizza's operational envelope.
4. In the spike report, the response time did not rise noticeably when the load jumped from 1 to 5 users (p(95) over the whole run was 379 ms, close to the 336 ms of the smoke test). So the recovery time was effectively 0 seconds, because 5 users is still far inside QuickPizza's operational envelope.
5. Neither response time nor error rate degraded over time during the soak test. A soak test lasting several hours could reveal long-term memory leaks, thread pool leaks, or unclosed database connection pools.
6. p(95) is the time that 95% of requests finish within, so 1 request in 20 is slower than it. The average hides these slow requests: a few very slow responses barely move the average, but real users who hit them have a bad experience. Requirements and thresholds therefore use percentiles.
7. 1. Public internet latency and jitter add uncontrolled network delays. 2) Shared public servers experience fluctuating background traffic from other users worldwide, unlike an isolated lab environment.
8. Launching high-load tests against unauthorized systems simulates a Distributed Denial of Service (DDoS) attack, which can crash the application, disrupt service for legitimate users, and incur legal liability.
9. Vague requirement: "The social media feed should load quickly under typical load."  
   Testable requirement: "When 50 concurrent virtual users request the home feed over 5 minutes, 95% of requests must complete within 800ms and less than 1% of requests may fail."  
   K6 options & thresholds:

```javascript
export const options = {
  vus: 50,
  duration: "5m",
  thresholds: {
    http_req_duration: ["p(95)<800"],
    http_req_failed: ["rate<0.01"],
  },
};
```
