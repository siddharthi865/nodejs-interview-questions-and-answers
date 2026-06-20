# Set 14

| S.No. | Question                                                                                                                                                                          |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you implement retries for failed HTTP requests in Node.js?](#question-1-how-do-you-implement-retries-for-failed-http-requests-in-nodejs)                                  |
| 2.    | [How do you implement exponential backoff in Node.js?](#question-2-how-do-you-implement-exponential-backoff-in-nodejs)                                                            |
| 3.    | [How do you implement a job queue using `bull` or `bee-queue`?](#question-3-how-do-you-implement-a-job-queue-using-bull-or-bee-queue)                                             |
| 4.    | [How do you process jobs asynchronously using worker processes?](#question-4-how-do-you-process-jobs-asynchronously-using-worker-processes)                                       |
| 5.    | [How do you handle failed jobs in Node.js queues?](#question-5-how-do-you-handle-failed-jobs-in-nodejs-queues)                                                                    |
| 6.    | [How do you schedule cron jobs using `node-cron`?](#question-6-how-do-you-schedule-cron-jobs-using-node-cron)                                                                     |
| 7.    | [How do you implement health checks in a Node.js application?](#question-7-how-do-you-implement-health-checks-in-a-nodejs-application)                                            |
| 8.    | [How do you implement graceful shutdown for background workers?](#question-8-how-do-you-implement-graceful-shutdown-for-background-workers)                                       |
| 9.    | [How do you implement dynamic configuration updates without restarting the server?](#question-9-how-do-you-implement-dynamic-configuration-updates-without-restarting-the-server) |
| 10.   | [How do you implement versioned APIs using Express.js?](#question-10-how-do-you-implement-versioned-apis-using-expressjs)                                                         |
| 11.   | [How does Node.js manage event loop ticks and timers internally?](#question-11-how-does-nodejs-manage-event-loop-ticks-and-timers-internally)                                     |
| 12.   | [How do microtasks and macrotasks affect Node.js event loop performance?](#question-12-how-do-microtasks-and-macrotasks-affect-nodejs-event-loop-performance)                     |
| 13.   | [How do you implement backpressure handling in network streams?](#question-13-how-do-you-implement-backpressure-handling-in-network-streams)                                      |
| 14.   | [How do you implement clustering with load balancing across CPU cores?](#question-14-how-do-you-implement-clustering-with-load-balancing-across-cpu-cores)                        |
| 15.   | [How do you share state between clustered Node.js workers?](#question-15-how-do-you-share-state-between-clustered-nodejs-workers)                                                 |
| 16.   | [How do you implement distributed caching with Redis or Memcached?](#question-16-how-do-you-implement-distributed-caching-with-redis-or-memcached)                                |
| 17.   | [How do you implement zero-downtime deployment for Node.js services?](#question-17-how-do-you-implement-zero-downtime-deployment-for-nodejs-services)                             |
| 18.   | [How do you implement health check endpoints for Kubernetes deployment?](#question-18-how-do-you-implement-health-check-endpoints-for-kubernetes-deployment)                      |
| 19.   | [How do you implement rate limiting in a distributed system?](#question-19-how-do-you-implement-rate-limiting-in-a-distributed-system)                                            |
| 20.   | [How do you handle high concurrency in Node.js without blocking the event loop?](#question-20-how-do-you-handle-high-concurrency-in-nodejs-without-blocking-the-event-loop)       |

## Question 1. How do you implement retries for failed HTTP requests in Node.js?

# Short answer

Implement retries by wrapping HTTP requests in a retry mechanism that retries **only transient failures** (e.g., network errors, `408`, `429`, `502`, `503`, `504`) using **exponential backoff with jitter**, a **maximum retry limit**, and **timeouts**. Avoid retrying non-idempotent requests unless they use idempotency keys. In production, combine retries with **circuit breakers**, **rate limiting**, and **observability**.

---

# Explanation

Retries improve resilience against temporary failures but can also amplify outages if implemented incorrectly.

### When should you retry?

Retry only for **transient errors**, such as:

- Connection reset (`ECONNRESET`)
- Connection timeout (`ETIMEDOUT`)
- DNS temporary failures (`EAI_AGAIN`)
- HTTP:
  - `408 Request Timeout`
  - `429 Too Many Requests`
  - `500` (sometimes)
  - `502 Bad Gateway`
  - `503 Service Unavailable`
  - `504 Gateway Timeout`

Avoid retries for:

- `400 Bad Request`
- `401 Unauthorized`
- `403 Forbidden`
- `404 Not Found`
- Validation errors
- Business logic failures

---

## Exponential backoff

Instead of retrying immediately:

```
Retry 1 -> 100ms
Retry 2 -> 200ms
Retry 3 -> 400ms
Retry 4 -> 800ms
```

Formula:

```
delay = baseDelay × (2 ^ attempt)
```

This reduces load on an already struggling service.

---

## Add jitter

Without jitter:

```
1000 clients retry exactly after 1 second
```

This causes a **thundering herd** problem.

Instead:

```
Retry after:
830ms
920ms
1180ms
1045ms
```

Typical implementation:

```js
delay = exponentialDelay + Math.random() * 100;
```

or use **full jitter**:

```
random(0, exponentialDelay)
```

---

## Respect idempotency

Retrying

```
GET
HEAD
OPTIONS
PUT
DELETE
```

is usually safe.

Retrying

```
POST
PATCH
```

can duplicate operations.

For payment/order APIs, use:

```
Idempotency-Key
```

so repeated requests are treated as one operation.

---

## Runtime behavior

Retries are usually implemented with:

- `await`
- `setTimeout`
- Promise-based delay

During the delay:

- Event loop remains free
- No thread is blocked
- libuv timer queue wakes the callback later

Example:

```
request
   ↓
fails
   ↓
setTimeout(500ms)
   ↓
event loop continues processing other requests
   ↓
retry
```

This is inexpensive even with many concurrent retries, provided retry limits are enforced.

---

## Production considerations

Good retry implementations include:

- maximum retries
- overall timeout
- exponential backoff
- jitter
- retry only transient errors
- cancellation (`AbortController`)
- circuit breaker
- metrics

Avoid:

```
while(true){
   retry();
}
```

This can overwhelm downstream services.

---

# Example (JavaScript)

```javascript
import { setTimeout as delay } from "node:timers/promises";

async function fetchWithRetry(url, options = {}, maxRetries = 3) {
  let attempt = 0;

  while (true) {
    try {
      const response = await fetch(url, {
        ...options,
        signal: AbortSignal.timeout(5000), // Node.js 18+
      });

      if (
        response.ok ||
        ![408, 429, 500, 502, 503, 504].includes(response.status)
      ) {
        return response;
      }

      throw new Error(`Retryable HTTP ${response.status}`);
    } catch (err) {
      if (attempt >= maxRetries) {
        throw err;
      }

      const backoff = Math.min(1000 * 2 ** attempt, 8000);
      const jitter = Math.random() * 200;

      await delay(backoff + jitter);
      attempt++;
    }
  }
}

// Example usage
(async () => {
  try {
    const response = await fetchWithRetry("https://httpbin.org/status/503");
    console.log(await response.text());
  } catch (err) {
    console.error("Request failed:", err.message);
  }
})();
```

**Why this approach?**

- Uses native `fetch` (Node.js 18+).
- Uses `AbortSignal.timeout()` for request timeouts.
- Applies exponential backoff with jitter.
- Retries only retryable HTTP status codes.
- Limits retry attempts to prevent infinite loops.

---

# Testing

### Unit testing

Mock `fetch` to simulate:

- success on first attempt
- success after retries
- permanent failure
- timeout
- retryable vs non-retryable status codes

Example using the built-in test runner:

```bash
node --test
```

Example assertion:

```javascript
import test from "node:test";
import assert from "node:assert/strict";

// Mock global.fetch and verify the number of retry attempts.
```

### Integration testing

- Use a mock HTTP server (e.g., `http.createServer`) that intentionally returns `503` before succeeding.
- Verify retry count, backoff behavior (within tolerance), and eventual success or failure.

---

# Ops & Monitoring

- **Logging:** Log retry attempts with request ID, attempt number, delay, and error class. Avoid logging every retry at error level to reduce noise.
- **Metrics:** Track retry count, success-after-retry rate, final failure rate, latency, and downstream error rates.
- **Tracing:** Use **OpenTelemetry** spans and annotate retries as events or child spans to visualize retry behavior.
- **Error handling:** Honor `Retry-After` headers for `429`/`503` responses when present. Differentiate transient and permanent failures.
- **Process management:** Run Node.js with a process manager (PM2/systemd) or in containers with proper health checks. Retries should not replace crash recovery or service health monitoring.

---

# Deployment & Scaling

- Use connection pooling (`keep-alive`) to reduce connection overhead.
- Apply retries at **one layer only** (client library or API gateway) to avoid retry storms.
- For microservices, combine retries with circuit breakers and bulkheads.
- In containers or Kubernetes, configure readiness/liveness probes to avoid routing traffic to unhealthy instances.
- In serverless environments, keep retry budgets small to minimize cold-start amplification and execution costs.
- Prefer **Node.js 18+** for built-in `fetch` and `AbortSignal.timeout()`. Newer LTS releases (Node.js 20/22+) provide further runtime improvements.

---

# Pitfalls

- **Retrying every error:** Retry only transient failures; retrying validation or authentication errors wastes resources.
- **No backoff or jitter:** Immediate retries can overwhelm a degraded service and create a thundering herd effect.
- **Retrying non-idempotent operations:** Retrying `POST` requests without idempotency protection can create duplicate side effects.

## Question 2. How do you implement exponential backoff in Node.js?

## Question 3. How do you implement a job queue using `bull` or `bee-queue`?

## Question 4. How do you process jobs asynchronously using worker processes?

## Question 5. How do you handle failed jobs in Node.js queues?

## Question 6. How do you schedule cron jobs using `node-cron`?

## Question 7. How do you implement health checks in a Node.js application?

## Question 8. How do you implement graceful shutdown for background workers?

## Question 9. How do you implement dynamic configuration updates without restarting the server?

## Question 10. How do you implement versioned APIs using Express.js?

## Question 11. How does Node.js manage event loop ticks and timers internally?

## Question 12. How do microtasks and macrotasks affect Node.js event loop performance?

## Question 13. How do you implement backpressure handling in network streams?

## Question 14. How do you implement clustering with load balancing across CPU cores?

## Question 15. How do you share state between clustered Node.js workers?

## Question 16. How do you implement distributed caching with Redis or Memcached?

## Question 17. How do you implement zero-downtime deployment for Node.js services?

## Question 18. How do you implement health check endpoints for Kubernetes deployment?

## Question 19. How do you implement rate limiting in a distributed system?

## Question 20. How do you handle high concurrency in Node.js without blocking the event loop?
