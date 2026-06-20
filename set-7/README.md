# Set 7

| S.No. | Question                                                                                                                                                              |
| ----- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [What is the difference between `setInterval` and `setTimeout`?](#question-1-what-is-the-difference-between-setinterval-and-settimeout)                               |
| 2.    | [How do you use the `crypto` module in Node.js?](#question-2-how-do-you-use-the-crypto-module-in-nodejs)                                                              |
| 3.    | [How do you generate a random UUID in Node.js?](#question-3-how-do-you-generate-a-random-uuid-in-nodejs)                                                              |
| 4.    | [What are the advantages of using Node.js streams for reading large files?](#question-4-what-are-the-advantages-of-using-nodejs-streams-for-reading-large-files)      |
| 5.    | [What is the difference between `fs.readFile` and `fs.createReadStream`?](#question-5-what-is-the-difference-between-fsreadfile-and-fscreatereadstream)               |
| 6.    | [How do you handle JSON parsing errors in Node.js?](#question-6-how-do-you-handle-json-parsing-errors-in-nodejs)                                                      |
| 7.    | [How do you use environment variables in Node.js?](#question-7-how-do-you-use-environment-variables-in-nodejs)                                                        |
| 8.    | [How do you stop a Node.js server programmatically?](#question-8-how-do-you-stop-a-nodejs-server-programmatically)                                                    |
| 9.    | [Explain the role of `exports` object in Node.js](#question-9-explain-the-role-of-exports-object-in-nodejs)                                                           |
| 10.   | [How do you create a simple TCP server in Node.js?](#question-10-how-do-you-create-a-simple-tcp-server-in-nodejs)                                                     |
| 11.   | [How do you handle command-line input in Node.js?](#question-11-how-do-you-handle-command-line-input-in-nodejs)                                                       |
| 12.   | [What is the difference between Node.js REPL and a script file?](#question-12-what-is-the-difference-between-nodejs-repl-and-a-script-file)                           |
| 13.   | [How do you check memory usage in a Node.js application?](#question-13-how-do-you-check-memory-usage-in-a-nodejs-application)                                         |
| 14.   | [How do you handle uncaught exceptions globally in Node.js?](#question-14-how-do-you-handle-uncaught-exceptions-globally-in-nodejs)                                   |
| 15.   | [What is the purpose of `npm run` scripts?](#question-15-what-is-the-purpose-of-npm-run-scripts)                                                                      |
| 16.   | [Explain the difference between synchronous and asynchronous event handling](#question-16-explain-the-difference-between-synchronous-and-asynchronous-event-handling) |
| 17.   | [What is the difference between `fs.readFile` and `fs.promises.readFile`?](#question-17-what-is-the-difference-between-fsreadfile-and-fspromisesreadfile)             |
| 18.   | [How do you use `util.promisify` in Node.js?](#question-18-how-do-you-use-utilpromisify-in-nodejs)                                                                    |
| 19.   | [What is the difference between `require.resolve()` and `require()`?](#question-19-what-is-the-difference-between-requireresolve-and-require)                         |
| 20.   | [How does Node.js resolve module paths?](#question-20-how-does-nodejs-resolve-module-paths)                                                                           |

## Question 1. What is the difference between `setInterval` and `setTimeout`?

## Short answer

`setTimeout` runs a function once after a delay, while `setInterval` runs a function repeatedly at a fixed interval until it is cleared.

---

## Explanation

From a Node.js runtime perspective, both `setTimeout` and `setInterval` are provided by **libuv’s timer phase** in the event loop.

### `setTimeout(fn, delay)`

- Schedules **one-time execution**
- The callback is placed in the timer queue after at least `delay` ms
- Actual execution may be delayed due to event loop backlog (timers are not precise)

### `setInterval(fn, interval)`

- Schedules **repeated execution**
- After each execution, the timer is rescheduled automatically
- Drift can occur if execution time + event loop delay exceeds interval

### Key runtime behavior difference

- `setTimeout` creates a **single timer entry**
- `setInterval` creates a **recurring timer entry managed by libuv**
- If the event loop is blocked (CPU-heavy work), both are delayed, but intervals may “pile up” behaviorally depending on timing

### Trade-offs

- `setTimeout` is safer for **controlled scheduling and recursion patterns**
- `setInterval` is simpler but can cause **overlapping executions or drift**
- In production systems, `setTimeout`-based recursion is often preferred for reliability

---

## Example (TypeScript)

```ts
// Node.js >= 18+, TypeScript (ESM)

function periodicTask() {
  console.log("Task executed at", new Date().toISOString());

  // safer alternative to setInterval: recursive setTimeout
  setTimeout(periodicTask, 1000);
}

// start after 1 second
setTimeout(periodicTask, 1000);

// Traditional setInterval example (less precise under load)
const intervalId = setInterval(() => {
  console.log("setInterval tick at", new Date().toISOString());
}, 1000);

// stop after 5 seconds
setTimeout(() => {
  clearInterval(intervalId);
  console.log("Interval cleared");
}, 5000);
```

---

## Testing

- Use fake timers to control time-based behavior.

### Jest example:

```ts
import { jest } from "@jest/globals";

jest.useFakeTimers();

test("setTimeout runs callback", () => {
  const fn = jest.fn();

  setTimeout(fn, 1000);

  jest.advanceTimersByTime(1000);

  expect(fn).toHaveBeenCalledTimes(1);
});
```

Run:

```bash
npx jest
```

---

## Ops & Monitoring

- Avoid long-running intervals without monitoring (can accumulate memory/state leaks)
- Track timer-based tasks with:
  - OpenTelemetry spans for scheduled jobs
  - Metrics: execution duration, drift, missed intervals

- Always ensure cleanup (`clearTimeout`, `clearInterval`) on shutdown signals (`SIGTERM`)
- In PM2/systemd, handle graceful shutdown to avoid orphan timers

---

## Deployment & Scaling

- In clustered setups, `setInterval` runs **per process**, not globally
- For distributed systems, prefer:
  - Redis-based schedulers (BullMQ, Agenda)
  - External cron systems (Kubernetes CronJobs, cloud schedulers)

- In serverless (AWS Lambda):
  - Avoid long intervals; functions are stateless and ephemeral

- Ensure Node.js version ≥ 18 for stable timer behavior and improved event loop performance

---

## Pitfalls and Best Practices

- `setInterval` can cause **overlapping executions** if tasks take longer than interval
- Timer drift increases under CPU load or blocking synchronous code
- Forgetting cleanup leads to **memory leaks and hanging processes**

Best practices:

- Prefer recursive `setTimeout` for precision
- Keep timer callbacks non-blocking
- Always handle shutdown cleanup

## Question 2. How do you use the `crypto` module in Node.js?

## Question 3. How do you generate a random UUID in Node.js?

## Question 4. What are the advantages of using Node.js streams for reading large files?

## Question 5. What is the difference between `fs.readFile` and `fs.createReadStream`?

## Question 6. How do you handle JSON parsing errors in Node.js?

## Question 7. How do you use environment variables in Node.js?

## Question 8. How do you stop a Node.js server programmatically?

## Question 9. Explain the role of `exports` object in Node.js

## Question 10. How do you create a simple TCP server in Node.js?

## Question 11. How do you handle command-line input in Node.js?

## Question 12. What is the difference between Node.js REPL and a script file?

## Question 13. How do you check memory usage in a Node.js application?

## Question 14. How do you handle uncaught exceptions globally in Node.js?

## Question 15. What is the purpose of `npm run` scripts?

## Question 16. Explain the difference between synchronous and asynchronous event handling

## Question 17. What is the difference between `fs.readFile` and `fs.promises.readFile`?

## Question 18. How do you use `util.promisify` in Node.js?

## Question 19. What is the difference between `require.resolve()` and `require()`?

## Question 20. How does Node.js resolve module paths?
