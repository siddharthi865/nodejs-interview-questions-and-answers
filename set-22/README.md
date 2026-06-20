# Set 22

| S.No. | Question                                                                                                                                                                   |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you get the current script filename using Node.js globals?](#question-1-how-do-you-get-the-current-script-filename-using-nodejs-globals)                           |
| 2.    | [How do you create a simple event listener using EventEmitter?](#question-2-how-do-you-create-a-simple-event-listener-using-eventemitter)                                  |
| 3.    | [How do you remove an event listener in Node.js?](#question-3-how-do-you-remove-an-event-listener-in-nodejs)                                                               |
| 4.    | [How do you check if an event has listeners attached?](#question-4-how-do-you-check-if-an-event-has-listeners-attached)                                                    |
| 5.    | [How do you handle synchronous errors inside event handlers?](#question-5-how-do-you-handle-synchronous-errors-inside-event-handlers)                                      |
| 6.    | [How do you create a simple TCP client?](#question-6-how-do-you-create-a-simple-tcp-client)                                                                                |
| 7.    | [How do you create a simple TCP server?](#question-7-how-do-you-create-a-simple-tcp-server)                                                                                |
| 8.    | [How do you implement echo functionality using a TCP server?](#question-8-how-do-you-implement-echo-functionality-using-a-tcp-server)                                      |
| 9.    | [How do you implement basic UDP message sending and receiving?](#question-9-how-do-you-implement-basic-udp-message-sending-and-receiving)                                  |
| 10.   | [How do you use Node.js REPL to evaluate code dynamically?](#question-10-how-do-you-use-nodejs-repl-to-evaluate-code-dynamically)                                          |
| 11.   | [How do you implement a file watcher for multiple directories using `chokidar`?](#question-11-how-do-you-implement-a-file-watcher-for-multiple-directories-using-chokidar) |
| 12.   | [How do you implement JSON schema validation in Node.js?](#question-12-how-do-you-implement-json-schema-validation-in-nodejs)                                              |
| 13.   | [How do you implement centralized error handling for API routes?](#question-13-how-do-you-implement-centralized-error-handling-for-api-routes)                             |
| 14.   | [How do you implement request logging with unique request IDs?](#question-14-how-do-you-implement-request-logging-with-unique-request-ids)                                 |
| 15.   | [How do you implement rate limiting per API key?](#question-15-how-do-you-implement-rate-limiting-per-api-key)                                                             |
| 16.   | [How do you implement in-memory caching using `lru-cache`?](#question-16-how-do-you-implement-in-memory-caching-using-lru-cache)                                           |
| 17.   | [How do you implement response compression selectively in Express.js?](#question-17-how-do-you-implement-response-compression-selectively-in-expressjs)                    |
| 18.   | [How do you implement streaming large responses from MongoDB?](#question-18-how-do-you-implement-streaming-large-responses-from-mongodb)                                   |
| 19.   | [How do you implement retry logic for failed HTTP requests using `axios-retry`?](#question-19-how-do-you-implement-retry-logic-for-failed-http-requests-using-axios-retry) |
| 20.   | [How do you implement JWT token refresh without invalidating old tokens?](#question-20-how-do-you-implement-jwt-token-refresh-without-invalidating-old-tokens)             |

## Question 1. How do you get the current script filename using Node.js globals?

# Short answer

In **CommonJS**, use the global variable `__filename` to get the absolute path of the current script.

In **ES Modules (ESM)**, `__filename` is **not available**. Instead, derive it from `import.meta.url` using `fileURLToPath()`.

---

# Explanation

### CommonJS (`.js` or `.cjs`)

Node.js automatically provides several globals to every CommonJS module:

- `__filename` → Absolute path of the current file.
- `__dirname` → Absolute path of the current directory.
- `module`
- `exports`
- `require`

Example:

```text
File: /home/app/server.js

__filename
=> /home/app/server.js
```

These values are injected by Node.js during module loading—they are **not true global variables** available everywhere in JavaScript.

### ES Modules (`.mjs` or `"type": "module"`)

ES Modules do **not** expose `__filename` or `__dirname`.

Instead:

1. Use `import.meta.url` to get the file URL.
2. Convert it to a filesystem path using `fileURLToPath()`.

This approach is portable across operating systems.

---

# Example

### JavaScript (CommonJS)

```javascript
console.log("Filename:", __filename);
console.log("Directory:", __dirname);
```

Output:

```text
Filename: /Users/john/projects/demo/index.js
Directory: /Users/john/projects/demo
```

### JavaScript (ESM)

```javascript
import { fileURLToPath } from "node:url";
import path from "node:path";

const __filename = fileURLToPath(import.meta.url);
const __dirname = path.dirname(__filename);

console.log(__filename);
console.log(__dirname);
```

---

# Testing

Unit test (using Node's built-in test runner):

```javascript
import test from "node:test";
import assert from "node:assert";

test("filename should be defined", () => {
  assert.ok(__filename.length > 0);
});
```

Run:

```bash
node --test
```

For integration testing, verify that the resolved paths match the expected project structure when the application is executed from different working directories.

---

# Ops & Monitoring

- Avoid hardcoding absolute paths; build paths using `path.join(__dirname, ...)` (CommonJS) or the ESM equivalent.
- Log resolved paths only during startup or debugging, as they may expose filesystem structure.
- Include file path information in structured logs for debugging, but avoid leaking sensitive deployment paths in production.
- OpenTelemetry tracing is generally unrelated to filename resolution, but startup diagnostics can record module metadata if useful.

---

# Deployment & Scaling

- `__filename` is stable regardless of the process's current working directory (`process.cwd()`).
- Containerized applications should use relative paths from `__dirname`/resolved module paths instead of assuming fixed filesystem layouts.
- For serverless environments, bundle tools may rewrite paths; validate path-dependent code after bundling.
- Prefer modern Node.js versions (Node.js 18+ LTS, 20+, or newer) for consistent ESM behavior.

---

# Pitfalls

- **Do not confuse `__filename` with `process.cwd()`**—`__filename` is the current module's path, while `process.cwd()` is where the process was started.
- **`__filename` is unavailable in ES Modules**; use `import.meta.url` with `fileURLToPath()`.
- Use the `path` module instead of string concatenation when constructing filesystem paths.

## Question 2. How do you create a simple event listener using EventEmitter?

## Question 3. How do you remove an event listener in Node.js?

## Question 4. How do you check if an event has listeners attached?

## Question 5. How do you handle synchronous errors inside event handlers?

## Question 6. How do you create a simple TCP client?

## Question 7. How do you create a simple TCP server?

## Question 8. How do you implement echo functionality using a TCP server?

## Question 9. How do you implement basic UDP message sending and receiving?

## Question 10. How do you use Node.js REPL to evaluate code dynamically?

## Question 11. How do you implement a file watcher for multiple directories using `chokidar`?

## Question 12. How do you implement JSON schema validation in Node.js?

## Question 13. How do you implement centralized error handling for API routes?

## Question 14. How do you implement request logging with unique request IDs?

## Question 15. How do you implement rate limiting per API key?

## Question 16. How do you implement in-memory caching using `lru-cache`?

## Question 17. How do you implement response compression selectively in Express.js?

## Question 18. How do you implement streaming large responses from MongoDB?

## Question 19. How do you implement retry logic for failed HTTP requests using `axios-retry`?

## Question 20. How do you implement JWT token refresh without invalidating old tokens?
