# Set 19

| S.No. | Question                                                                                                                                                           |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [How do you implement load shedding for high-traffic endpoints?](#question-1-how-do-you-implement-load-shedding-for-high-traffic-endpoints)                        |
| 2.    | [How do you implement monitoring using `prom-client` for Prometheus?](#question-2-how-do-you-implement-monitoring-using-prom-client-for-prometheus)                |
| 3.    | [How do you implement health checks with dependency checks?](#question-3-how-do-you-implement-health-checks-with-dependency-checks)                                |
| 4.    | [How do you implement request context propagation across async calls?](#question-4-how-do-you-implement-request-context-propagation-across-async-calls)            |
| 5.    | [How do you implement rate limiting based on user roles?](#question-5-how-do-you-implement-rate-limiting-based-on-user-roles)                                      |
| 6.    | [How do you implement input sanitization to prevent XSS attacks?](#question-6-how-do-you-implement-input-sanitization-to-prevent-xss-attacks)                      |
| 7.    | [How do you implement API key authentication in Node.js?](#question-7-how-do-you-implement-api-key-authentication-in-nodejs)                                       |
| 8.    | [How do you implement background jobs that survive process restarts?](#question-8-how-do-you-implement-background-jobs-that-survive-process-restarts)              |
| 9.    | [How do you implement dynamic configuration loading from a database?](#question-9-how-do-you-implement-dynamic-configuration-loading-from-a-database)              |
| 10.   | [How do you implement structured error responses for APIs?](#question-10-how-do-you-implement-structured-error-responses-for-apis)                                 |
| 11.   | [How does Node.js handle libuv thread pool tasks?](#question-11-how-does-nodejs-handle-libuv-thread-pool-tasks)                                                    |
| 12.   | [How do you tune the libuv thread pool size?](#question-12-how-do-you-tune-the-libuv-thread-pool-size)                                                             |
| 13.   | [How do you profile asynchronous code execution in Node.js?](#question-13-how-do-you-profile-asynchronous-code-execution-in-nodejs)                                |
| 14.   | [How do you measure latency in Node.js applications?](#question-14-how-do-you-measure-latency-in-nodejs-applications)                                              |
| 15.   | [How do you implement backpressure handling in network streams?](#question-15-how-do-you-implement-backpressure-handling-in-network-streams)                       |
| 16.   | [How do you implement clustered WebSocket servers with sticky sessions?](#question-16-how-do-you-implement-clustered-websocket-servers-with-sticky-sessions)       |
| 17.   | [How do you implement service discovery for microservices in Node.js?](#question-17-how-do-you-implement-service-discovery-for-microservices-in-nodejs)            |
| 18.   | [How do you implement distributed tracing with Jaeger or OpenTelemetry?](#question-18-how-do-you-implement-distributed-tracing-with-jaeger-or-opentelemetry)       |
| 19.   | [How do you implement distributed rate limiting across multiple instances?](#question-19-how-do-you-implement-distributed-rate-limiting-across-multiple-instances) |
| 20.   | [How do you implement bulk job processing with concurrency control?](#question-20-how-do-you-implement-bulk-job-processing-with-concurrency-control)               |

## Question 1. How do you implement load shedding for high-traffic endpoints?

# Short answer

Load shedding is the practice of **intentionally rejecting or limiting requests when the system is overloaded** to protect overall availability and latency. In Node.js, it is typically implemented using a combination of **concurrency limits, rate limiting, circuit breakers, backpressure, request queues with bounded size, and returning `429` or `503` responses** before the application becomes unhealthy.

---

# Explanation

Load shedding is preferable to allowing the event loop, memory, or downstream services (database, Redis, APIs) to become saturated. A well-designed Node.js service should fail fast instead of degrading for every client.

## Common load shedding strategies

### 1. Limit concurrent requests (recommended)

Instead of accepting unlimited work, allow only a fixed number of requests to execute simultaneously.

```
Incoming Requests
       │
       ▼
+----------------+
| Concurrency    |
| Limit (100)    |
+----------------+
   │        │
Accepted  Rejected (503)
```

Benefits:

- Protects CPU
- Prevents event loop starvation
- Prevents memory spikes
- Stabilizes latency

---

### 2. Bounded request queue

Instead of immediate rejection:

- execute up to N requests
- queue another M requests
- reject anything beyond that

```
Running: 100
Waiting: 50
Beyond 150 → 503
```

Trade-off:

- Better user experience for small bursts
- Queue must stay bounded or latency explodes.

---

### 3. Rate limiting

Limit requests by:

- user
- IP
- API key
- organization

Example:

```
100 requests/minute
```

Useful against:

- abusive clients
- bots
- accidental traffic spikes

Common middleware:

- express-rate-limit
- rate-limiter-flexible

---

### 4. Priority-based load shedding

Reject low-priority traffic first.

Example:

```
Health checks      ✓
Checkout           ✓
Payments           ✓

Analytics          ✗
Recommendation API ✗
Image resize       ✗
```

Large distributed systems commonly use this approach.

---

### 5. Adaptive load shedding

Monitor metrics like:

- event loop delay
- CPU
- memory
- queue length
- downstream latency

When thresholds exceed limits:

```
Event loop delay > 100ms

↓

Reject 20% requests

↓

Delay grows

↓

Reject 50%
```

This is common in high-scale production systems.

---

## Why this matters in Node.js

Node has:

- one JavaScript thread
- shared event loop

If thousands of expensive requests enter simultaneously:

- event loop becomes delayed
- timers fire late
- sockets wait
- memory increases
- GC pauses become longer

Instead of slowing every request:

```
100 requests succeed quickly

900 receive 503 immediately
```

This keeps the service healthy.

---

## HTTP status codes

Typically return:

**429 Too Many Requests**

Use when:

- per-client rate limiting

**503 Service Unavailable**

Use when:

- server overload
- concurrency exceeded
- maintenance
- load shedding

Often include:

```
Retry-After: 5
```

---

## Production architecture

```
Internet
    │
    ▼
Load Balancer
    │
    ▼
Rate Limiter
    │
    ▼
Concurrency Limiter
    │
    ▼
Application
    │
    ▼
Database
```

Each layer protects the next.

---

# Example (JavaScript)

```javascript
import express from "express";

const app = express();

const MAX_CONCURRENT = 5;
let activeRequests = 0;

app.get("/work", async (req, res) => {
  if (activeRequests >= MAX_CONCURRENT) {
    return res.status(503).set("Retry-After", "5").json({
      error: "Server overloaded. Please retry later.",
    });
  }

  activeRequests++;

  try {
    // Simulate expensive work
    await new Promise((resolve) => setTimeout(resolve, 3000));

    res.json({
      success: true,
    });
  } finally {
    activeRequests--;
  }
});

app.listen(3000, () => {
  console.log("Server listening on port 3000");
});
```

When more than five requests arrive simultaneously:

- first five execute
- remaining requests receive `503 Service Unavailable`

This simple pattern is often combined with semaphores (e.g., `p-limit` or `async-mutex`) for more robust concurrency control.

---

# Testing

**Unit testing**

- Verify requests below the concurrency threshold are accepted.
- Verify requests above the threshold receive `503`.
- Ensure the active request counter is decremented in success and error paths.

**Integration testing**

Generate concurrent traffic with a tool such as `autocannon`:

```bash
npx autocannon -c 100 -d 20 http://localhost:3000/work
```

Expect a mix of successful (`200`) and shed (`503`) responses while maintaining stable latency for accepted requests.

---

# Ops & Monitoring

Monitor:

- Event loop delay (`perf_hooks.monitorEventLoopDelay()`)
- Active request count
- Queue depth
- `503` and `429` response rates
- CPU and memory utilization
- Database connection pool saturation

Instrument these metrics with OpenTelemetry and export them to your observability stack (e.g., Prometheus/Grafana). Log load-shedding events with structured logs, including the reason (concurrency limit, queue full, downstream failure) and request metadata. Use a process manager such as PM2, `systemd`, or container orchestration with health/readiness probes so overloaded instances can be managed appropriately.

---

# Deployment & Scaling

- Perform load shedding as close to the edge as possible (API gateway, ingress, or reverse proxy) to reduce unnecessary application work.
- Configure database and HTTP client connection pools with sensible upper bounds to avoid cascading failures.
- Scale horizontally behind a load balancer rather than increasing per-instance concurrency indefinitely.
- In containers, set CPU and memory limits carefully; tune concurrency based on available resources.
- For serverless environments, use reserved/provisioned concurrency where supported and design for cold starts by keeping initialization lightweight.
- Prefer modern LTS Node.js versions (Node.js 20+ or newer) for improved performance and diagnostics.

---

# Pitfalls

- Rejecting requests without a `Retry-After` header, making client retry behavior less predictable.
- Allowing unbounded request queues, which increases latency and memory usage instead of protecting the service.
- Applying only IP-based rate limiting and ignoring per-user, per-tenant, or downstream resource limits.

## Question 2. How do you implement monitoring using `prom-client` for Prometheus?

## Question 3. How do you implement health checks with dependency checks?

## Question 4. How do you implement request context propagation across async calls?

## Question 5. How do you implement rate limiting based on user roles?

## Question 6. How do you implement input sanitization to prevent XSS attacks?

## Question 7. How do you implement API key authentication in Node.js?

## Question 8. How do you implement background jobs that survive process restarts?

## Question 9. How do you implement dynamic configuration loading from a database?

## Question 10. How do you implement structured error responses for APIs?

## Question 11. How does Node.js handle libuv thread pool tasks?

## Question 12. How do you tune the libuv thread pool size?

## Question 13. How do you profile asynchronous code execution in Node.js?

## Question 14. How do you measure latency in Node.js applications?

## Question 15. How do you implement backpressure handling in network streams?

## Question 16. How do you implement clustered WebSocket servers with sticky sessions?

## Question 17. How do you implement service discovery for microservices in Node.js?

## Question 18. How do you implement distributed tracing with Jaeger or OpenTelemetry?

## Question 19. How do you implement distributed rate limiting across multiple instances?

## Question 20. How do you implement bulk job processing with concurrency control?
