# Set 1

| S.No. | Question                                                                                                                                                                                                         |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is Node.js and how does it differ from JavaScript in the browser?](#question-1-what-is-nodejs-and-how-does-it-differ-from-javascript-in-the-browser)                                                       |
| 2.    | [What is the difference between Node.js and traditional server-side technologies like PHP or Java?](#question-2-what-is-the-difference-between-nodejs-and-traditional-server-side-technologies-like-php-or-java) |
| 3.    | [Explain the event-driven architecture of Node.js](#question-3-explain-the-event-driven-architecture-of-nodejs)                                                                                                  |
| 4.    | [What is the V8 engine in Node.js?](#question-4-what-is-the-v8-engine-in-nodejs)                                                                                                                                 |
| 5.    | [What is npm and how does it work?](#question-5-what-is-npm-and-how-does-it-work)                                                                                                                                |
| 6.    | [How do you install a Node.js package locally and globally?](#question-6-how-do-you-install-a-nodejs-package-locally-and-globally)                                                                               |
| 7.    | [What are Node.js modules?](#question-7-what-are-nodejs-modules)                                                                                                                                                 |
| 8.    | [Explain `require()` and `module.exports` in Node.js](#question-8-explain-require-and-moduleexports-in-nodejs)                                                                                                   |
| 9.    | [What is the difference between `exports` and `module.exports`?](#question-9-what-is-the-difference-between-exports-and-moduleexports)                                                                           |
| 10.   | [How do you create a simple HTTP server in Node.js?](#question-10-how-do-you-create-a-simple-http-server-in-nodejs)                                                                                              |
| 11.   | [What is Asynchronous Programming?](#question-11-what-is-asynchronous-programming)                                                                                                                               |
| 12.   | [What is a callback function in Node.js?](#question-12-what-is-a-callback-function-in-nodejs)                                                                                                                    |
| 13.   | [What are the disadvantages of using callbacks?](#question-13-what-are-the-disadvantages-of-using-callbacks)                                                                                                     |
| 14.   | [What is the difference between synchronous and asynchronous functions?](#question-14-what-is-the-difference-between-synchronous-and-asynchronous-functions)                                                     |
| 15.   | [How do you read a file asynchronously in Node.js?](#question-15-how-do-you-read-a-file-asynchronously-in-nodejs)                                                                                                |
| 16.   | [What is the `package.json` file? What is its purpose?](#question-16-what-is-the-packagejson-file-what-is-its-purpose)                                                                                           |
| 17.   | [Explain the difference between `dependencies` and `devDependencies` in `package.json`](#question-17-explain-the-difference-between-dependencies-and-devdependencies-in-packagejson)                             |
| 18.   | [What is the `process` object in Node.js?](#question-18-what-is-the-process-object-in-nodejs)                                                                                                                    |
| 19.   | [How can you access command-line arguments in Node.js?](#question-19-how-can-you-access-command-line-arguments-in-nodejs)                                                                                        |
| 20.   | [What is the global object in Node.js?](#question-20-what-is-the-global-object-in-nodejs)                                                                                                                        |

## Question 1. What is Node.js and how does it differ from JavaScript in the browser?

## Short answer

**Node.js** is a JavaScript runtime built on Google's **V8 engine** that allows JavaScript to run outside the browser, primarily for server-side applications, APIs, CLI tools, automation, and backend services.

The key difference is:

- **JavaScript in the browser** runs inside a browser environment and has access to browser APIs such as `window`, `document`, `localStorage`, and the DOM.
- **Node.js** runs on the operating system and provides server-side APIs such as `fs`, `http`, `net`, `stream`, `worker_threads`, and process management, but it does **not** have access to the DOM.

---

## Explanation

### JavaScript vs Node.js

A common interview misconception is treating Node.js and JavaScript as the same thing.

- **JavaScript** is the programming language (ECMAScript specification).
- **Node.js** is a runtime environment that executes JavaScript outside the browser.

Think of it this way:

```text
JavaScript = Language
Node.js     = Runtime + APIs + Event Loop + libuv + V8
```

### Browser Runtime

In browsers:

```javascript
console.log(window.location.href);
```

works because the browser exposes the `window` object.

Browser responsibilities include:

- Rendering HTML/CSS
- DOM manipulation
- User interaction
- Web APIs (Fetch, WebSocket, Storage)
- Security sandboxing

### Node.js Runtime

Node.js provides:

- File system access
- Networking
- TCP/UDP sockets
- HTTP servers
- Process management
- Streams
- Worker threads

Example:

```javascript
import { readFile } from "node:fs/promises";

const content = await readFile("./file.txt", "utf8");
console.log(content);
```

This is impossible in normal browser JavaScript because browsers cannot freely access local files.

---

### Architecture Differences

#### Browser

```text
JavaScript
     ↓
Browser Engine
     ↓
DOM + Web APIs
```

#### Node.js

```text
JavaScript
     ↓
V8 Engine
     ↓
Node.js Runtime
     ↓
libuv
     ↓
OS Kernel
```

### Role of libuv

A senior-level interview point:

Node.js achieves non-blocking I/O through **libuv**, which provides:

- Event loop
- Thread pool
- File I/O handling
- DNS operations
- Networking abstractions

Example:

```javascript
await fs.readFile(...)
```

The file read is delegated to libuv, allowing the main thread to continue processing other requests.

This is one reason Node.js scales efficiently for I/O-heavy workloads.

---

### Global Objects Comparison

| Feature        | Browser | Node.js             |
| -------------- | ------- | ------------------- |
| `window`       | ✅      | ❌                  |
| `document`     | ✅      | ❌                  |
| `localStorage` | ✅      | ❌                  |
| `fetch()`      | ✅      | ✅ (modern Node.js) |
| `process`      | ❌      | ✅                  |
| `fs`           | ❌      | ✅                  |
| `http`         | ❌      | ✅                  |
| DOM APIs       | ✅      | ❌                  |

Modern Node.js (v18+) includes many browser-like APIs such as:

- `fetch`
- `AbortController`
- `URL`
- `TextEncoder`

which reduces differences, but the runtime environments remain fundamentally different.

---

## Example

**JavaScript (Node.js, ESM, Node 18+)**

```javascript
import http from "node:http";

const server = http.createServer((req, res) => {
  res.writeHead(200, { "content-type": "application/json" });
  res.end(JSON.stringify({ runtime: "Node.js" }));
});

server.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

Why this demonstrates the difference:

- Uses Node-specific `http` module.
- Creates a TCP-based web server.
- Cannot run directly in a browser environment.

---

## Testing

### Unit Testing

Use Node's built-in test runner:

```javascript
import test from "node:test";
import assert from "node:assert";

test("runtime check", () => {
  assert.ok(process.version.startsWith("v"));
});
```

Run:

```bash
node --test
```

### Integration Testing

- Start the server.
- Use `fetch()` or `supertest` to verify endpoints.
- Test filesystem and database interactions separately.

---

## Ops & Monitoring

### Logging

Use structured JSON logging:

```javascript
logger.info({
  requestId,
  userId,
  action: "create-order",
});
```

Popular tools:

- [Pino](https://getpino.io/?utm_source=chatgpt.com#/)
- [Winston](https://github.com/winstonjs/winston?utm_source=chatgpt.com)

### Metrics

Track:

- Event loop lag
- Memory usage
- Request latency
- Error rate

### Tracing

Use:

- [OpenTelemetry for Node.js](https://opentelemetry.io/docs/languages/js/?utm_source=chatgpt.com)

### Error Handling

Prefer:

```javascript
try {
  await service();
} catch (err) {
  logger.error(err);
}
```

Handle:

```javascript
process.on("uncaughtException");
process.on("unhandledRejection");
```

carefully and terminate gracefully when appropriate.

### Process Management

- Containers (preferred)
- PM2 for simple deployments
- systemd for VM-based environments

---

## Deployment & Scaling

### Horizontal Scaling

Node.js is single-threaded for JavaScript execution.

Scale via:

- Multiple containers/pods
- Load balancers
- `cluster` (less common in containerized environments)
- `worker_threads` for CPU-intensive work

### Connection Pooling

Always use pooling for:

- PostgreSQL
- MySQL
- Redis

Avoid creating a new connection per request.

### Serverless

Watch for:

- Cold starts
- Large dependency trees
- Database connection exhaustion

### Node Version

For modern production systems:

- Prefer active LTS releases.
- Use Node 20+ or newer supported LTS versions where possible.

---

## Pitfalls

- Blocking the event loop with CPU-heavy work (large loops, crypto, image processing).
- Assuming browser globals (`window`, `document`) exist in Node.js.
- Using synchronous APIs (`readFileSync`, `writeFileSync`) on request paths.

**Interview one-liner:**
_"JavaScript is the language, while Node.js is a server-side runtime built on V8 that provides operating-system capabilities, an event loop, and non-blocking I/O APIs that browsers do not expose."_

## Question 2. What is the difference between Node.js and traditional server-side technologies like PHP or Java?

## Question 3. Explain the event-driven architecture of Node.js

## Question 4. What is the V8 engine in Node.js?

## Question 5. What is npm and how does it work?

## Question 6. How do you install a Node.js package locally and globally?

## Question 7. What are Node.js modules?

## Question 8. Explain `require()` and `module.exports` in Node.js

## Question 9. What is the difference between `exports` and `module.exports`?

## Question 10. How do you create a simple HTTP server in Node.js?

## Question 11. What is Asynchronous Programming?

## Question 12. What is a callback function in Node.js?

## Question 13. What are the disadvantages of using callbacks?

## Question 14. What is the difference between synchronous and asynchronous functions?

## Question 15. How do you read a file asynchronously in Node.js?

## Question 16. What is the `package.json` file? What is its purpose?

## Question 17. Explain the difference between `dependencies` and `devDependencies` in `package.json`

## Question 18. What is the `process` object in Node.js?

## Question 19. How can you access command-line arguments in Node.js?

## Question 20. What is the global object in Node.js?
