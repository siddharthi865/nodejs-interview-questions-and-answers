# Set 11

| S.No. | Question                                                                                                                                                           |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [What is the difference between Node.js and Deno?](#question-1-what-is-the-difference-between-nodejs-and-deno)                                                     |
| 2.    | [Can Node.js be used for scripting tasks outside web servers? Give examples](#question-2-can-nodejs-be-used-for-scripting-tasks-outside-web-servers-give-examples) |
| 3.    | [How do you check if a Node.js module exists before requiring it?](#question-3-how-do-you-check-if-a-nodejs-module-exists-before-requiring-it)                     |
| 4.    | [What is the cluster module in Node.js and how does it work?](#question-4-what-is-the-cluster-module-in-nodejs-and-how-does-it-work)                               |
| 5.    | [How do you create a simple UDP server using Node.js?](#question-5-how-do-you-create-a-simple-udp-server-using-nodejs)                                             |
| 6.    | [What is the difference between TCP and UDP servers in Node.js?](#question-6-what-is-the-difference-between-tcp-and-udp-servers-in-nodejs)                         |
| 7.    | [How do you list all globally installed npm packages?](#question-7-how-do-you-list-all-globally-installed-npm-packages)                                            |
| 8.    | [What is the difference between `npm` and `npx`?](#question-8-what-is-the-difference-between-npm-and-npx)                                                          |
| 9.    | [How do you update a Node.js package to the latest version?](#question-9-how-do-you-update-a-nodejs-package-to-the-latest-version)                                 |
| 10.   | [What is the difference between `npm` and `yarn`?](#question-10-what-is-the-difference-between-npm-and-yarn)                                                       |
| 11.   | [How do you handle deprecation warnings in Node.js?](#question-11-how-do-you-handle-deprecation-warnings-in-nodejs)                                                |
| 12.   | [What is the purpose of `.npmignore` file?](#question-12-what-is-the-purpose-of-npmignore-file)                                                                    |
| 13.   | [How do you create a simple CLI tool using Node.js?](#question-13-how-do-you-create-a-simple-cli-tool-using-nodejs)                                                |
| 14.   | [How do you read environment variables using `dotenv`?](#question-14-how-do-you-read-environment-variables-using-dotenv)                                           |
| 15.   | [How do you set default values for environment variables in Node.js?](#question-15-how-do-you-set-default-values-for-environment-variables-in-nodejs)              |
| 16.   | [What is the difference between `process.env` and a configuration file?](#question-16-what-is-the-difference-between-processenv-and-a-configuration-file)          |
| 17.   | [How do you write a JSON file asynchronously in Node.js?](#question-17-how-do-you-write-a-json-file-asynchronously-in-nodejs)                                      |
| 18.   | [How do you rename a file in Node.js?](#question-18-how-do-you-rename-a-file-in-nodejs)                                                                            |
| 19.   | [How do you delete a file in Node.js?](#question-19-how-do-you-delete-a-file-in-nodejs)                                                                            |
| 20.   | [What is the difference between `fs.exists` and `fs.access`?](#question-20-what-is-the-difference-between-fsexists-and-fsaccess)                                   |

## Question 1. What is the difference between Node.js and Deno?

# Short answer

**Node.js** is the mature, widely adopted JavaScript runtime with the largest ecosystem (npm), extensive backward compatibility, and broad production usage.

**Deno** is a newer runtime created by Ryan Dahl (the creator of Node.js) that focuses on **security by default**, **built-in TypeScript support**, **ES Modules**, and an integrated developer experience with built-in tooling such as formatting, linting, testing, and dependency management.

For most enterprise applications today, **Node.js remains the default choice**, while **Deno is attractive for greenfield projects that value modern defaults and security.**

---

# Explanation

## 1. Origin and Design Goals

| Node.js              | Deno                                |
| -------------------- | ----------------------------------- |
| Released in 2009     | Released in 2020                    |
| Created by Ryan Dahl | Also created by Ryan Dahl           |
| Built on V8 + libuv  | Built on V8 + Rust (Tokio)          |
| Huge ecosystem       | Modern runtime with secure defaults |

Ryan Dahl designed Deno after identifying several architectural issues in Node.js, including:

- Lack of security permissions
- Dependency management problems
- CommonJS design limitations
- External tooling fragmentation

---

## 2. Module System

### Node.js

Historically:

```js
const fs = require("node:fs");
```

Modern Node.js also supports ES Modules:

```js
import fs from "node:fs";
```

Node supports **both CommonJS and ESM**, which introduces compatibility complexity.

---

### Deno

Only supports ES Modules.

```ts
import { serve } from "jsr:@std/http";
```

No `require()`.

This makes module resolution simpler.

---

## 3. Package Management

### Node.js

Uses

- npm
- pnpm
- Yarn

Dependencies stored in

```
node_modules/
```

Example:

```
npm install express
```

---

### Deno

Originally imported packages directly from URLs.

Example:

```ts
import { serve } from "https://deno.land/std/http/server.ts";
```

Today, Deno primarily supports the **JSR** package registry and also has good npm compatibility.

```bash
deno add jsr:@std/http
```

No massive `node_modules` folder is required for native Deno modules.

---

## 4. Security Model

This is one of Deno's biggest advantages.

### Node.js

Has unrestricted access.

```js
import fs from "node:fs";

fs.readFileSync("/etc/passwd");
```

No permission required.

---

### Deno

Everything is sandboxed.

Trying:

```ts
await Deno.readTextFile("secret.txt");
```

fails unless started with:

```bash
deno run --allow-read app.ts
```

Permissions include:

```
--allow-read
--allow-write
--allow-net
--allow-env
--allow-run
```

This greatly reduces accidental security risks.

---

## 5. TypeScript Support

### Node.js

Needs:

- TypeScript compiler (`tsc`)
- or runtime like tsx/ts-node

Compilation step required.

---

### Deno

Native TypeScript support.

```bash
deno run app.ts
```

No extra compiler setup.

---

## 6. Standard Library

### Node.js

Relies heavily on npm packages.

Need a UUID?

```
npm install uuid
```

Need dotenv?

```
npm install dotenv
```

---

### Deno

Ships with a well-maintained standard library and built-in APIs for many common tasks, reducing reliance on third-party packages.

---

## 7. Built-in Tooling

### Node.js

Common tools:

- ESLint
- Prettier
- Jest
- ts-node
- nodemon

Installed separately.

---

### Deno

Built-in commands:

```bash
deno fmt
deno lint
deno test
deno bench
deno compile
```

No external setup needed.

---

## 8. Performance

Both use Google's V8 engine.

### Node.js

- Highly optimized
- Mature ecosystem
- Excellent HTTP performance
- Very stable under production workloads

---

### Deno

Also very fast.

Advantages include:

- Faster startup for some workloads
- Efficient async runtime
- Rust implementation
- Modern APIs

In real-world backend applications, performance differences are often small compared with architecture, I/O patterns, and application design.

---

## 9. Ecosystem

### Node.js

- 2M+ npm packages
- Huge community
- Excellent enterprise support
- Extensive cloud tooling

---

### Deno

Smaller ecosystem.

However:

- Supports npm packages
- Supports Node compatibility APIs

Compatibility has improved significantly.

---

## 10. Runtime Architecture

### Node.js

```
JavaScript

↓

V8 Engine

↓

libuv Event Loop

↓

OS
```

Uses **libuv** for:

- Event loop
- Thread pool
- File I/O
- Networking
- Timers

---

### Deno

```
JavaScript

↓

V8

↓

Rust Runtime

↓

Tokio Async Runtime

↓

OS
```

Uses **Rust** and **Tokio** instead of libuv as its primary async runtime.

---

## 11. When to Choose Which?

### Choose Node.js

- Existing enterprise systems
- Large backend services
- Microservices
- Express/Fastify/NestJS
- Large engineering teams
- Maximum ecosystem compatibility

### Choose Deno

- Greenfield projects
- Security-sensitive services
- Small APIs
- Edge/serverless workloads
- Teams wanting batteries-included tooling

---

# Example (TypeScript)

**Node.js (ESM)**

```ts
import { readFile } from "node:fs/promises";

async function main() {
  const text = await readFile("message.txt", "utf8");
  console.log(text);
}

main().catch(console.error);
```

**Deno**

```ts
const text = await Deno.readTextFile("message.txt");
console.log(text);

// Run with:
// deno run --allow-read app.ts
```

The Node.js example uses the `node:fs/promises` API, while the Deno example uses the built-in `Deno` namespace and requires explicit read permission.

---

# Testing

- **Node.js:** Use the built-in test runner (`node:test`), Jest, or Mocha. Unit test modules in isolation and use integration tests to verify filesystem, network, or database interactions.
- **Deno:** Use the built-in `deno test` command, which supports TypeScript out of the box.

Example (Node.js built-in test runner):

```bash
node --test
```

---

# Ops & Monitoring

- **Logging:** Use structured JSON logging (e.g., Pino in Node.js). In Deno, emit structured logs and integrate with your observability platform.
- **Metrics:** Expose Prometheus-compatible metrics where appropriate.
- **Tracing:** Instrument services with OpenTelemetry for distributed tracing.
- **Error handling:** Handle uncaught exceptions and unhandled promise rejections gracefully; fail fast on unrecoverable errors.
- **Process management:** For Node.js, use containers, `systemd`, or PM2 where appropriate. Deno services are commonly run in containers or under `systemd`.

---

# Deployment & Scaling

- **Containerization:** Use slim runtime images and multi-stage builds to minimize image size.
- **Connection pooling:** Reuse database and HTTP client connections to reduce latency.
- **Horizontal scaling:** Keep services stateless and scale behind a load balancer.
- **Serverless:** Deno's fast startup can be advantageous for cold starts, though platform support should guide your choice.
- **Runtime versions:** Use current LTS releases for Node.js in production. For Deno, pin a tested runtime version in CI/CD to avoid unexpected changes.

---

# Pitfalls

- Mixing CommonJS and ES Modules in Node.js can complicate builds and interoperability.
- In Deno, forgetting required permissions (such as `--allow-net` or `--allow-read`) will cause runtime failures.
- Avoid assuming every npm package works identically across runtimes; verify compatibility and test thoroughly.

## Question 2. Can Node.js be used for scripting tasks outside web servers? Give examples

## Question 3. How do you check if a Node.js module exists before requiring it?

## Question 4. What is the cluster module in Node.js and how does it work?

## Question 5. How do you create a simple UDP server using Node.js?

## Question 6. What is the difference between TCP and UDP servers in Node.js?

## Question 7. How do you list all globally installed npm packages?

## Question 8. What is the difference between `npm` and `npx`?

## Question 9. How do you update a Node.js package to the latest version?

## Question 10. What is the difference between `npm` and `yarn`?

## Question 11. How do you handle deprecation warnings in Node.js?

## Question 12. What is the purpose of `.npmignore` file?

## Question 13. How do you create a simple CLI tool using Node.js?

## Question 14. How do you read environment variables using `dotenv`?

## Question 15. How do you set default values for environment variables in Node.js?

## Question 16. What is the difference between `process.env` and a configuration file?

## Question 17. How do you write a JSON file asynchronously in Node.js?

## Question 18. How do you rename a file in Node.js?

## Question 19. How do you delete a file in Node.js?

## Question 20. What is the difference between `fs.exists` and `fs.access`?
