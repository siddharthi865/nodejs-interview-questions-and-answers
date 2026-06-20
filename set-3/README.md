# Set 3

| S.No. | Question                                                                                                                                                                 |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [Explain the Node.js event loop in detail](#question-1-explain-the-nodejs-event-loop-in-detail)                                                                          |
| 2.    | [What is the difference between microtasks and macrotasks in Node.js?](#question-2-what-is-the-difference-between-microtasks-and-macrotasks-in-nodejs)                   |
| 3.    | [How do Promises work in Node.js?](#question-3-how-do-promises-work-in-nodejs)                                                                                           |
| 4.    | [Explain `async/await` in Node.js](#question-4-explain-asyncawait-in-nodejs)                                                                                             |
| 5.    | [What is the difference between `process.nextTick()` and `Promise.resolve().then()`?](#question-5-what-is-the-difference-between-processnexttick-and-promiseresolvethen) |
| 6.    | [What are child processes in Node.js?](#question-6-what-are-child-processes-in-nodejs)                                                                                   |
| 7.    | [Explain the difference between `fork()` and `spawn()` in Node.js child processes](#question-7-explain-the-difference-between-fork-and-spawn-in-nodejs-child-processes)  |
| 8.    | [How do you create a simple REST API using Node.js?](#question-8-how-do-you-create-a-simple-rest-api-using-nodejs)                                                       |
| 9.    | [How do you handle CORS in Node.js?](#question-9-how-do-you-handle-cors-in-nodejs)                                                                                       |
| 10.   | [What is the difference between `app.use()` and `app.get()` in Express.js?](#question-10-what-is-the-difference-between-appuse-and-appget-in-expressjs)                  |
| 11.   | [Explain the role of `next()` in Express middleware](#question-11-explain-the-role-of-next-in-express-middleware)                                                        |
| 12.   | [How do you handle file uploads in Node.js?](#question-12-how-do-you-handle-file-uploads-in-nodejs)                                                                      |
| 13.   | [What is JWT and how is it used in Node.js authentication?](#question-13-what-is-jwt-and-how-is-it-used-in-nodejs-authentication)                                        |
| 14.   | [How do you connect Node.js with MongoDB?](#question-14-how-do-you-connect-nodejs-with-mongodb)                                                                          |
| 15.   | [How do you handle database errors in Node.js?](#question-15-how-do-you-handle-database-errors-in-nodejs)                                                                |
| 16.   | [How do you implement caching in Node.js?](#question-16-how-do-you-implement-caching-in-nodejs)                                                                          |
| 17.   | [How do you prevent memory leaks in Node.js?](#question-17-how-do-you-prevent-memory-leaks-in-nodejs)                                                                    |
| 18.   | [Explain `cluster` module in Node.js](#question-18-explain-cluster-module-in-nodejs)                                                                                     |
| 19.   | [How do you scale a Node.js application horizontally?](#question-19-how-do-you-scale-a-nodejs-application-horizontally)                                                  |
| 20.   | [What is the difference between PM2 and Node.js built-in `cluster`?](#question-20-what-is-the-difference-between-pm2-and-nodejs-built-in-cluster)                        |

## Question 1. Explain the Node.js event loop in detail

# Short answer

The **Node.js event loop** is the mechanism that allows Node.js to handle many concurrent I/O operations using a single JavaScript thread. It continuously processes callbacks through a series of phases managed by **libuv**, while offloading I/O, networking, file system operations, and some CPU-intensive tasks to the operating system or a worker thread pool.

---

# Explanation

## High-level architecture

When a Node.js application runs:

1. JavaScript executes on the **main thread**.
2. The **V8 engine** executes JS code.
3. **libuv** provides the event loop, thread pool, and OS abstractions.
4. Expensive or asynchronous operations are delegated:
   - Network sockets → OS kernel
   - File system operations → libuv thread pool
   - DNS lookups (some types) → libuv thread pool
   - Crypto/compression → libuv thread pool

Once work completes, callbacks are queued and eventually executed by the event loop.

```text
┌───────────────┐
│ JavaScript    │
│ Main Thread   │
└───────┬───────┘
        │
        ▼
┌───────────────┐
│ Event Loop    │
│ (libuv)       │
└───────┬───────┘
        │
 ┌──────┼─────────────┐
 ▼      ▼             ▼
OS I/O  Thread Pool   Timers
```

---

## Why Node.js needs an event loop

Traditional thread-per-request servers create one thread per client.

```text
Request 1 → Thread 1
Request 2 → Thread 2
Request 3 → Thread 3
...
```

This consumes significant memory and CPU context-switching overhead.

Node.js instead uses:

```text
Many Requests
      │
      ▼
 Single Event Loop
      │
      ▼
 Non-blocking I/O
```

This makes Node particularly efficient for:

- APIs
- Real-time systems
- Streaming
- Proxies
- WebSockets

Less ideal for:

- Heavy CPU calculations
- Image/video processing
- Machine learning workloads

unless using Worker Threads.

---

## Event Loop Phases

The event loop operates in a sequence of phases.

```text
┌─────────────┐
│ Timers      │
└──────┬──────┘
       ▼
┌─────────────┐
│ Pending     │
│ Callbacks   │
└──────┬──────┘
       ▼
┌─────────────┐
│ Idle/Prepare│
└──────┬──────┘
       ▼
┌─────────────┐
│ Poll        │
└──────┬──────┘
       ▼
┌─────────────┐
│ Check       │
└──────┬──────┘
       ▼
┌─────────────┐
│ Close       │
└─────────────┘
```

---

## 1. Timers Phase

Executes callbacks scheduled by:

```js
setTimeout();
setInterval();
```

Example:

```js
setTimeout(() => {
  console.log("timer");
}, 1000);
```

Important:

`setTimeout(1000)` means:

> Execute after **at least** 1000ms.

It does not guarantee exact timing.

The callback executes only when:

- Timer expires
- Event loop reaches timer phase

---

## 2. Pending Callbacks Phase

Executes deferred system-level callbacks.

Examples:

- TCP errors
- Some network-related callbacks

Most application developers rarely interact with this phase directly.

---

## 3. Idle / Prepare Phase

Internal libuv phase.

Used for internal bookkeeping before entering Poll.

Not exposed to user code.

---

## 4. Poll Phase (Most Important)

This is the heart of the event loop.

Responsibilities:

### Execute I/O callbacks

```js
fs.readFile("file.txt", () => {
  console.log("file read");
});
```

### Wait for new I/O

If nothing else is pending, Node may block here waiting for:

- Network requests
- Database responses
- File operations

### Determine next phase

The Poll phase decides whether to:

- Continue processing I/O
- Move to Check phase
- Return to Timers phase

---

## 5. Check Phase

Executes callbacks registered by:

```js
setImmediate();
```

Example:

```js
setImmediate(() => {
  console.log("immediate");
});
```

This phase runs immediately after Poll.

---

## 6. Close Callbacks Phase

Handles closed resources.

Example:

```js
socket.on("close", () => {
  console.log("closed");
});
```

Typical use cases:

- TCP connections
- Streams
- Sockets

---

# Microtasks vs Event Loop Phases

Senior interviews often focus here.

Node.js maintains separate queues:

### process.nextTick Queue

Highest priority.

```js
process.nextTick(() => {
  console.log("nextTick");
});
```

### Promise Microtask Queue

```js
Promise.resolve().then(() => {
  console.log("promise");
});
```

Both execute **before moving to the next event-loop phase**.

Priority:

```text
Current JS
   ↓
process.nextTick()
   ↓
Promise Microtasks
   ↓
Event Loop Phase
```

---

## Execution Order Example

```js
setTimeout(() => console.log("timeout"), 0);

setImmediate(() => console.log("immediate"));

Promise.resolve().then(() => {
  console.log("promise");
});

process.nextTick(() => {
  console.log("nextTick");
});

console.log("sync");
```

Output:

```text
sync
nextTick
promise
timeout/immediate (order may vary)
```

Key point:

```text
process.nextTick
   >
Promise.then
   >
Timers / I/O
```

---

## setTimeout vs setImmediate

A very common interview question.

### Outside I/O

```js
setTimeout(() => console.log("timeout"), 0);
setImmediate(() => console.log("immediate"));
```

Order is not guaranteed.

---

### Inside I/O

```js
import fs from "node:fs";

fs.readFile(__filename, () => {
  setTimeout(() => console.log("timeout"), 0);

  setImmediate(() => {
    console.log("immediate");
  });
});
```

Output:

```text
immediate
timeout
```

Why?

After Poll completes, the event loop enters Check phase before returning to Timers.

---

## libuv Thread Pool

Not all async operations are truly non-blocking.

Examples:

```js
fs.readFile();
crypto.pbkdf2();
zlib.gzip();
dns.lookup();
```

These use libuv's thread pool.

Default size:

```bash
UV_THREADPOOL_SIZE=4
```

Can be increased:

```bash
UV_THREADPOOL_SIZE=16 node app.js
```

Be cautious:

- Too small → queueing delays
- Too large → CPU contention

---

## Event Loop Blocking

The biggest production mistake.

Bad:

```js
app.get("/users", (req, res) => {
  while (true) {}
});
```

This blocks:

- Requests
- Timers
- Promises
- Everything

Another example:

```js
const data = fs.readFileSync("huge-file.txt");
```

Synchronous APIs block the event loop.

---

## Handling CPU-Intensive Work

Use Worker Threads.

```js
import { Worker } from "node:worker_threads";

new Worker("./worker.js");
```

Good for:

- Image processing
- PDF generation
- Encryption
- ML inference
- Large calculations

---

# Example

**JavaScript (Node.js 20+)**

```js
import fs from "node:fs";

console.log("start");

setTimeout(() => {
  console.log("timer");
}, 0);

setImmediate(() => {
  console.log("immediate");
});

Promise.resolve().then(() => {
  console.log("promise");
});

process.nextTick(() => {
  console.log("nextTick");
});

fs.readFile(__filename, () => {
  console.log("file read");
});

console.log("end");
```

Typical output:

```text
start
end
nextTick
promise
timer
immediate
file read
```

(Some ordering between timer/immediate/file-read may vary depending on timing and platform.)

---

# Testing

### Unit Testing

Verify ordering of microtasks and timers.

```js
import test from "node:test";
import assert from "node:assert";

test("microtask order", async () => {
  const result = [];

  process.nextTick(() => result.push("nextTick"));

  Promise.resolve().then(() => result.push("promise"));

  await new Promise(setImmediate);

  assert.deepStrictEqual(result, ["nextTick", "promise"]);
});
```

Run:

```bash
node --test
```

### Integration Testing

- Test HTTP latency under load.
- Verify event-loop responsiveness.
- Simulate slow I/O and CPU-heavy tasks.

Tools:

- autocannon
- k6
- Artillery

---

# Ops & Monitoring

Monitor event-loop health in production.

### Event Loop Delay

```js
import { monitorEventLoopDelay } from "node:perf_hooks";

const histogram = monitorEventLoopDelay();
histogram.enable();
```

Track:

- p95
- p99
- max latency

### Metrics

Useful metrics:

- Event loop lag
- Active handles
- Heap usage
- GC pauses
- Thread pool saturation

### Tracing

Use:

- [OpenTelemetry Node.js](https://opentelemetry.io/docs/languages/js/?utm_source=chatgpt.com)

### Logging

Prefer structured logging:

- [Pino](https://getpino.io/?utm_source=chatgpt.com#/)
- [Winston](https://github.com/winstonjs/winston?utm_source=chatgpt.com)

---

# Deployment & Scaling

### Horizontal Scaling

Use:

- Multiple containers
- Kubernetes
- Load balancer

Rather than relying solely on:

```js
cluster;
```

Worker Threads solve CPU parallelism; containers solve process scaling.

### Connection Pooling

Database drivers should use pools:

```text
Node Process
      ↓
DB Pool (10-50 connections)
      ↓
Database
```

Avoid opening a connection per request.

### Serverless

Watch for:

- Cold starts
- Connection reuse
- Event loop draining

### Node Version

Prefer active LTS versions (Node 22+ as of 2026) for improved event-loop and V8 performance.

---

# Pitfalls

- Overusing `process.nextTick()` can starve the event loop and prevent I/O from running.
- Using synchronous APIs (`readFileSync`, CPU-heavy loops) blocks all requests.
- Increasing `UV_THREADPOOL_SIZE` excessively can reduce performance due to context switching.

## Question 2. What is the difference between microtasks and macrotasks in Node.js?

## Question 3. How do Promises work in Node.js?

## Question 4. Explain `async/await` in Node.js

## Question 5. What is the difference between `process.nextTick()` and `Promise.resolve().then()`?

## Question 6. What are child processes in Node.js?

## Question 7. Explain the difference between `fork()` and `spawn()` in Node.js child processes

## Question 8. How do you create a simple REST API using Node.js?

## Question 9. How do you handle CORS in Node.js?

## Question 10. What is the difference between `app.use()` and `app.get()` in Express.js?

## Question 11. Explain the role of `next()` in Express middleware

## Question 12. How do you handle file uploads in Node.js?

## Question 13. What is JWT and how is it used in Node.js authentication?

## Question 14. How do you connect Node.js with MongoDB?

## Question 15. How do you handle database errors in Node.js?

## Question 16. How do you implement caching in Node.js?

## Question 17. How do you prevent memory leaks in Node.js?

## Question 18. Explain `cluster` module in Node.js

## Question 19. How do you scale a Node.js application horizontally?

## Question 20. What is the difference between PM2 and Node.js built-in `cluster`?
