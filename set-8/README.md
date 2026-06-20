# Set 8

| S.No. | Question                                                                                                                                                              |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between `exports` and `exports.default`?](#question-1-what-is-the-difference-between-exports-and-exportsdefault)                              |
| 2.    | [How do you implement RESTful routes in Express.js?](#question-2-how-do-you-implement-restful-routes-in-expressjs)                                                    |
| 3.    | [How do you implement middleware for logging requests in Express?](#question-3-how-do-you-implement-middleware-for-logging-requests-in-express)                       |
| 4.    | [Explain error-handling middleware in Express.js](#question-4-explain-error-handling-middleware-in-expressjs)                                                         |
| 5.    | [How do you serve static files in Node.js?](#question-5-how-do-you-serve-static-files-in-nodejs)                                                                      |
| 6.    | [What is the difference between `res.redirect()` and `res.sendFile()` in Express?](#question-6-what-is-the-difference-between-resredirect-and-ressendfile-in-express) |
| 7.    | [How do you parse query parameters in Express.js?](#question-7-how-do-you-parse-query-parameters-in-expressjs)                                                        |
| 8.    | [How do you handle URL-encoded and JSON request bodies?](#question-8-how-do-you-handle-url-encoded-and-json-request-bodies)                                           |
| 9.    | [How do you implement session management in Node.js?](#question-9-how-do-you-implement-session-management-in-nodejs)                                                  |
| 10.   | [What is the difference between cookies and sessions?](#question-10-what-is-the-difference-between-cookies-and-sessions)                                              |
| 11.   | [How do you store sessions in Redis with Node.js?](#question-11-how-do-you-store-sessions-in-redis-with-nodejs)                                                       |
| 12.   | [How do you implement Role-Based Access Control (RBAC) in Node.js?](#question-12-how-do-you-implement-role-based-access-control-rbac-in-nodejs)                       |
| 13.   | [How do you handle file uploads using `multer`?](#question-13-how-do-you-handle-file-uploads-using-multer)                                                            |
| 14.   | [How do you implement rate limiting in Node.js?](#question-14-how-do-you-implement-rate-limiting-in-nodejs)                                                           |
| 15.   | [How do you implement caching using `node-cache`?](#question-15-how-do-you-implement-caching-using-node-cache)                                                        |
| 16.   | [How do you handle API versioning in Node.js?](#question-16-how-do-you-handle-api-versioning-in-nodejs)                                                               |
| 17.   | [How do you implement request validation in Node.js?](#question-17-how-do-you-implement-request-validation-in-nodejs)                                                 |
| 18.   | [How do you implement pagination in Node.js APIs?](#question-18-how-do-you-implement-pagination-in-nodejs-apis)                                                       |
| 19.   | [How do you send emails using Node.js?](#question-19-how-do-you-send-emails-using-nodejs)                                                                             |
| 20.   | [How do you generate PDFs in Node.js?](#question-20-how-do-you-generate-pdfs-in-nodejs)                                                                               |

## Question 1. What is the difference between `exports` and `exports.default`?

# Short answer

- **`exports`** (or `module.exports`) is the object that a **CommonJS (CJS)** module returns when another file calls `require()`.
- **`exports.default`** is simply a property named `default` on that exported object. It has **no special meaning in CommonJS itself**. It is primarily used by transpilers (TypeScript, Babel) to emulate **ES Module (ESM) default exports**.

In Node.js interviews, the key point is that **`exports.default` is not equivalent to `module.exports`**.

---

# Explanation

## CommonJS exports

Every CommonJS module starts with:

```js
module.exports = {};
exports = module.exports;
```

`exports` is just a reference (alias) to `module.exports`.

These are equivalent:

```js
exports.sayHello = () => {};
```

```js
module.exports.sayHello = () => {};
```

When another module imports it:

```js
const obj = require("./module");
```

`obj` becomes:

```js
{
  sayHello: [Function];
}
```

---

## What is `exports.default`?

This simply creates a property called `default`.

```js
exports.default = function greet() {
  console.log("Hello");
};
```

The exported object becomes:

```js
{
  default: [Function: greet]
}
```

So consuming it requires:

```js
const mod = require("./module");

mod.default();
```

or

```js
const { default: greet } = require("./module");
greet();
```

Node.js itself does **not** automatically unwrap the `default` property.

---

## Why does `exports.default` exist?

It exists because **ES Modules** support default exports.

ESM:

```js
export default function greet() {}
```

TypeScript/Babel often transpile this into CommonJS similar to:

```js
Object.defineProperty(exports, "__esModule", {
  value: true,
});

exports.default = greet;
```

Now the CommonJS output mimics an ES Module.

This enables tools to do:

```js
import greet from "./module";
```

even when the runtime is CommonJS.

---

## Runtime behavior

Node.js loads CommonJS modules by executing them once and caching `module.exports`.

For example:

```js
exports.default = "hello";
```

results in

```js
module.exports = {
  default: "hello",
};
```

whereas

```js
module.exports = "hello";
```

results in

```js
"hello";
```

These produce completely different runtime values.

---

## Architecture considerations

### Prefer `module.exports`

For pure CommonJS packages:

```js
module.exports = myFunction;
```

Consumers get:

```js
const fn = require("./module");
```

Simple and predictable.

---

### Prefer ESM for new projects

Modern Node.js supports native ES Modules.

```js
export default myFunction;
```

Consumers:

```js
import myFunction from "./module.js";
```

Avoid manually writing:

```js
exports.default = ...
```

unless you're generating CommonJS via a transpiler.

---

## Performance implications

There is essentially **no measurable runtime performance difference**.

The differences are about:

- interoperability
- maintainability
- developer experience
- compatibility between CJS and ESM

---

## Example (JavaScript)

```js
// math.js (CommonJS)

exports.add = (a, b) => a + b;

exports.default = (a, b) => a * b;
```

```js
// app.js

const math = require("./math");

console.log(math.add(2, 3)); // 5
console.log(math.default(2, 3)); // 6
```

If instead:

```js
module.exports = (a, b) => a * b;
```

then:

```js
const multiply = require("./math");

console.log(multiply(2, 3));
```

works directly.

---

# Testing

### Unit testing

Verify both named and default-style exports.

Using Node's built-in test runner:

```js
import test from "node:test";
import assert from "node:assert/strict";

const math = require("./math");

test("default export", () => {
  assert.equal(math.default(2, 3), 6);
});
```

Run:

```bash
node --test
```

Integration tests should verify interoperability between CommonJS (`require`) and ES Modules (`import`) if your package supports both.

---

# Ops & Monitoring

- Log module initialization only when useful; avoid side effects during module load.
- Instrument application behavior (rather than module exports) with OpenTelemetry where appropriate.
- Surface module-loading errors early and fail fast on startup.
- Use process managers such as PM2, `systemd`, or container orchestrators to supervise the Node.js process.

---

# Deployment & Scaling

- Prefer native ESM (`"type": "module"`) for new Node.js services when your ecosystem supports it.
- If publishing libraries, consider dual ESM/CJS builds for compatibility.
- Avoid mixing `exports.default` with manual CommonJS exports unless required by your build tooling.
- Connection pooling, horizontal scaling, and containerization are unaffected by the export style.
- Target modern LTS versions of Node.js (Node.js 20+ or newer) for the best ESM interoperability.

---

# Pitfalls

- **Don't reassign `exports`.** `exports = ...` breaks the link with `module.exports`; use `module.exports = ...` instead.
- **Don't assume `exports.default` behaves like ESM `export default`.** In CommonJS it's just a regular object property.
- **Avoid mixing CommonJS and ESM without understanding interop.** Consumers may need `.default` depending on how the module was authored or transpiled.

## Question 2. How do you implement RESTful routes in Express.js?

## Question 3. How do you implement middleware for logging requests in Express?

## Question 4. Explain error-handling middleware in Express.js

## Question 5. How do you serve static files in Node.js?

## Question 6. What is the difference between `res.redirect()` and `res.sendFile()` in Express?

## Question 7. How do you parse query parameters in Express.js?

## Question 8. How do you handle URL-encoded and JSON request bodies?

## Question 9. How do you implement session management in Node.js?

## Question 10. What is the difference between cookies and sessions?

## Question 11. How do you store sessions in Redis with Node.js?

## Question 12. How do you implement Role-Based Access Control (RBAC) in Node.js?

## Question 13. How do you handle file uploads using `multer`?

## Question 14. How do you implement rate limiting in Node.js?

## Question 15. How do you implement caching using `node-cache`?

## Question 16. How do you handle API versioning in Node.js?

## Question 17. How do you implement request validation in Node.js?

## Question 18. How do you implement pagination in Node.js APIs?

## Question 19. How do you send emails using Node.js?

## Question 20. How do you generate PDFs in Node.js?
