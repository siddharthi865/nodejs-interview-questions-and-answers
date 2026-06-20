# Set 10

| S.No. | Question                                                                                                                                                                                        |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you monitor Node.js performance using `clinic.js` or Node.js profiling?](#question-1-how-do-you-monitor-nodejs-performance-using-clinicjs-or-nodejs-profiling)                          |
| 2.    | [How do you handle high traffic in Node.js?](#question-2-how-do-you-handle-high-traffic-in-nodejs)                                                                                              |
| 3.    | [How do you implement graceful shutdown in Node.js?](#question-3-how-do-you-implement-graceful-shutdown-in-nodejs)                                                                              |
| 4.    | [How do you implement distributed locks in Node.js?](#question-4-how-do-you-implement-distributed-locks-in-nodejs)                                                                              |
| 5.    | [How do you manage multiple microservices with Node.js?](#question-5-how-do-you-manage-multiple-microservices-with-nodejs)                                                                      |
| 6.    | [How do you implement request tracing in Node.js microservices?](#question-6-how-do-you-implement-request-tracing-in-nodejs-microservices)                                                      |
| 7.    | [How do you implement service-to-service authentication in Node.js?](#question-7-how-do-you-implement-service-to-service-authentication-in-nodejs)                                              |
| 8.    | [How do you implement rate limiting in distributed Node.js systems?](#question-8-how-do-you-implement-rate-limiting-in-distributed-nodejs-systems)                                              |
| 9.    | [How do you prevent denial-of-service (DoS) attacks in Node.js?](#question-9-how-do-you-prevent-denial-of-service-dos-attacks-in-nodejs)                                                        |
| 10.   | [How do you implement OAuth2 with refresh tokens in Node.js?](#question-10-how-do-you-implement-oauth2-with-refresh-tokens-in-nodejs)                                                           |
| 11.   | [How do you implement API gateways with Node.js?](#question-11-how-do-you-implement-api-gateways-with-nodejs)                                                                                   |
| 12.   | [How do you implement real-time analytics in Node.js?](#question-12-how-do-you-implement-real-time-analytics-in-nodejs)                                                                         |
| 13.   | [How do you implement backpressure handling in Node.js streams?](#question-13-how-do-you-implement-backpressure-handling-in-nodejs-streams)                                                     |
| 14.   | [How do you handle large file uploads in a memory-efficient way?](#question-14-how-do-you-handle-large-file-uploads-in-a-memory-efficient-way)                                                  |
| 15.   | [How do you implement async queue processing in Node.js?](#question-15-how-do-you-implement-async-queue-processing-in-nodejs)                                                                   |
| 16.   | [How do you implement message brokers (like RabbitMQ or Kafka) with Node.js?](#question-16-how-do-you-implement-message-brokers-like-rabbitmq-or-kafka-with-nodejs)                             |
| 17.   | [How do you handle cross-service communication in Node.js microservices?](#question-17-how-do-you-handle-cross-service-communication-in-nodejs-microservices)                                   |
| 18.   | [How do you ensure fault tolerance in Node.js applications?](#question-18-how-do-you-ensure-fault-tolerance-in-nodejs-applications)                                                             |
| 19.   | [How do you implement hot reload or zero-downtime deployment in Node.js?](#question-19-how-do-you-implement-hot-reload-or-zero-downtime-deployment-in-nodejs)                                   |
| 20.   | [How do you implement observability (metrics, tracing, logging) in Node.js production apps?](#question-20-how-do-you-implement-observability-metrics-tracing-logging-in-nodejs-production-apps) |

## Question 1. How do you monitor Node.js performance using `clinic.js` or Node.js profiling?

# Short answer

`clinic.js` is a production-focused performance diagnostics toolkit that automates profiling and visualizes bottlenecks in CPU, event loop, async operations, and memory. Native Node.js profiling (`--inspect`, `--cpu-prof`, `--heap-prof`, `perf_hooks`, `trace_events`) provides lower-level profiling data that integrates with Chrome DevTools and other profilers. In practice, I use application metrics (OpenTelemetry/Prometheus) for continuous monitoring and `clinic.js` or CPU/heap profiling only when investigating performance regressions.

---

# Explanation

Performance monitoring in Node.js typically happens at three levels:

1. **Continuous production monitoring**
   - Request latency (P50, P95, P99)
   - Throughput (RPS)
   - Event loop lag
   - Memory usage
   - CPU utilization
   - Garbage Collection pauses
   - Error rates

2. **Application profiling**
   - CPU hotspots
   - Blocking synchronous code
   - Memory leaks
   - Async bottlenecks

3. **System profiling**
   - OS scheduling
   - Network
   - Disk I/O
   - Container resource limits

---

## 1. Clinic.js

`clinic.js` is a suite of profiling tools developed by NearForm.

It contains several utilities:

| Tool                 | Purpose                          |
| -------------------- | -------------------------------- |
| Clinic Doctor        | Finds general performance issues |
| Clinic Flame         | CPU flamegraphs                  |
| Clinic Bubbleprof    | Async flow visualization         |
| Clinic Heap Profiler | Memory analysis                  |

Example:

```bash
npm install -g clinic

clinic doctor -- node app.js
```

Generate load:

```bash
autocannon http://localhost:3000
```

After stopping:

```
clinic open
```

Doctor may report:

- Event loop blocked
- Excessive GC
- Slow synchronous work
- High CPU
- Async bottlenecks

---

## 2. Clinic Flame

Shows where CPU time is actually spent.

Example:

```bash
clinic flame -- node app.js
```

Produces an interactive flamegraph.

Useful for detecting:

- expensive JSON parsing
- regex bottlenecks
- nested loops
- crypto
- compression
- synchronous filesystem usage

A flamegraph quickly answers:

> Which function consumes the most CPU?

---

## 3. Clinic Bubbleprof

Bubbleprof visualizes asynchronous execution.

```bash
clinic bubbleprof -- node app.js
```

Excellent for diagnosing:

- Promise chains
- callback-heavy applications
- slow database calls
- unnecessary async hops
- timer delays

Instead of CPU usage, it explains async relationships.

---

## 4. Native CPU profiling

Node has built-in CPU profiling.

```bash
node --cpu-prof app.js
```

After execution:

```
CPU.<id>.cpuprofile
```

Open in Chrome DevTools.

This shows:

- function call tree
- self time
- total time
- optimization status

Useful when you don't want additional tooling.

---

## 5. Inspector Profiling

Start Node:

```bash
node --inspect app.js
```

Open:

```
chrome://inspect
```

Then:

```
Profiler
→ Start profiling
→ Stop
```

You'll get:

- flame chart
- call stack
- memory allocation
- heap snapshots

---

## 6. Heap profiling

Memory leaks require heap analysis.

Run:

```bash
node --heap-prof app.js
```

or

```
Chrome DevTools
→ Memory
→ Heap Snapshot
```

Look for:

- growing object counts
- retained closures
- event listeners
- Maps/Sets never cleared
- cache growth

---

## 7. Event loop monitoring

Node provides `perf_hooks`.

```javascript
// JavaScript (Node.js 18+)

import { monitorEventLoopDelay } from "node:perf_hooks";

const histogram = monitorEventLoopDelay();

histogram.enable();

setInterval(() => {
  console.log({
    meanMs: (histogram.mean / 1e6).toFixed(2),
    maxMs: (histogram.max / 1e6).toFixed(2),
  });

  histogram.reset();
}, 5000);
```

High event-loop delay usually means:

- synchronous loops
- blocking crypto
- sync file operations
- large JSON serialization

---

## 8. Measuring execution time

Use Performance API.

```javascript
import { performance } from "node:perf_hooks";

performance.mark("start");

await doWork();

performance.mark("end");

performance.measure("doWork", "start", "end");

console.log(performance.getEntriesByName("doWork")[0].duration);
```

Better than using `Date.now()`.

---

## 9. Memory monitoring

```javascript
setInterval(() => {
  console.log(process.memoryUsage());
}, 5000);
```

Important values:

- rss
- heapUsed
- heapTotal
- external
- arrayBuffers

A steadily increasing `heapUsed` after GC cycles may indicate a memory leak.

---

## 10. Production metrics

Typical dashboards monitor:

- Request latency
- P95 latency
- Event loop lag
- Active requests
- CPU
- Memory
- GC pause duration
- Database latency
- Cache hit ratio

These are exported using OpenTelemetry and scraped by Prometheus, then visualized in Grafana.

---

# Example

**JavaScript (Node.js 18+)**

```javascript
import http from "node:http";
import { performance, monitorEventLoopDelay } from "node:perf_hooks";

const histogram = monitorEventLoopDelay({ resolution: 20 });
histogram.enable();

const server = http.createServer(async (_req, res) => {
  const start = performance.now();

  // Simulate async work
  await new Promise((resolve) => setTimeout(resolve, 50));

  const duration = performance.now() - start;

  res.end(`Handled in ${duration.toFixed(2)} ms\n`);
});

server.listen(3000, () => {
  console.log("Listening on :3000");
});

setInterval(() => {
  const mem = process.memoryUsage();

  console.log({
    eventLoopLagMs: (histogram.mean / 1e6).toFixed(2),
    heapUsedMB: (mem.heapUsed / 1024 / 1024).toFixed(2),
    rssMB: (mem.rss / 1024 / 1024).toFixed(2),
  });

  histogram.reset();
}, 5000);
```

Run under load:

```bash
clinic doctor -- node server.js
```

Generate traffic:

```bash
npx autocannon http://localhost:3000
```

This combination provides both live metrics and a post-run performance report.

---

# Testing

Performance profiling is best treated separately from functional tests.

**Unit tests**

- Benchmark critical utilities with representative input sizes.
- Verify instrumentation (e.g., metrics emission) using mocks.

**Integration tests**

- Run load tests using `autocannon`, `k6`, or `wrk`.
- Compare latency and throughput against a performance baseline to detect regressions.

Example using Node's built-in test runner:

```bash
node --test
```

---

# Ops & Monitoring

- Use structured logging (e.g., Pino) with request IDs for correlation.
- Export metrics (latency, event-loop lag, memory, GC, CPU) via OpenTelemetry to Prometheus/Grafana.
- Enable distributed tracing with OpenTelemetry to identify slow downstream services.
- Use `clinic.js` or CPU/heap profiles during incident investigations rather than continuously in production due to profiling overhead.
- Configure process managers (PM2, systemd, Kubernetes) with health checks, graceful shutdown, and automatic restarts.

---

# Deployment & Scaling

- Profile in environments that closely resemble production workloads and data sizes.
- Use horizontal scaling for CPU-bound workloads after identifying hotspots; move heavy computation to `worker_threads` where appropriate.
- Configure database connection pools appropriately to avoid artificial bottlenecks.
- In containers, set realistic CPU and memory limits, as throttling can significantly affect event-loop latency.
- For serverless, minimize cold-start overhead by reducing initialization work and profile warm execution paths separately.
- Prefer modern LTS versions (Node.js 20+ or newer supported LTS) for improved V8 performance and profiling capabilities.

---

# Pitfalls

- **Profiling under unrealistic workloads** can lead to optimizing code paths that are not actual production bottlenecks.
- **Ignoring event-loop delay** while focusing only on CPU usage can miss blocking synchronous operations that hurt latency.
- **Leaving profilers enabled continuously in production** may introduce unnecessary overhead and generate large profiling artifacts.

## Question 2. How do you handle high traffic in Node.js?

## Question 3. How do you implement graceful shutdown in Node.js?

## Question 4. How do you implement distributed locks in Node.js?

## Question 5. How do you manage multiple microservices with Node.js?

## Question 6. How do you implement request tracing in Node.js microservices?

## Question 7. How do you implement service-to-service authentication in Node.js?

## Question 8. How do you implement rate limiting in distributed Node.js systems?

## Question 9. How do you prevent denial-of-service (DoS) attacks in Node.js?

## Question 10. How do you implement OAuth2 with refresh tokens in Node.js?

## Question 11. How do you implement API gateways with Node.js?

## Question 12. How do you implement real-time analytics in Node.js?

## Question 13. How do you implement backpressure handling in Node.js streams?

## Question 14. How do you handle large file uploads in a memory-efficient way?

## Question 15. How do you implement async queue processing in Node.js?

## Question 16. How do you implement message brokers (like RabbitMQ or Kafka) with Node.js?

## Question 17. How do you handle cross-service communication in Node.js microservices?

## Question 18. How do you ensure fault tolerance in Node.js applications?

## Question 19. How do you implement hot reload or zero-downtime deployment in Node.js?

## Question 20. How do you implement observability (metrics, tracing, logging) in Node.js production apps?
