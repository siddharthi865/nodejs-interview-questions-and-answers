# Set 6

| S.No. | Question                                                                                                                                                                     |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What are the main advantages of using Node.js over other server-side platforms?](#question-1-what-are-the-main-advantages-of-using-nodejs-over-other-server-side-platforms) |
| 2.    | [Can Node.js be used for desktop applications? If yes, how?](#question-2-can-nodejs-be-used-for-desktop-applications-if-yes-how)                                             |
| 3.    | [Explain how Node.js handles concurrency](#question-3-explain-how-nodejs-handles-concurrency)                                                                                |
| 4.    | [What is the difference between Node.js runtime and the Node.js library?](#question-4-what-is-the-difference-between-nodejs-runtime-and-the-nodejs-library)                  |
| 5.    | [What are the core modules in Node.js? Give examples](#question-5-what-are-the-core-modules-in-nodejs-give-examples)                                                         |
| 6.    | [What is the difference between CommonJS and ES Modules in Node.js?](#question-6-what-is-the-difference-between-commonjs-and-es-modules-in-nodejs)                           |
| 7.    | [How do you import only a specific function from a module in Node.js?](#question-7-how-do-you-import-only-a-specific-function-from-a-module-in-nodejs)                       |
| 8.    | [What is the difference between local and global Node.js packages?](#question-8-what-is-the-difference-between-local-and-global-nodejs-packages)                             |
| 9.    | [Explain `npm install` vs `npm ci`](#question-9-explain-npm-install-vs-npm-ci)                                                                                               |
| 10.   | [How does Node.js handle file system operations?](#question-10-how-does-nodejs-handle-file-system-operations)                                                                |
| 11.   | [How do you check for a file's existence in Node.js?](#question-11-how-do-you-check-for-a-files-existence-in-nodejs)                                                         |
| 12.   | [How do you append data to a file asynchronously in Node.js?](#question-12-how-do-you-append-data-to-a-file-asynchronously-in-nodejs)                                        |
| 13.   | [Explain the difference between `http` and `https` modules in Node.js](#question-13-explain-the-difference-between-http-and-https-modules-in-nodejs)                         |
| 14.   | [How do you parse URL parameters in Node.js?](#question-14-how-do-you-parse-url-parameters-in-nodejs)                                                                        |
| 15.   | [How do you parse JSON data in Node.js?](#question-15-how-do-you-parse-json-data-in-nodejs)                                                                                  |
| 16.   | [How do you convert a JSON object to a string in Node.js?](#question-16-how-do-you-convert-a-json-object-to-a-string-in-nodejs)                                              |
| 17.   | [What is the `os` module in Node.js?](#question-17-what-is-the-os-module-in-nodejs-implied-from-content)                                                                     |
| 18.   | [How can you get the system memory or CPU information in Node.js?](#question-18-how-can-you-get-the-system-memory-or-cpu-information-in-nodejs)                              |
| 19.   | [What are timers in Node.js?](#question-19-what-are-timers-in-nodejs)                                                                                                        |
| 20.   | [How do you clear a timeout or interval in Node.js?](#question-20-how-do-you-clear-a-timeout-or-interval-in-nodejs)                                                          |

## Question 1. What are the main advantages of using Node.js over other server-side platforms?

# Short answer

Node.js is well-suited for **I/O-intensive, real-time, API-driven, and microservice applications** because it uses a **single-threaded event loop with asynchronous, non-blocking I/O**, enabling high concurrency with relatively low resource usage. It also offers a massive npm ecosystem, allows JavaScript/TypeScript across the full stack, starts quickly, and scales well horizontally.

---

# Explanation

The advantages of Node.js depend on the workload. It's not universally faster than platforms like Java, Go, or .NET, but it excels in specific scenarios.

## 1. Non-blocking, event-driven architecture

Node.js uses an **event loop** built on **libuv**. Rather than blocking a thread while waiting for disk, database, or network operations, it registers callbacks/promises and continues serving other requests.

Example:

- Traditional blocking server
  - Request waits for database.
  - Thread remains occupied.

- Node.js
  - Database request is initiated.
  - Event loop immediately serves another request.
  - Response is processed when I/O completes.

This allows thousands of concurrent connections with relatively few OS threads.

**Best for:**

- REST APIs
- GraphQL servers
- Chat applications
- Streaming
- WebSockets
- Reverse proxies

---

## 2. High concurrency with lower memory usage

Many traditional server platforms allocate one thread per request (or maintain large thread pools).

Threads consume:

- stack memory
- context switching overhead
- scheduler time

Node instead handles most concurrent work using one event loop plus a small worker pool for operations like filesystem access, DNS, compression, and crypto.

Benefits:

- Less RAM
- Fewer context switches
- Higher throughput for I/O-heavy applications

For CPU-heavy workloads, Node can use:

- `worker_threads`
- `cluster`
- multiple containers/pods

---

## 3. Excellent performance for I/O-heavy workloads

Node uses:

- Google's V8 JavaScript engine
- JIT compilation
- optimized event loop
- asynchronous networking

Because web applications spend much of their time waiting on databases or remote APIs, Node can often achieve very good throughput with modest hardware.

Examples:

- API Gateway
- Authentication server
- Notification service
- File upload service
- WebSocket server

---

## 4. JavaScript/TypeScript everywhere

One language across:

- frontend
- backend
- serverless
- build tools
- testing
- automation

Benefits:

- Easier hiring
- Shared models
- Shared validation logic
- Shared utility libraries
- Reduced context switching

Example:

```text
React
   │
Shared Types (TypeScript)
   │
Express / Fastify
```

This improves developer productivity and reduces duplicated code.

---

## 5. Massive npm ecosystem

Node has one of the largest open-source package ecosystems.

Examples:

- Express
- Fastify
- NestJS
- Prisma
- TypeORM
- Zod
- OpenTelemetry
- Jest
- Vitest

Advantages:

- Faster development
- Mature libraries
- Community support
- Frequent updates

A trade-off is that you should carefully evaluate dependencies for security, maintenance, and supply-chain risk.

---

## 6. Fast development speed

Node emphasizes:

- simple APIs
- JSON everywhere
- async programming
- quick startup
- lightweight frameworks

This makes rapid prototyping and iterative development straightforward.

---

## 7. Excellent for microservices

Node services typically:

- start quickly
- consume modest memory
- expose HTTP/gRPC APIs
- containerize easily

This works well with Kubernetes and serverless platforms.

---

## 8. Strong support for real-time applications

Node is particularly good for:

- WebSockets
- Server-Sent Events (SSE)
- live dashboards
- collaborative editing
- multiplayer games
- chat systems

Keeping many open connections is one of Node's strengths.

---

## 9. Great streaming capabilities

Streams avoid loading entire files into memory.

Example use cases:

- video streaming
- file uploads
- compression
- proxy servers
- ETL pipelines

Streams improve:

- latency
- memory usage
- scalability

---

## 10. Cross-platform support

Node runs consistently on:

- Linux
- Windows
- macOS
- Docker
- Cloud platforms
- Serverless environments

The runtime and APIs are largely consistent across operating systems.

---

## When Node.js is **not** the best choice

Node is less suitable for workloads dominated by CPU-bound computation, such as:

- image/video processing
- scientific computing
- machine learning training
- heavy cryptography
- large numerical simulations

In these cases, consider:

- Worker Threads
- external processing services
- languages like Go, Rust, Java, or C++ depending on the workload

---

# Example (JavaScript)

```javascript
// server.js
import http from "node:http";
import { setTimeout as delay } from "node:timers/promises";

const server = http.createServer(async (_req, res) => {
  // Simulate asynchronous I/O (e.g., DB or remote API)
  await delay(100);

  res.writeHead(200, { "Content-Type": "application/json" });
  res.end(
    JSON.stringify({ message: "Handled without blocking the event loop" }),
  );
});

server.listen(3000, () => {
  console.log("Server listening on http://localhost:3000");
});
```

Even while one request is awaiting the simulated I/O, the event loop can continue accepting and processing other incoming requests.

---

# Testing

**Unit testing**

- Mock asynchronous dependencies (database clients, HTTP calls).
- Verify request handlers and business logic independently.

**Integration testing**

- Start the server and make HTTP requests using tools like `supertest` or the built-in `fetch` API.

Example using Node's built-in test runner:

```bash
node --test
```

---

# Ops & Monitoring

- **Logging:** Use structured JSON logging with libraries like `pino`.
- **Metrics:** Export Prometheus metrics (request rate, latency, event loop lag, memory usage).
- **Tracing:** Instrument services with OpenTelemetry for distributed tracing.
- **Error handling:** Handle rejected promises centrally, return consistent error responses, and shut down gracefully on fatal errors.
- **Process management:** Run in containers or under `systemd`; use PM2 primarily when not orchestrated by a container platform.

---

# Deployment & Scaling

- Use the current **Active LTS** version of Node.js in production.
- Build small container images (e.g., Debian slim or Alpine where compatible with native modules).
- Scale horizontally behind a load balancer instead of relying on a single process.
- Reuse database connections via connection pooling to reduce latency and resource usage.
- For serverless deployments, minimize cold starts by reducing dependencies and performing initialization lazily.
- Use `worker_threads` for CPU-intensive work instead of blocking the event loop.

---

# Pitfalls

- **Don't block the event loop** with CPU-intensive or synchronous APIs (`fs.readFileSync`, expensive loops) in request handlers.
- **Avoid excessive npm dependencies**; audit packages regularly and keep them updated to reduce security and maintenance risks.
- **Use streams for large payloads** instead of buffering entire files or responses into memory.

## Question 2. Can Node.js be used for desktop applications? If yes, how?

## Question 3. Explain how Node.js handles concurrency

## Question 4. What is the difference between Node.js runtime and the Node.js library?

## Question 5. What are the core modules in Node.js? Give examples

## Question 6. What is the difference between CommonJS and ES Modules in Node.js?

## Question 7. How do you import only a specific function from a module in Node.js?

## Question 8. What is the difference between local and global Node.js packages?

## Question 9. Explain `npm install` vs `npm ci`

## Question 10. How does Node.js handle file system operations?

## Question 11. How do you check for a file's existence in Node.js?

## Question 12. How do you append data to a file asynchronously in Node.js?

## Question 13. Explain the difference between `http` and `https` modules in Node.js

## Question 14. How do you parse URL parameters in Node.js?

## Question 15. How do you parse JSON data in Node.js?

## Question 16. How do you convert a JSON object to a string in Node.js?

## Question 17. What is the `os` module in Node.js? (implied from content)

## Question 18. How can you get the system memory or CPU information in Node.js?

## Question 19. What are timers in Node.js?

## Question 20. How do you clear a timeout or interval in Node.js?
