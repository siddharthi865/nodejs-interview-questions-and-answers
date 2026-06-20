# Set 16

| S.No. | Question                                                                                                                                                                       |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [How do you check the memory usage of a Node.js process?](#question-1-how-do-you-check-the-memory-usage-of-a-nodejs-process)                                                   |
| 2.    | [How do you determine the Node.js process uptime?](#question-2-how-do-you-determine-the-nodejs-process-uptime)                                                                 |
| 3.    | [How do you list all running Node.js processes on a machine?](#question-3-how-do-you-list-all-running-nodejs-processes-on-a-machine)                                           |
| 4.    | [How do you terminate a Node.js process programmatically?](#question-4-how-do-you-terminate-a-nodejs-process-programmatically)                                                 |
| 5.    | [What is the purpose of the `console` module in Node.js?](#question-5-what-is-the-purpose-of-the-console-module-in-nodejs)                                                     |
| 6.    | [How do you use `console.time()` and `console.timeEnd()`?](#question-6-how-do-you-use-consoletime-and-consoletimeend)                                                          |
| 7.    | [How do you clear the console in Node.js?](#question-7-how-do-you-clear-the-console-in-nodejs)                                                                                 |
| 8.    | [How do you print objects and arrays in a readable format in Node.js?](#question-8-how-do-you-print-objects-and-arrays-in-a-readable-format-in-nodejs)                         |
| 9.    | [How do you handle exceptions thrown inside asynchronous callbacks?](#question-9-how-do-you-handle-exceptions-thrown-inside-asynchronous-callbacks)                            |
| 10.   | [How do you handle exceptions thrown inside Promises?](#question-10-how-do-you-handle-exceptions-thrown-inside-promises)                                                       |
| 11.   | [How do you handle exceptions inside `async/await`?](#question-11-how-do-you-handle-exceptions-inside-asyncawait)                                                              |
| 12.   | [How do you exit a Node.js process with a custom exit code?](#question-12-how-do-you-exit-a-nodejs-process-with-a-custom-exit-code)                                            |
| 13.   | [How do you check if a path points to a file or directory?](#question-13-how-do-you-check-if-a-path-points-to-a-file-or-directory)                                             |
| 14.   | [How do you read the content of a directory in Node.js?](#question-14-how-do-you-read-the-content-of-a-directory-in-nodejs)                                                    |
| 15.   | [How do you copy files in Node.js?](#question-15-how-do-you-copy-files-in-nodejs)                                                                                              |
| 16.   | [How do you watch multiple files for changes in Node.js?](#question-16-how-do-you-watch-multiple-files-for-changes-in-nodejs)                                                  |
| 17.   | [How do you schedule a function to run immediately after the current event loop?](#question-17-how-do-you-schedule-a-function-to-run-immediately-after-the-current-event-loop) |
| 18.   | [What is the difference between `process.exit()` and `process.kill()`?](#question-18-what-is-the-difference-between-processexit-and-processkill)                               |
| 19.   | [How do you listen to SIGINT and SIGTERM signals in Node.js?](#question-19-how-do-you-listen-to-sigint-and-sigterm-signals-in-nodejs)                                          |
| 20.   | [How do you generate random numbers securely in Node.js?](#question-20-how-do-you-generate-random-numbers-securely-in-nodejs)                                                  |

## Question 1. How do you check the memory usage of a Node.js process?

# Short answer

Use the built-in `process.memoryUsage()` API to inspect the current process's memory consumption. For deeper analysis, combine it with `process.memoryUsage.rss()`, `v8.getHeapStatistics()`, the `--inspect` flag, heap snapshots, and production monitoring tools (e.g., Prometheus/OpenTelemetry).

---

# Explanation

Node.js exposes several APIs to inspect memory usage, each serving a different purpose.

### 1. `process.memoryUsage()`

This returns an object containing memory metrics in bytes.

```js
{
  rss: 52326400,
  heapTotal: 20971520,
  heapUsed: 14567832,
  external: 1324567,
  arrayBuffers: 456789
}
```

Meaning of each field:

| Metric         | Description                                                                                                 |
| -------------- | ----------------------------------------------------------------------------------------------------------- |
| `rss`          | Resident Set Size – total memory allocated for the process (heap, stack, native modules, shared libraries). |
| `heapTotal`    | Total V8 heap allocated.                                                                                    |
| `heapUsed`     | Currently used portion of the V8 heap.                                                                      |
| `external`     | Memory allocated outside V8 but referenced by JS (e.g., native addons, `Buffer`s).                          |
| `arrayBuffers` | Memory used by `ArrayBuffer` and `SharedArrayBuffer`, including Buffers.                                    |

---

### 2. `process.memoryUsage.rss()`

If you only need RSS, Node provides a faster API.

```js
const rss = process.memoryUsage.rss();
```

This avoids collecting the other metrics.

---

### 3. Heap statistics

For V8-specific information:

```js
import v8 from "node:v8";

console.log(v8.getHeapStatistics());
```

Useful values include:

- heap size limit
- available heap
- used heap
- malloced memory

---

### Runtime behavior

Memory is managed by the V8 garbage collector.

Typical lifecycle:

1. Objects are allocated.
2. V8 places them in the Young Generation.
3. Surviving objects move to the Old Generation.
4. Garbage collection periodically frees unreachable objects.
5. `heapUsed` fluctuates after GC.

A steadily increasing `heapUsed` that never decreases often indicates a memory leak.

---

### Performance implications

Calling `process.memoryUsage()` occasionally (e.g., every 30–60 seconds) is inexpensive.

Avoid:

- calling it on every request
- forcing garbage collection (`global.gc()`) in production
- generating heap snapshots under heavy load

Heap snapshots pause the process and should primarily be used during debugging.

---

### Memory leak investigation

Useful tools include:

- Chrome DevTools (`node --inspect`)
- Heap snapshots
- Allocation timeline
- `clinic heapprof`
- `heapdump`
- `llnode` for postmortem debugging

A common production strategy is to alert when:

- RSS continually increases
- Heap usage exceeds ~70–80% of the V8 heap limit
- GC pause duration grows

---

# Example

**JavaScript (Node.js 20+)**

```javascript
function formatMB(bytes) {
  return `${(bytes / 1024 / 1024).toFixed(2)} MB`;
}

setInterval(() => {
  const mem = process.memoryUsage();

  console.log({
    rss: formatMB(mem.rss),
    heapTotal: formatMB(mem.heapTotal),
    heapUsed: formatMB(mem.heapUsed),
    external: formatMB(mem.external),
    arrayBuffers: formatMB(mem.arrayBuffers),
  });
}, 5000);
```

Example output:

```text
{
  rss: '72.51 MB',
  heapTotal: '28.00 MB',
  heapUsed: '17.62 MB',
  external: '2.13 MB',
  arrayBuffers: '1.54 MB'
}
```

---

# Testing

**Unit testing**

Abstract memory collection into a helper and mock `process.memoryUsage()`.

Example using the built-in test runner:

```bash
node --test
```

Example assertion:

```javascript
import test from "node:test";
import assert from "node:assert/strict";

test("memory metrics exist", () => {
  const mem = process.memoryUsage();

  assert.ok(mem.heapUsed > 0);
  assert.ok(mem.rss > 0);
});
```

**Integration testing**

- Run the application under sustained load.
- Verify memory stabilizes after garbage collection.
- Detect continuous RSS or heap growth across multiple GC cycles.

---

# Ops & Monitoring

- Export memory metrics to Prometheus (`process_resident_memory_bytes`, heap usage).
- Instrument the application with OpenTelemetry metrics for memory and GC events.
- Log periodic memory summaries instead of logging on every request.
- Configure alerts for:
  - high RSS
  - heap usage > 80%
  - increasing GC frequency

- Capture heap snapshots only during controlled debugging, as they pause the process.

---

# Deployment & Scaling

- Set an appropriate heap limit with `--max-old-space-size` based on container memory.
- Ensure container memory limits exceed the configured V8 heap to accommodate native memory and RSS.
- In clustered deployments, monitor memory per worker and restart workers showing persistent memory growth.
- In serverless environments, watch cold-start memory usage and optimize dependency size.
- Use Node.js 20+ (or newer LTS) for improved diagnostics and garbage collection behavior.

---

# Pitfalls

- Don't confuse **RSS** with **V8 heap**; RSS includes native memory, stacks, shared libraries, and Buffers.
- Rising `heapUsed` immediately after allocations is normal—look for sustained growth across garbage collection cycles before suspecting a leak.
- Large `Buffer`s increase `external` memory, so a stable heap does not necessarily mean overall process memory is stable.

## Question 2. How do you determine the Node.js process uptime?

## Question 3. How do you list all running Node.js processes on a machine?

## Question 4. How do you terminate a Node.js process programmatically?

## Question 5. What is the purpose of the `console` module in Node.js?

## Question 6. How do you use `console.time()` and `console.timeEnd()`?

## Question 7. How do you clear the console in Node.js?

## Question 8. How do you print objects and arrays in a readable format in Node.js?

## Question 9. How do you handle exceptions thrown inside asynchronous callbacks?

## Question 10. How do you handle exceptions thrown inside Promises?

## Question 11. How do you handle exceptions inside `async/await`?

## Question 12. How do you exit a Node.js process with a custom exit code?

## Question 13. How do you check if a path points to a file or directory?

## Question 14. How do you read the content of a directory in Node.js?

## Question 15. How do you copy files in Node.js?

## Question 16. How do you watch multiple files for changes in Node.js?

## Question 17. How do you schedule a function to run immediately after the current event loop?

## Question 18. What is the difference between `process.exit()` and `process.kill()`?

## Question 19. How do you listen to SIGINT and SIGTERM signals in Node.js?

## Question 20. How do you generate random numbers securely in Node.js?
