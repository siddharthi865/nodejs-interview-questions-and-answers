# Set 4

| S.No. | Question                                                                                                                                                                                 |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How does Node.js handle multiple requests simultaneously?](#question-1-how-does-nodejs-handle-multiple-requests-simultaneously)                                                         |
| 2.    | [What are the different ways to manage environment-specific configurations in Node.js?](#question-2-what-are-the-different-ways-to-manage-environment-specific-configurations-in-nodejs) |
| 3.    | [How do you secure a Node.js application?](#question-3-how-do-you-secure-a-nodejs-application)                                                                                           |
| 4.    | [How do you prevent SQL injection in Node.js?](#question-4-how-do-you-prevent-sql-injection-in-nodejs)                                                                                   |
| 5.    | [How do you implement rate limiting in Node.js APIs?](#question-5-how-do-you-implement-rate-limiting-in-nodejs-apis)                                                                     |
| 6.    | [What is the difference between `res.send()`, `res.json()`, and `res.end()` in Express?](#question-6-what-is-the-difference-between-ressend-resjson-and-resend-in-express)               |
| 7.    | [What is the purpose of `helmet` middleware in Node.js?](#question-7-what-is-the-purpose-of-helmet-middleware-in-nodejs)                                                                 |
| 8.    | [How do you handle uncaught promise rejections in Node.js?](#question-8-how-do-you-handle-uncaught-promise-rejections-in-nodejs)                                                         |
| 9.    | [How do you implement logging in a Node.js application?](#question-9-how-do-you-implement-logging-in-a-nodejs-application)                                                               |
| 10.   | [How do you test a Node.js application?](#question-10-how-do-you-test-a-nodejs-application)                                                                                              |
| 11.   | [Explain the inner workings of the Node.js event loop phases](#question-11-explain-the-inner-workings-of-the-nodejs-event-loop-phases)                                                   |
| 12.   | [Explain how Node.js handles asynchronous I/O internally](#question-12-explain-how-nodejs-handles-asynchronous-io-internally)                                                            |
| 13.   | [Explain libuv and its role in Node.js](#question-13-explain-libuv-and-its-role-in-nodejs)                                                                                               |
| 14.   | [What is the difference between synchronous and asynchronous I/O in Node.js?](#question-14-what-is-the-difference-between-synchronous-and-asynchronous-io-in-nodejs)                     |
| 15.   | [Explain non-blocking I/O in Node.js](#question-15-explain-non-blocking-io-in-nodejs)                                                                                                    |
| 16.   | [Explain the role of worker threads in Node.js](#question-16-explain-the-role-of-worker-threads-in-nodejs)                                                                               |
| 17.   | [How do you implement graceful shutdown in a Node.js server?](#question-17-how-do-you-implement-graceful-shutdown-in-a-nodejs-server)                                                    |
| 18.   | [Explain memory management in Node.js](#question-18-explain-memory-management-in-nodejs)                                                                                                 |
| 19.   | [How do you profile Node.js applications for performance bottlenecks?](#question-19-how-do-you-profile-nodejs-applications-for-performance-bottlenecks)                                  |
| 20.   | [How do you handle high CPU-intensive tasks in Node.js?](#question-20-how-do-you-handle-high-cpu-intensive-tasks-in-nodejs)                                                              |

## Question 1. How does Node.js handle multiple requests simultaneously?

# Short answer

Node.js handles multiple requests simultaneously using a **single-threaded event loop** combined with **non-blocking I/O**. Instead of creating a new thread per request, it delegates I/O operations (network, file system, DNS, database drivers, etc.) to the OS kernel or libuv's thread pool and continues processing other requests. When an operation completes, a callback, promise, or async function continuation is queued back onto the event loop.

---

# Explanation

### Core architecture

Node.js runs JavaScript on a single main thread using Google's **V8** engine. Concurrency comes from:

1. **Event Loop**
   - Continuously checks for completed operations and executes their callbacks.
   - Processes many connections without blocking.

2. **libuv**
   - Provides the event loop implementation.
   - Manages asynchronous operations.
   - Uses OS-level async APIs where possible.
   - Uses a thread pool (default size: 4) for operations that cannot be handled asynchronously by the OS.

3. **Kernel Networking**
   - TCP sockets are handled efficiently by the operating system.
   - Thousands of open connections can be managed without thousands of threads.

### Request lifecycle

Suppose 1,000 users send requests simultaneously:

```text
Incoming Requests
       |
       v
+----------------+
| Event Loop     |
+----------------+
       |
       | Async DB Query
       v
+----------------+
| DB Driver/OS   |
+----------------+
       |
       | Response Ready
       v
+----------------+
| Callback Queue |
+----------------+
       |
       v
Event Loop sends response
```

The event loop does **not wait** for the database query.

Instead:

1. Request arrives.
2. Node initiates DB query.
3. Event loop immediately handles the next request.
4. DB query finishes later.
5. Completion event is queued.
6. Event loop processes result and sends response.

This is why Node can efficiently handle large numbers of concurrent I/O-bound requests.

---

### Event loop phases

The event loop cycles through phases such as:

1. Timers (`setTimeout`, `setInterval`)
2. Pending callbacks
3. Idle/prepare
4. Poll (I/O events)
5. Check (`setImmediate`)
6. Close callbacks

Microtasks (`Promise.then`, `queueMicrotask`) run between phases and before moving to the next event loop iteration.

---

### What about CPU-intensive work?

Node's concurrency model works extremely well for:

- HTTP APIs
- WebSockets
- Streaming
- Database-heavy applications
- Proxy services

It struggles when JavaScript performs long CPU-bound work:

```js
while (true) {
  // blocks event loop
}
```

During that time:

- No requests are processed.
- No callbacks run.
- Latency spikes.

For CPU-intensive tasks use:

- `worker_threads`
- Separate services
- Background job queues

---

### Thread pool usage

Some operations use libuv's thread pool:

- `fs.readFile()`
- `crypto.pbkdf2()`
- `zlib`
- Some DNS operations

Default size:

```bash
UV_THREADPOOL_SIZE=4
```

Can be increased:

```bash
UV_THREADPOOL_SIZE=16 node app.js
```

Be careful—larger pools consume more memory and CPU.

---

### Why this scales well

Traditional thread-per-request servers:

```text
1000 requests
    |
1000 threads
    |
High memory usage
```

Node.js:

```text
1000 requests
    |
1 Event Loop
+ OS async I/O
+ Thread Pool
    |
Low overhead
```

This significantly reduces:

- Context switching
- Memory consumption
- Thread management costs

---

# Example

**JavaScript (Node.js 20+)**

```js
import http from "node:http";

const server = http.createServer(async (req, res) => {
  // Simulate async I/O
  await new Promise((resolve) => setTimeout(resolve, 2000));

  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Done\n");
});

server.listen(3000, () => {
  console.log("Listening on port 3000");
});
```

If 100 requests arrive simultaneously:

- Node does not create 100 threads.
- Each request starts an async timer.
- Event loop continues accepting requests.
- Responses are sent when timers complete.

---

# Testing

### Unit Testing

Test request handlers independently.

```js
import test from "node:test";
import assert from "node:assert";

test("returns expected response", () => {
  assert.strictEqual(1 + 1, 2);
});
```

Run:

```bash
node --test
```

### Integration Testing

Use:

- Supertest
- Built-in `fetch`
- Autocannon for load testing

Example load test:

```bash
npx autocannon http://localhost:3000
```

This demonstrates concurrent request handling under load.

---

# Ops & Monitoring

### Logging

Use structured logging:

```bash
pino
```

Log:

- Request ID
- Latency
- Status code
- Error details

### Metrics

Track:

- Event loop lag
- Throughput (RPS)
- Memory usage
- CPU utilization
- Active handles

Useful tools:

- Prometheus
- Grafana

### Tracing

Use:

- OpenTelemetry
- Distributed tracing across services

### Error Handling

Always handle:

```js
process.on('unhandledRejection', ...);
process.on('uncaughtException', ...);
```

Prefer graceful shutdown over continuing in a corrupted state.

### Process Management

- PM2 for simple deployments
- systemd for VMs
- Kubernetes for containers

---

# Deployment & Scaling

### Horizontal Scaling

Use all CPU cores:

```js
import cluster from "node:cluster";
```

or preferably:

- Kubernetes replicas
- PM2 cluster mode

### Worker Threads

For CPU-bound workloads:

```js
import { Worker } from "node:worker_threads";
```

### Database Connections

Avoid creating connections per request.

Use pools:

- PostgreSQL (`pg.Pool`)
- MySQL pools
- MongoDB connection reuse

### Containers

- Use Node.js LTS (currently Node 22 LTS or newer supported production versions).
- Set memory limits.
- Run one process per container.

### Serverless

Watch for:

- Cold starts
- Connection reuse
- Initialization overhead

---

# Pitfalls

- Blocking the event loop with CPU-intensive code prevents all requests from being served.
- Excessive Promise microtasks can starve I/O processing.
- Increasing `UV_THREADPOOL_SIZE` blindly can reduce overall performance rather than improve it.

## Question 2. What are the different ways to manage environment-specific configurations in Node.js?

## Question 3. How do you secure a Node.js application?

## Question 4. How do you prevent SQL injection in Node.js?

## Question 5. How do you implement rate limiting in Node.js APIs?

## Question 6. What is the difference between `res.send()`, `res.json()`, and `res.end()` in Express?

## Question 7. What is the purpose of `helmet` middleware in Node.js?

## Question 8. How do you handle uncaught promise rejections in Node.js?

## Question 9. How do you implement logging in a Node.js application?

## Question 10. How do you test a Node.js application?

## Question 11. Explain the inner workings of the Node.js event loop phases

## Question 12. Explain how Node.js handles asynchronous I/O internally

## Question 13. Explain libuv and its role in Node.js

## Question 14. What is the difference between synchronous and asynchronous I/O in Node.js?

## Question 15. Explain non-blocking I/O in Node.js

## Question 16. Explain the role of worker threads in Node.js

## Question 17. How do you implement graceful shutdown in a Node.js server?

## Question 18. Explain memory management in Node.js

## Question 19. How do you profile Node.js applications for performance bottlenecks?

## Question 20. How do you handle high CPU-intensive tasks in Node.js?
