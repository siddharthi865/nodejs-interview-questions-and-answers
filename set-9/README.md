# Set 9

| S.No. | Question                                                                                                                                                          |
| ----- | ----------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you handle environment-specific configurations in Node.js?](#question-1-how-do-you-handle-environment-specific-configurations-in-nodejs)                  |
| 2.    | [How do you implement logging using `winston` or `pino`?](#question-2-how-do-you-implement-logging-using-winston-or-pino)                                         |
| 3.    | [How do you test Node.js APIs using Jest or Mocha?](#question-3-how-do-you-test-nodejs-apis-using-jest-or-mocha)                                                  |
| 4.    | [How do you mock external services in Node.js tests?](#question-4-how-do-you-mock-external-services-in-nodejs-tests)                                              |
| 5.    | [How do you handle database migrations in Node.js?](#question-5-how-do-you-handle-database-migrations-in-nodejs)                                                  |
| 6.    | [How do you implement WebSocket communication using `ws` or `Socket.IO`?](#question-6-how-do-you-implement-websocket-communication-using-ws-or-socketio)          |
| 7.    | [How do you implement real-time chat using Node.js?](#question-7-how-do-you-implement-real-time-chat-using-nodejs)                                                |
| 8.    | [How do you handle uncaught promise rejections in Node.js?](#question-8-how-do-you-handle-uncaught-promise-rejections-in-nodejs)                                  |
| 9.    | [How do you implement request throttling in Node.js?](#question-9-how-do-you-implement-request-throttling-in-nodejs-implied-from-content)                         |
| 10.   | [How do you implement file streaming with progress monitoring in Node.js?](#question-10-how-do-you-implement-file-streaming-with-progress-monitoring-in-nodejs)   |
| 11.   | [How does the Node.js event loop work internally?](#question-11-how-does-the-nodejs-event-loop-work-internally)                                                   |
| 12.   | [Explain the phases of the Node.js event loop](#question-12-explain-the-phases-of-the-nodejs-event-loop)                                                          |
| 13.   | [What is the difference between blocking and non-blocking code in Node.js?](#question-13-what-is-the-difference-between-blocking-and-non-blocking-code-in-nodejs) |
| 14.   | [How does Node.js handle thread management?](#question-14-how-does-nodejs-handle-thread-management)                                                               |
| 15.   | [What is libuv and how does it relate to the Node.js event loop?](#question-15-what-is-libuv-and-how-does-it-relate-to-the-nodejs-event-loop)                     |
| 16.   | [How do worker threads differ from child processes in Node.js?](#question-16-how-do-worker-threads-differ-from-child-processes-in-nodejs)                         |
| 17.   | [How do you perform CPU-intensive tasks without blocking the event loop?](#question-17-how-do-you-perform-cpu-intensive-tasks-without-blocking-the-event-loop)    |
| 18.   | [How do you implement horizontal scaling of Node.js applications?](#question-18-how-do-you-implement-horizontal-scaling-of-nodejs-applications)                   |
| 19.   | [What is sticky sessions and why is it needed in Node.js clustering?](#question-19-what-is-sticky-sessions-and-why-is-it-needed-in-nodejs-clustering)             |
| 20.   | [How do you manage memory leaks in long-running Node.js applications?](#question-20-how-do-you-manage-memory-leaks-in-long-running-nodejs-applications)           |

## Question 1. How do you handle environment-specific configurations in Node.js?

# Short answer

Environment-specific configuration in Node.js is typically handled by separating **configuration from code**. Use environment variables (`process.env`) as the source of truth, load them using tools like `dotenv` for local development, validate them at startup, and expose a centralized configuration module that the rest of the application consumes. Avoid hardcoding secrets or environment-specific values.

---

# Explanation

A senior Node.js application should follow the **12-Factor App** principle of storing configuration in the environment instead of embedding it in code.

## Common approach

```
project/
├── src/
│   ├── config/
│   │   └── index.ts
│   ├── app.ts
│   └── server.ts
├── .env
├── .env.development
├── .env.production
├── .env.test
└── package.json
```

Example environment files:

```text
# .env.development
PORT=3000
DB_URL=postgres://localhost/dev
LOG_LEVEL=debug
```

```text
# .env.production
PORT=8080
DB_URL=postgres://prod-db
LOG_LEVEL=info
```

---

## Loading configuration

During local development:

```bash
NODE_ENV=development
```

Load variables:

```js
import "dotenv/config";
```

or

```js
import dotenv from "dotenv";
dotenv.config();
```

Once loaded:

```js
const port = process.env.PORT;
```

---

## Centralize configuration

Instead of using `process.env` everywhere:

```ts
export const config = {
  port: Number(process.env.PORT) || 3000,
  dbUrl: process.env.DB_URL!,
  logLevel: process.env.LOG_LEVEL ?? "info",
};
```

Application code:

```ts
import { config } from "./config";

app.listen(config.port);
```

Benefits:

- Single source of truth
- Easier testing
- Easier validation
- Cleaner dependency injection
- Better maintainability

---

## Validate configuration

A production application should fail fast if required configuration is missing.

Example using Zod:

```ts
const schema = z.object({
  PORT: z.coerce.number(),
  DB_URL: z.string().url(),
});
```

If validation fails:

- application exits immediately
- prevents runtime failures later
- avoids partially initialized services

---

## Runtime behavior

Configuration loading happens **once during process startup**.

```
Node starts
      │
Load .env
      │
Populate process.env
      │
Validate
      │
Create config object
      │
Start HTTP server
```

Reading `process.env` repeatedly is inexpensive, but accessing a validated configuration object avoids scattered parsing logic and keeps code consistent.

---

## Different environments

Typical environments include:

| Environment | Purpose                    |
| ----------- | -------------------------- |
| development | Local development          |
| test        | Unit/integration tests     |
| staging     | Production-like validation |
| production  | Live deployment            |

Example:

```ts
switch (process.env.NODE_ENV) {
  case "production":
    enableCompression();
    break;

  case "development":
    enableVerboseLogging();
    break;
}
```

Avoid putting business logic behind `NODE_ENV`; reserve it for environment-specific behavior such as logging, debugging, or performance tuning.

---

## Secrets management

Never commit secrets to Git.

Instead use:

- Cloud Secret Managers
- Kubernetes Secrets
- Docker Secrets
- CI/CD environment variables
- AWS Secrets Manager
- Azure Key Vault
- Google Secret Manager
- HashiCorp Vault

Local development may use:

```
.env
```

Production should inject secrets securely through the deployment platform.

---

## Performance considerations

Configuration is typically loaded only once.

Good:

```
Startup
 ↓
Read env
 ↓
Validate
 ↓
Freeze config
```

Avoid:

```ts
function connect() {
  const url = process.env.DB_URL;
}
```

Prefer:

```ts
const config = loadConfig();

function connect() {
  const url = config.dbUrl;
}
```

While repeatedly reading `process.env` is rarely a bottleneck, centralizing configuration improves consistency, type safety, and testability.

---

# Example (TypeScript)

**Requires:** Node.js 20+ and `zod`.

```ts
import "dotenv/config";
import { z } from "zod";

const EnvSchema = z.object({
  NODE_ENV: z
    .enum(["development", "test", "production"])
    .default("development"),
  PORT: z.coerce.number().default(3000),
  DB_URL: z.string().min(1),
});

const env = EnvSchema.parse(process.env);

export const config = Object.freeze({
  env: env.NODE_ENV,
  port: env.PORT,
  dbUrl: env.DB_URL,
});

// Example usage
console.log(`Starting server on port ${config.port}`);
```

This example validates configuration once, provides typed access, and freezes the resulting object to prevent accidental mutation.

---

# Testing

Configuration should be testable in isolation.

**Unit tests**

- Test valid configurations.
- Test missing required variables.
- Test invalid values (e.g., non-numeric ports).
- Restore `process.env` after each test to avoid cross-test contamination.

Using the built-in test runner:

```js
import test from "node:test";
import assert from "node:assert/strict";

test("PORT should be numeric", () => {
  process.env.PORT = "3000";
  assert.equal(Number(process.env.PORT), 3000);
});
```

Run:

```bash
node --test
```

Integration tests should use a dedicated `.env.test` (or equivalent injected environment variables) so they do not depend on development or production settings.

---

# Ops & Monitoring

- Validate configuration during startup and exit with a non-zero status if invalid.
- Never log secrets; mask sensitive values in logs.
- Emit application version, environment, and configuration metadata (excluding secrets) at startup.
- Use structured logging (e.g., Pino) and instrument services with OpenTelemetry for metrics and tracing.
- In containers, inject configuration via environment variables or orchestrator-managed secrets rather than baking them into the image.

---

# Deployment & Scaling

- Build one immutable container image and inject environment-specific values at deployment time.
- Use connection pooling settings (database, Redis, etc.) from configuration to tune per environment.
- In serverless deployments, load and validate configuration during cold start to fail fast on misconfiguration.
- Keep production, staging, and test environments as similar as possible to reduce deployment surprises.
- Target an active LTS version of Node.js (Node.js 20+ or newer) and keep runtime versions consistent across environments.

---

# Pitfalls

- **Don't commit `.env` files containing secrets** to version control.
- **Don't access `process.env` throughout the codebase**; use a centralized, validated configuration module.
- **Don't skip validation**—missing or malformed configuration should stop the application from starting.

## Question 2. How do you implement logging using `winston` or `pino`?

## Question 3. How do you test Node.js APIs using Jest or Mocha?

## Question 4. How do you mock external services in Node.js tests?

## Question 5. How do you handle database migrations in Node.js?

## Question 6. How do you implement WebSocket communication using `ws` or `Socket.IO`?

## Question 7. How do you implement real-time chat using Node.js?

## Question 8. How do you handle uncaught promise rejections in Node.js?

## Question 9. How do you implement request throttling in Node.js? (implied from content)

## Question 10. How do you implement file streaming with progress monitoring in Node.js?

## Question 11. How does the Node.js event loop work internally?

## Question 12. Explain the phases of the Node.js event loop

## Question 13. What is the difference between blocking and non-blocking code in Node.js?

## Question 14. How does Node.js handle thread management?

## Question 15. What is libuv and how does it relate to the Node.js event loop?

## Question 16. How do worker threads differ from child processes in Node.js?

## Question 17. How do you perform CPU-intensive tasks without blocking the event loop?

## Question 18. How do you implement horizontal scaling of Node.js applications?

## Question 19. What is sticky sessions and why is it needed in Node.js clustering?

## Question 20. How do you manage memory leaks in long-running Node.js applications?
