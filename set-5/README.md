# Set 5

| S.No. | Question                                                                                                                                                                                                               |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between `require.cache` and clearing cache manually?](#question-1-what-is-the-difference-between-requirecache-and-clearing-cache-manually)                                                     |
| 2.    | [How do you prevent callback hell in Node.js?](#question-2-how-do-you-prevent-callback-hell-in-nodejs)                                                                                                                 |
| 3.    | [How do Node.js streams differ from traditional file I/O in terms of memory usage?](#question-3-how-do-nodejs-streams-differ-from-traditional-file-io-in-terms-of-memory-usage)                                        |
| 4.    | [How do you implement microservices using Node.js?](#question-4-how-do-you-implement-microservices-using-nodejs)                                                                                                       |
| 5.    | [How does Node.js handle HTTPS requests internally?](#question-5-how-does-nodejs-handle-https-requests-internally)                                                                                                     |
| 6.    | [How do you implement WebSockets in Node.js?](#question-6-how-do-you-implement-websockets-in-nodejs)                                                                                                                   |
| 7.    | [Explain Server-Sent Events (SSE) and how it works with Node.js](#question-7-explain-server-sent-events-sse-and-how-it-works-with-nodejs)                                                                              |
| 8.    | [What is the difference between Node.js `cluster` and load balancers?](#question-8-what-is-the-difference-between-nodejs-cluster-and-load-balancers)                                                                   |
| 9.    | [How do you implement authentication using OAuth2 in Node.js?](#question-9-how-do-you-implement-authentication-using-oauth2-in-nodejs)                                                                                 |
| 10.   | [Explain how to handle file streaming and large file uploads efficiently in Node.js](#question-10-explain-how-to-handle-file-streaming-and-large-file-uploads-efficiently-in-nodejs)                                   |
| 11.   | [How do you manage Node.js process memory usage?](#question-11-how-do-you-manage-nodejs-process-memory-usage)                                                                                                          |
| 12.   | [Explain the difference between Node.js single-threaded architecture and multi-threaded architecture](#question-12-explain-the-difference-between-nodejs-single-threaded-architecture-and-multi-threaded-architecture) |
| 13.   | [What are the best practices to improve Node.js application performance?](#question-13-what-are-the-best-practices-to-improve-nodejs-application-performance)                                                          |
| 14.   | [How do you implement real-time notifications in Node.js?](#question-14-how-do-you-implement-real-time-notifications-in-nodejs)                                                                                        |
| 15.   | [How do you handle multiple concurrent database connections in Node.js?](#question-15-how-do-you-handle-multiple-concurrent-database-connections-in-nodejs)                                                            |
| 16.   | [Explain the differences between Node.js callback, promise, and `async/await` performance-wise](#question-16-explain-the-differences-between-nodejs-callback-promise-and-asyncawait-performance-wise)                  |
| 17.   | [How do you implement distributed caching in Node.js?](#question-17-how-do-you-implement-distributed-caching-in-nodejs)                                                                                                |
| 18.   | [How do you handle clustered Node.js applications with shared sessions?](#question-18-how-do-you-handle-clustered-nodejs-applications-with-shared-sessions)                                                            |
| 19.   | [How do you use Node.js with TypeScript for large-scale applications?](#question-19-how-do-you-use-nodejs-with-typescript-for-large-scale-applications)                                                                |
| 20.   | [How do you monitor Node.js applications in production?](#question-20-how-do-you-monitor-nodejs-applications-in-production)                                                                                            |

## Question 1. What is the difference between `require.cache` and clearing cache manually?

# Short answer

`require.cache` is the built-in Node.js object that stores all loaded CommonJS modules. Clearing cache manually means deleting entries from `require.cache` (or using custom invalidation logic) so that the next `require()` reloads the module from disk.

In short:

- `require.cache` = the cache storage maintained by Node.js.
- Manual cache clearing = the act of removing entries from that storage.

---

# Explanation

When a CommonJS module is loaded via `require()`:

1. Node resolves the module path.
2. Reads and executes the file.
3. Creates a `Module` instance.
4. Stores the module in `require.cache`.
5. Future `require()` calls return the cached `exports` object instead of re-executing the file.

```txt
require('./config')
        ↓
execute module once
        ↓
store in require.cache
        ↓
subsequent require() returns cached exports
```

### Inspecting `require.cache`

```js
console.log(Object.keys(require.cache));
```

Each key is the absolute path of a loaded module.

Example:

```js
console.log(require.cache[require.resolve("./config")]);
```

---

## Manual cache clearing

To force Node.js to reload a module:

```js
delete require.cache[require.resolve("./config")];
```

Next call:

```js
const config = require("./config");
```

will:

- Re-read the file
- Re-execute module code
- Create a new exports object

---

## Runtime behavior

### Without cache clearing

```txt
Request 1
 └─ require(config) → executes

Request 2
 └─ require(config) → cached

Request 3
 └─ require(config) → cached
```

Only one execution occurs.

### With cache clearing

```txt
Request 1
 └─ require(config)

delete cache

Request 2
 └─ require(config) → executes again
```

The module initialization cost happens repeatedly.

---

## Performance implications

### Cached modules

Advantages:

- Faster startup after first load
- Lower disk I/O
- Shared singleton instances
- Reduced memory churn

Typical examples:

- Database pools
- Logger instances
- Configuration loaders

### Frequently clearing cache

Disadvantages:

- More filesystem access
- Re-execution overhead
- Potential memory leaks
- Loss of singleton behavior

Example:

```js
// db.js
module.exports = createDatabasePool();
```

If cache is cleared repeatedly:

```js
delete require.cache[require.resolve("./db")];
require("./db");
```

you may accidentally create multiple database pools.

---

## Common use cases for manual cache clearing

### Hot reloading

Development servers:

- Nodemon
- Custom plugin systems
- Dynamic configuration reloads

### Testing

Each test gets a fresh module instance:

```js
beforeEach(() => {
  delete require.cache[require.resolve("../config")];
});
```

### Plugin architectures

Reload plugins without restarting the process.

---

## CommonJS vs ESM

`require.cache` only applies to **CommonJS**.

```js
require("./file"); // cached in require.cache
```

For ES Modules:

```js
import "./file.js";
```

there is no public equivalent of `require.cache`.

Node maintains an internal ESM module graph, but it is not exposed for direct manipulation.

---

# Example

**JavaScript (CommonJS)**

```js
// counter.js
let count = 0;

count++;

module.exports = {
  count,
};
```

```js
// app.js
const path = require.resolve("./counter");

console.log(require("./counter").count); // 1
console.log(require("./counter").count); // 1

delete require.cache[path];

console.log(require("./counter").count); // 1 (module re-executed)
```

The third load re-executes the module because the cache entry was removed.

---

# Testing

### Unit testing

Verify that module state persists while cached:

```js
const mod1 = require("../counter");
const mod2 = require("../counter");

expect(mod1).toBe(mod2);
```

Verify reload behavior:

```js
delete require.cache[require.resolve("../counter")];

const fresh = require("../counter");
```

### Built-in test runner

```bash
node --test
```

Example:

```js
import test from "node:test";
import assert from "node:assert";
```

---

# Ops & Monitoring

- Avoid clearing cache in production request paths.
- Log module reload events when implementing hot reload systems.
- Track memory usage (`process.memoryUsage()`).
- Use OpenTelemetry spans around dynamic module loading.
- Handle reload failures gracefully and keep the previous module version active when possible.

---

# Deployment & Scaling

- Cache invalidation is process-local; in clustered environments each worker has its own cache.
- In containers, prefer rolling deployments over runtime cache manipulation.
- For serverless functions, module caching improves warm-start performance.
- If using `cluster` or multiple containers, cache clearing in one process does not affect others.
- Use Node.js 20+ or 22+ LTS for the latest module loader improvements.

---

# Pitfalls

- Deleting a cache entry does **not** automatically clear its child dependencies.
- Requiring a module again may create duplicate database connections or singleton instances.
- `require.cache` works only for CommonJS, not ES Modules.

## Question 2. How do you prevent callback hell in Node.js?

## Question 3. How do Node.js streams differ from traditional file I/O in terms of memory usage?

## Question 4. How do you implement microservices using Node.js?

## Question 5. How does Node.js handle HTTPS requests internally?

## Question 6. How do you implement WebSockets in Node.js?

## Question 7. Explain Server-Sent Events (SSE) and how it works with Node.js

## Question 8. What is the difference between Node.js `cluster` and load balancers?

## Question 9. How do you implement authentication using OAuth2 in Node.js?

## Question 10. Explain how to handle file streaming and large file uploads efficiently in Node.js

## Question 11. How do you manage Node.js process memory usage?

## Question 12. Explain the difference between Node.js single-threaded architecture and multi-threaded architecture

## Question 13. What are the best practices to improve Node.js application performance?

## Question 14. How do you implement real-time notifications in Node.js?

## Question 15. How do you handle multiple concurrent database connections in Node.js?

## Question 16. Explain the differences between Node.js callback, promise, and `async/await` performance-wise

## Question 17. How do you implement distributed caching in Node.js?

## Question 18. How do you handle clustered Node.js applications with shared sessions?

## Question 19. How do you use Node.js with TypeScript for large-scale applications?

## Question 20. How do you monitor Node.js applications in production?
