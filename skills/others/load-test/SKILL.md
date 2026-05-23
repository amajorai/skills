---
name: load-test
description: Run load tests against any API or web app using k6. Identifies throughput limits, latency degradation, and failure modes under realistic traffic. Use before launch, before a traffic spike, or after a major backend change.
argument-hint: <target URL or API endpoints to test>
---

# Load Test

You are running load tests to find where the system breaks before users do. Work through each phase in order.

**Target:** {{args}}


## Phase 1: Define Test Scenarios

Ask the user (combine related questions):

- **Endpoints**: Which endpoints matter most? (Auth, core CRUD, any expensive queries?)
- **Expected traffic**: What is realistic daily/peak concurrent users?
- **Threshold goals**: What are acceptable response times? (e.g., P95 < 500ms, error rate < 1%)
- **Environment**: Test against staging (preferred) or production? (Never run load tests on production without a maintenance window)

Identify 2–3 scenarios to test:
1. **Baseline**: Steady-state traffic (normal load)
2. **Spike**: Sudden traffic burst (launch day, viral moment)
3. **Soak**: Sustained load over 30+ minutes (memory leaks, connection pool exhaustion)


## Phase 2: Explore

Spawn **1 subagent** to:
- Find API routes and their auth requirements
- Identify any obvious bottlenecks: N+1 queries, missing indexes, synchronous operations in request path
- Check connection pool settings and any rate limiting in place


## Phase 3: Install k6

```bash
# macOS
brew install k6

# Windows
winget install k6

# Docker (no install needed)
docker run --rm -i grafana/k6 run - <script.js
```

k6 is a JavaScript-based load testing tool. Tests are JS files that describe virtual user behavior.


## Phase 4: Write Test Scripts

Create `load-tests/` directory with test files.

**Baseline scenario** (`load-tests/baseline.js`):

```javascript
import http from 'k6/http'
import { check, sleep } from 'k6'

export const options = {
  vus: 50,          // 50 concurrent virtual users
  duration: '5m',
  thresholds: {
    http_req_duration: ['p(95)<500'],   // 95th percentile < 500ms
    http_req_failed: ['rate<0.01'],      // error rate < 1%
  },
}

export default function () {
  const res = http.get('https://staging.yourdomain.com/api/posts', {
    headers: { Authorization: `Bearer ${__ENV.TEST_TOKEN}` },
  })
  check(res, { 'status 200': (r) => r.status === 200 })
  sleep(1)
}
```

**Spike scenario** (`load-tests/spike.js`):

```javascript
export const options = {
  stages: [
    { duration: '1m', target: 10 },   // warm up
    { duration: '30s', target: 200 },  // spike to 200 users
    { duration: '1m', target: 200 },   // hold
    { duration: '30s', target: 10 },   // ramp down
    { duration: '1m', target: 10 },    // recovery
  ],
}
```

Write one script per endpoint or user flow to test.


## Phase 5: Run Tests

Against staging (never production without approval):

```bash
k6 run load-tests/baseline.js
k6 run load-tests/spike.js --env TEST_TOKEN=<token>

# With HTML report
k6 run --out json=results.json load-tests/baseline.js
```

Monitor during the run:
- Database CPU and connection count
- Server CPU and memory
- Error logs
- Response time dashboard (if available)


## Phase 6: Interpret Results

k6 output key metrics:

| Metric | What it means | Alert threshold |
|--------|---------------|-----------------|
| `http_req_duration` P95 | 95% of requests faster than this | > 1s is a problem |
| `http_req_failed` | % of requests that errored | > 1% is a problem |
| `vus` | Concurrent virtual users at peak | |
| `http_reqs/s` | Throughput (requests per second) | |

Identify:
- At what VU count does P95 exceed the threshold?
- What error messages appear under load?
- Does performance degrade gradually (capacity limit) or suddenly (connection pool exhausted, OOM)?


## Phase 7: Fix Bottlenecks

Common findings and fixes:

| Finding | Fix |
|---------|-----|
| P95 spikes at N users | Add database index, optimize query, add caching |
| Connection pool exhaustion | Increase pool size, add connection pooler (PgBouncer) |
| Memory grows during soak test | Fix memory leak: check for unbounded caches, event listeners not removed |
| 5xx errors under spike | Add rate limiting, circuit breaker, or queue for expensive operations |
| Slow 3rd-party calls | Add timeout, cache response, or make async |

After each fix, re-run the test that exposed the issue and confirm improvement.


## Phase 8: Report

Produce a load test report:

- Baseline: max concurrent users at < 500ms P95
- Spike behavior: how the system responds to 5x normal traffic
- Soak: memory/CPU stability over 30 min
- Bottlenecks found and fixed
- Recommended scale-up trigger (CPU%, memory%, requests/s)


## Completion Report

- Scenarios tested (list with VU counts and duration)
- Thresholds set and whether they passed
- Bottlenecks identified and fixed
- Maximum sustainable load (users, req/s)
- Recommended scaling triggers
