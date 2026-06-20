# Set 2

| S.No. | Question                                                                                                                                                |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What are buffers in Node.js?](#question-1-what-are-buffers-in-nodejs)                                                                                  |
| 2.    | [How do you handle errors in Node.js?](#question-2-how-do-you-handle-errors-in-nodejs)                                                                  |
| 3.    | [What is the difference between `throw` and `return` in error handling?](#question-3-what-is-the-difference-between-throw-and-return-in-error-handling) |
| 4.    | [What is the difference between `__dirname` and `__filename`?](#question-4-what-is-the-difference-between-__dirname-and-__filename)                     |
| 5.    | [How do you set environment variables in Node.js?](#question-5-how-do-you-set-environment-variables-in-nodejs)                                          |
| 6.    | [Explain the difference between `fs.readFile` and `fs.readFileSync`](#question-6-explain-the-difference-between-fsreadfile-and-fsreadfilesync)          |
| 7.    | [What is the difference between Node.js `require()` and ES6 `import`?](#question-7-what-is-the-difference-between-nodejs-require-and-es6-import)        |
| 8.    | [What is the difference between `setTimeout()` and `setImmediate()`?](#question-8-what-is-the-difference-between-settimeout-and-setimmediate)           |
| 9.    | [Explain the difference between `process.nextTick()` and `setImmediate()`](#question-9-explain-the-difference-between-processnexttick-and-setimmediate) |
| 10.   | [How do you debug a Node.js application?](#question-10-how-do-you-debug-a-nodejs-application)                                                           |
| 11.   | [How do you handle uncaught exceptions in Node.js?](#question-11-how-do-you-handle-uncaught-exceptions-in-nodejs)                                       |
| 12.   | [What is REPL in Node.js?](#question-12-what-is-repl-in-nodejs)                                                                                         |
| 13.   | [How do you check the Node.js version installed on your system?](#question-13-how-do-you-check-the-nodejs-version-installed-on-your-system)             |
| 14.   | [What is middleware in Node.js?](#question-14-what-is-middleware-in-nodejs)                                                                             |
| 15.   | [Explain the difference between Node.js and Express.js](#question-15-explain-the-difference-between-nodejs-and-expressjs)                               |
| 16.   | [What are streams in Node.js?](#question-16-what-are-streams-in-nodejs)                                                                                 |
| 17.   | [What is the difference between streams and buffers?](#question-17-what-is-the-difference-between-streams-and-buffers)                                  |
| 18.   | [Explain piping in Node.js streams](#question-18-explain-piping-in-nodejs-streams)                                                                      |
| 19.   | [What is the purpose of `EventEmitter` in Node.js?](#question-19-what-is-the-purpose-of-eventemitter-in-nodejs)                                         |
| 20.   | [How do you create a custom event using `EventEmitter`?](#question-20-how-do-you-create-a-custom-event-using-eventemitter)                              |

## Question 1. What are buffers in Node.js?

# Short answer

A **Buffer** in Node.js is a fixed-size block of raw binary memory used to store and manipulate data outside the JavaScript heap. Buffers are essential for handling streams, files, network packets, cryptographic data, and any binary protocol efficiently.

---

# Explanation

JavaScript in browsers primarily works with strings, objects, and typed arrays. Node.js, however, frequently interacts with operating system resources such as files, TCP sockets, HTTP bodies, and databases, where data arrives as **bytes** rather than JavaScript strings.

Node.js provides the global `Buffer` class to efficiently represent binary data.

## Why Buffers exist

When reading a file, receiving a network packet, or processing a stream:

- The OS delivers raw bytes.
- Converting everything immediately into strings would be inefficient.
- Some data (images, videos, compressed files, encrypted payloads) isn't text at all.

Buffers allow Node.js to:

- Store binary data efficiently.
- Avoid unnecessary encoding/decoding.
- Interoperate directly with C/C++ code and libuv.
- Handle streaming workloads with low memory overhead.

## Internal architecture

A Buffer:

- Extends `Uint8Array`.
- Allocates memory outside the V8 JavaScript heap.
- Is backed by native memory managed by Node.js.
- Can be passed between networking, filesystem, crypto, and stream APIs without conversion.

```text
┌──────────────┐
│ OS / Kernel  │
└──────┬───────┘
       │ bytes
       ▼
┌──────────────┐
│ libuv        │
└──────┬───────┘
       ▼
┌──────────────┐
│ Buffer       │
│ Raw Memory   │
└──────┬───────┘
       ▼
 JavaScript
```

## Common Buffer operations

### Create a Buffer

```js
const buf = Buffer.from("hello");
```

### Allocate memory

```js
const buf = Buffer.alloc(1024);
```

### Convert to string

```js
console.log(buf.toString("utf8"));
```

### Access bytes

```js
console.log(buf[0]);
```

### Concatenate

```js
const combined = Buffer.concat([buf1, buf2]);
```

---

## Performance implications

### Good

- Efficient binary handling.
- Reduced string conversions.
- Ideal for streams and network I/O.
- Supports zero-copy slices in many cases.

### Potential issues

Large Buffers can increase memory usage because they are allocated outside the V8 heap.

Example:

```js
Buffer.alloc(500 * 1024 * 1024);
```

This allocates ~500 MB immediately.

### Buffer pooling

Node internally uses memory pools for small Buffer allocations to reduce:

- malloc/free calls
- fragmentation
- GC pressure

This improves throughput for high-volume networking applications.

---

## Encoding considerations

Buffers are bytes; strings require an encoding.

Common encodings:

| Encoding | Use Case           |
| -------- | ------------------ |
| utf8     | Default text       |
| ascii    | Legacy protocols   |
| base64   | APIs, JWTs, images |
| hex      | Binary debugging   |

Example:

```js
const text = Buffer.from("Node.js").toString("base64");
console.log(text);
```

---

# Example

**JavaScript (Node.js 18+)**

Reading a file as a Buffer and converting it to text:

```js
import { readFile } from "node:fs/promises";

async function main() {
  const buffer = await readFile("./message.txt");

  console.log("Buffer length:", buffer.length);
  console.log("First byte:", buffer[0]);
  console.log("Content:", buffer.toString("utf8"));
}

main().catch(console.error);
```

Output:

```text
Buffer length: 12
First byte: 72
Content: Hello World
```

---

# Testing

### Unit testing

Verify:

- Correct encoding conversions
- Buffer sizes
- Binary transformations

Using Node's built-in test runner:

```js
import test from "node:test";
import assert from "node:assert";

test("buffer conversion", () => {
  const buf = Buffer.from("hello");
  assert.equal(buf.toString(), "hello");
});
```

Run:

```bash
node --test
```

### Integration testing

Test:

- File uploads/downloads
- HTTP request bodies
- Stream pipelines
- Binary protocol communication

---

# Ops & Monitoring

### Logging

Avoid logging entire Buffers:

```js
logger.info({ size: buffer.length });
```

instead of:

```js
logger.info(buffer);
```

### Metrics

Track:

- Upload/download sizes
- Stream throughput
- Memory consumption
- Buffer allocation rates

### Tracing

With [OpenTelemetry](https://opentelemetry.io/?utm_source=chatgpt.com), trace:

- File operations
- HTTP payload processing
- Stream pipelines

### Error handling

Validate buffer lengths before parsing:

```js
if (buffer.length < expectedLength) {
  throw new Error("Incomplete packet");
}
```

---

# Deployment & Scaling

### Containers

Set memory limits carefully:

```bash
docker run --memory=1g app
```

Buffers live outside the V8 heap, so monitor total process RSS, not just heap usage.

### High-throughput services

Prefer:

- Streams
- Chunked processing
- Backpressure-aware pipelines

instead of:

```js
await fs.readFile("5gb-file.bin");
```

### Scaling

For large binary workloads:

- Use streaming uploads/downloads.
- Keep payloads off memory when possible.
- Consider worker threads for CPU-heavy binary transformations.

### Node versions

Modern Buffer APIs are stable in all supported Node.js versions (Node 18+ recommended in production).

---

# Pitfalls

- **Using `Buffer.allocUnsafe()` without overwriting memory** can expose old memory contents.
- **Loading huge files into memory** instead of streaming them can cause OOM errors.
- **Assuming binary data is UTF-8 text** can corrupt data during encoding conversion.

## Question 2. How do you handle errors in Node.js?

## Question 3. What is the difference between `throw` and `return` in error handling?

## Question 4. What is the difference between `__dirname` and `__filename`?

## Question 5. How do you set environment variables in Node.js?

## Question 6. Explain the difference between `fs.readFile` and `fs.readFileSync`

## Question 7. What is the difference between Node.js `require()` and ES6 `import`?

## Question 8. What is the difference between `setTimeout()` and `setImmediate()`?

## Question 9. Explain the difference between `process.nextTick()` and `setImmediate()`

## Question 10. How do you debug a Node.js application?

## Question 11. How do you handle uncaught exceptions in Node.js?

## Question 12. What is REPL in Node.js?

## Question 13. How do you check the Node.js version installed on your system?

## Question 14. What is middleware in Node.js?

## Question 15. Explain the difference between Node.js and Express.js

## Question 16. What are streams in Node.js?

## Question 17. What is the difference between streams and buffers?

## Question 18. Explain piping in Node.js streams

## Question 19. What is the purpose of `EventEmitter` in Node.js?

## Question 20. How do you create a custom event using `EventEmitter`?
