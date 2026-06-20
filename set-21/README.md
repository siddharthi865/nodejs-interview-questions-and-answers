# Set 21

| S.No. | Question                                                                                                                                                                      |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is Node.js single-threaded model, and how does it handle multiple requests?](#question-1-what-is-nodejs-single-threaded-model-and-how-does-it-handle-multiple-requests) |
| 2.    | [How do you handle uncaught exceptions using domains in Node.js?](#question-2-how-do-you-handle-uncaught-exceptions-using-domains-in-nodejs)                                  |
| 3.    | [What are the pros and cons of using Node.js for backend development?](#question-3-what-are-the-pros-and-cons-of-using-nodejs-for-backend-development)                        |
| 4.    | [How do you search for a package in npm?](#question-4-how-do-you-search-for-a-package-in-npm)                                                                                 |
| 5.    | [How do you initialize a package with `npm init -y`?](#question-5-how-do-you-initialize-a-package-with-npm-init--y)                                                           |
| 6.    | [How do you uninstall a globally installed npm package?](#question-6-how-do-you-uninstall-a-globally-installed-npm-package)                                                   |
| 7.    | [How do you update Node.js to the latest stable version?](#question-7-how-do-you-update-nodejs-to-the-latest-stable-version)                                                  |
| 8.    | [How do you check the npm version in Node.js?](#question-8-how-do-you-check-the-npm-version-in-nodejs)                                                                        |
| 9.    | [How do you install a specific version of a package using npm?](#question-9-how-do-you-install-a-specific-version-of-a-package-using-npm)                                     |
| 10.   | [How do you view the dependency tree of a project in npm?](#question-10-how-do-you-view-the-dependency-tree-of-a-project-in-npm)                                              |
| 11.   | [How do you list outdated packages in npm?](#question-11-how-do-you-list-outdated-packages-in-npm)                                                                            |
| 12.   | [How do you install a package as a development dependency?](#question-12-how-do-you-install-a-package-as-a-development-dependency)                                            |
| 13.   | [How do you remove a package from `package.json`?](#question-13-how-do-you-remove-a-package-from-packagejson)                                                                 |
| 14.   | [How do you use nvm to switch Node.js versions?](#question-14-how-do-you-use-nvm-to-switch-nodejs-versions)                                                                   |
| 15.   | [How do you set the default Node.js version using nvm?](#question-15-how-do-you-set-the-default-nodejs-version-using-nvm)                                                     |
| 16.   | [How do you pause and resume a readable stream manually?](#question-16-how-do-you-pause-and-resume-a-readable-stream-manually)                                                |
| 17.   | [How do you implement a simple HTTP GET request without external libraries?](#question-17-how-do-you-implement-a-simple-http-get-request-without-external-libraries)          |
| 18.   | [How do you implement a simple HTTP POST request without external libraries?](#question-18-how-do-you-implement-a-simple-http-post-request-without-external-libraries)        |
| 19.   | [How do you parse query strings manually in Node.js?](#question-19-how-do-you-parse-query-strings-manually-in-nodejs)                                                         |
| 20.   | [How do you handle URL encoding and decoding in Node.js?](#question-20-how-do-you-handle-url-encoding-and-decoding-in-nodejs)                                                 |

## Question 1. What is Node.js single-threaded model, and how does it handle multiple requests?

# Short answer

Node.js uses a **single-threaded JavaScript execution model**, meaning your application code runs on one main thread. It can still handle **thousands of concurrent requests** because I/O operations (network, file system, DNS, etc.) are asynchronous and delegated to **libuv**, the operating system, or a thread pool. Once an operation completes, its callback or Promise continuation is queued back to the **event loop** for execution.

---

# Explanation

The phrase **"Node.js is single-threaded"** is often misunderstood.

- **JavaScript execution** happens on a **single main thread**.
- **Node.js runtime is not single-threaded internally.**
  - **libuv** manages the event loop.
  - The OS handles non-blocking network sockets.
  - A configurable thread pool (default size **4**) handles operations like:
    - File system I/O
    - Some DNS lookups
    - Crypto operations
    - Compression (zlib)

## How multiple requests are handled

Suppose 10,000 clients send HTTP requests simultaneously.

1. Node accepts incoming socket connections.
2. Each request enters the event loop.
3. If the request performs asynchronous work:
   - Database query
   - HTTP API call
   - File read

4. Node immediately registers the operation and continues processing other requests.
5. When the operation completes, libuv places the callback (or Promise resolution) into the appropriate event loop queue.
6. The event loop executes the callback when the main thread becomes available.

This allows one JavaScript thread to manage many concurrent requests efficiently.

```
          Incoming Requests
                 │
                 ▼
         Node.js HTTP Server
                 │
                 ▼
           Event Loop (1 JS Thread)
          ┌──────────┬──────────┐
          │          │          │
          ▼          ▼          ▼
      DB Query   File Read   HTTP Call
          │          │          │
          ▼          ▼          ▼
     OS / libuv / Thread Pool
          │          │          │
          └──────┬───┴──────────┘
                 ▼
        Callback / Promise Ready
                 ▼
           Event Loop Executes
```

## Why this is efficient

Traditional thread-per-request servers create one OS thread for every request.

```
1000 Requests
    ↓
1000 Threads
```

This results in:

- High memory usage
- Frequent context switching
- Thread synchronization overhead

Node.js instead uses:

```
1000 Requests
      ↓
1 JavaScript Thread
+
Async I/O
+
Event Loop
```

This drastically reduces overhead for I/O-heavy applications.

## What happens with CPU-intensive work?

The event loop executes only one JavaScript task at a time.

Example:

```js
while (true) {}
```

or

```js
for (let i = 0; i < 1e10; i++) {}
```

During this time:

- No new requests are processed.
- Timers stop firing.
- Promises cannot continue.
- All clients wait.

This is called **blocking the event loop**.

For CPU-intensive work, use:

- `worker_threads`
- Child processes
- Separate microservices
- External job queues

## Event loop example

Request A:

```text
GET /users
```

Needs database access.

```
Main Thread
------------
Receive request
↓
Start DB query
↓
Continue handling other requests
```

While the database is working, Node processes:

```
GET /health
GET /products
GET /orders
GET /metrics
```

When the database responds:

```
Database finished
        ↓
Callback queued
        ↓
Event Loop executes callback
        ↓
Response sent
```

The JavaScript thread was never blocked waiting for the database.

---

# Example (JavaScript)

```javascript
import http from "node:http";

const server = http.createServer(async (_req, res) => {
  // Simulate asynchronous I/O
  await new Promise((resolve) => setTimeout(resolve, 1000));

  res.writeHead(200, { "Content-Type": "text/plain" });
  res.end("Done\n");
});

server.listen(3000, () => {
  console.log("Listening on http://localhost:3000");
});
```

Run multiple concurrent requests:

```bash
curl http://localhost:3000 &
curl http://localhost:3000 &
curl http://localhost:3000 &
```

All requests begin immediately, and none block the others while waiting for the asynchronous timer. This illustrates how the event loop can manage multiple concurrent operations without creating a JavaScript thread per request.

---

# Testing

**Unit testing**

- Mock asynchronous dependencies (database, HTTP clients, file system).
- Verify handlers return expected responses without blocking.

**Integration testing**

- Use concurrent request tests with tools such as `autocannon` or `supertest` to verify latency and throughput under load.

Example using the built-in test runner:

```bash
node --test
```

---

# Ops & Monitoring

- Monitor **event loop delay** using `perf_hooks.monitorEventLoopDelay()` to detect blocking code.
- Track CPU, memory, request latency, and throughput with metrics (e.g., Prometheus).
- Instrument services using OpenTelemetry for distributed tracing.
- Log structured JSON with request IDs for correlation.
- Handle uncaught exceptions and unhandled promise rejections gracefully, then restart via PM2, systemd, or container orchestrators.

---

# Deployment & Scaling

- Use horizontal scaling (multiple Node.js processes) with containers or orchestration platforms instead of relying on a single process.
- Use connection pooling for databases to avoid exhausting backend resources.
- Offload CPU-heavy work to `worker_threads` or dedicated worker services.
- For serverless deployments, minimize cold-start overhead by reducing startup work and reusing connections where supported.
- Prefer modern LTS releases (Node.js 20+ or newer supported LTS versions) for improved performance and runtime features.

---

# Pitfalls

- **Don't block the event loop** with CPU-heavy loops or synchronous APIs (`fs.readFileSync`, `crypto.pbkdf2Sync`) on request paths.
- **Don't confuse concurrency with parallelism**—Node handles many concurrent I/O operations, but JavaScript code executes on one main thread unless you explicitly use workers.
- **Avoid saturating the libuv thread pool** with many concurrent file system or crypto operations; increase `UV_THREADPOOL_SIZE` cautiously only after measuring.

## Question 2. How do you handle uncaught exceptions using domains in Node.js?

## Question 3. What are the pros and cons of using Node.js for backend development?

## Question 4. How do you search for a package in npm?

## Question 5. How do you initialize a package with `npm init -y`?

## Question 6. How do you uninstall a globally installed npm package?

## Question 7. How do you update Node.js to the latest stable version?

## Question 8. How do you check the npm version in Node.js?

## Question 9. How do you install a specific version of a package using npm?

## Question 10. How do you view the dependency tree of a project in npm?

## Question 11. How do you list outdated packages in npm?

## Question 12. How do you install a package as a development dependency?

## Question 13. How do you remove a package from `package.json`?

## Question 14. How do you use nvm to switch Node.js versions?

## Question 15. How do you set the default Node.js version using nvm?

## Question 16. How do you pause and resume a readable stream manually?

## Question 17. How do you implement a simple HTTP GET request without external libraries?

## Question 18. How do you implement a simple HTTP POST request without external libraries?

## Question 19. How do you parse query strings manually in Node.js?

## Question 20. How do you handle URL encoding and decoding in Node.js?
