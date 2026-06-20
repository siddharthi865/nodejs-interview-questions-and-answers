# Set 20

| S.No. | Question                                                                                                                                                                                               |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [How do you implement fault-tolerant job processing?](#question-1-how-do-you-implement-fault-tolerant-job-processing)                                                                                  |
| 2.    | [How do you implement graceful degradation of services under load?](#question-2-how-do-you-implement-graceful-degradation-of-services-under-load)                                                      |
| 3.    | [How do you implement circuit breakers using `opossum` or custom logic?](#question-3-how-do-you-implement-circuit-breakers-using-opossum-or-custom-logic)                                              |
| 4.    | [How do you implement dynamic scaling of Node.js workers?](#question-4-how-do-you-implement-dynamic-scaling-of-nodejs-workers)                                                                         |
| 5.    | [How do you implement zero-downtime deployments in clustered Node.js apps?](#question-5-how-do-you-implement-zero-downtime-deployments-in-clustered-nodejs-apps)                                       |
| 6.    | [How do you implement memory leak detection in production?](#question-6-how-do-you-implement-memory-leak-detection-in-production)                                                                      |
| 7.    | [How do you implement CPU profiling using Clinic or 0x?](#question-7-how-do-you-implement-cpu-profiling-using-clinic-or-0x)                                                                            |
| 8.    | [How do you implement streaming large CSVs from a database to clients?](#question-8-how-do-you-implement-streaming-large-csvs-from-a-database-to-clients)                                              |
| 9.    | [How do you implement real-time analytics pipelines using Node.js?](#question-9-how-do-you-implement-real-time-analytics-pipelines-using-nodejs)                                                       |
| 10.   | [How do you implement Pub/Sub messaging patterns for microservices?](#question-10-how-do-you-implement-pubsub-messaging-patterns-for-microservices)                                                    |
| 11.   | [How do you implement message retries with dead-letter queues?](#question-11-how-do-you-implement-message-retries-with-dead-letter-queues)                                                             |
| 12.   | [How do you implement distributed locks with Redis or etcd?](#question-12-how-do-you-implement-distributed-locks-with-redis-or-etcd)                                                                   |
| 13.   | [How do you implement async task pipelines with dependency management?](#question-13-how-do-you-implement-async-task-pipelines-with-dependency-management)                                             |
| 14.   | [How do you implement multi-tenant data isolation in Node.js?](#question-14-how-do-you-implement-multi-tenant-data-isolation-in-nodejs)                                                                |
| 15.   | [How do you implement server push with HTTP/2 in Node.js?](#question-15-how-do-you-implement-server-push-with-http2-in-nodejs)                                                                         |
| 16.   | [How do you secure HTTP/2 connections for Node.js?](#question-16-how-do-you-secure-http2-connections-for-nodejs)                                                                                       |
| 17.   | [How do you implement real-time collaborative editing using Operational Transforms or CRDTs?](#question-17-how-do-you-implement-real-time-collaborative-editing-using-operational-transforms-or-crdts) |
| 18.   | [How do you implement mutual TLS (mTLS) for microservices?](#question-18-how-do-you-implement-mutual-tls-mtls-for-microservices)                                                                       |
| 19.   | [How do you implement observability with metrics, logs, and traces?](#question-19-how-do-you-implement-observability-with-metrics-logs-and-traces)                                                     |
| 20.   | [How do you implement high-availability, fault-tolerant Node.js applications in production?](#question-20-how-do-you-implement-high-availability-fault-tolerant-nodejs-applications-in-production)     |

## Question 1. How do you implement fault-tolerant job processing?

# Short answer

Implement fault-tolerant job processing by combining a **durable message queue**, **idempotent workers**, **retry policies with exponential backoff**, **dead-letter queues (DLQs)**, **visibility timeouts/leases**, **distributed locking when required**, and **observability**. Workers should assume they may crash at any point and that jobs may be delivered more than once.

---

# Explanation

Fault tolerance means that jobs are eventually processed correctly even if:

- A worker crashes.
- The process restarts.
- The machine fails.
- The network is temporarily unavailable.
- An external API is slow or unavailable.

In distributed systems, **exactly-once processing is extremely difficult**. Most production systems implement **at-least-once delivery**, which means a job may execute multiple times. Therefore, **idempotency** becomes the most important design principle.

## Typical architecture

```
Producer
    │
    ▼
Persistent Queue (Redis/RabbitMQ/SQS/Kafka)
    │
    ▼
Multiple Worker Processes
    │
 ┌──┴─────────────┐
 │                │
Success        Failure
 │                │
Ack         Retry (Backoff)
                  │
         Max retries exceeded
                  │
                  ▼
          Dead Letter Queue
```

---

## 1. Durable queue

Never keep jobs only in memory.

Use durable queues such as:

- BullMQ (Redis)
- RabbitMQ
- Amazon SQS
- Kafka
- NATS JetStream

The queue survives application restarts.

---

## 2. Job acknowledgement

Workers should acknowledge jobs **only after successful completion**.

Bad:

```
Receive job
ACK
Process
Crash
```

The job is lost.

Correct:

```
Receive job
Process
ACK
```

If the worker crashes before ACK:

- queue detects timeout
- job becomes available again

---

## 3. Retry with exponential backoff

Transient failures happen frequently:

- network timeout
- database unavailable
- third-party API failure

Instead of immediate retry:

```
Retry 1 → 1 second

Retry 2 → 2 seconds

Retry 3 → 4 seconds

Retry 4 → 8 seconds

Retry 5 → 16 seconds
```

Benefits:

- avoids retry storms
- reduces load
- improves recovery

Often include random jitter:

```
delay = exponential + random
```

---

## 4. Dead Letter Queue (DLQ)

Some jobs will never succeed.

Examples:

- invalid payload
- deleted customer
- corrupted file

After N retries:

```
Main Queue
     │
     ▼
Retry
Retry
Retry
Retry
     │
     ▼
Dead Letter Queue
```

DLQ enables:

- investigation
- replay
- alerting

without blocking healthy jobs.

---

## 5. Idempotent processing

Since jobs may run twice:

```
Charge customer
```

must not charge twice.

Instead:

```
if paymentAlreadyProcessed(id)
    return

charge()

markProcessed(id)
```

Use:

- unique request IDs
- database constraints
- idempotency keys
- UPSERT operations

This is arguably the most important aspect of fault-tolerant workers.

---

## 6. Visibility timeout / lease

When a worker receives a job:

```
Worker A
receives Job #42

Queue hides Job #42
for 60 seconds
```

If Worker A crashes:

```
Timeout expires

Job becomes visible again

Worker B processes it
```

BullMQ uses job locks.

SQS uses visibility timeout.

---

## 7. Circuit breaker

If a downstream API is failing:

Instead of retrying endlessly:

```
API fails

↓

Circuit opens

↓

Fail fast

↓

Recover later
```

Benefits:

- protects workers
- avoids cascading failures
- reduces latency

---

## 8. Graceful shutdown

When Kubernetes or PM2 stops a worker:

```
SIGTERM

↓

Stop receiving new jobs

↓

Finish current jobs

↓

ACK completed jobs

↓

Exit
```

Never terminate immediately.

---

## 9. Horizontal scaling

Run multiple workers.

```
Queue

 ├── Worker 1
 ├── Worker 2
 ├── Worker 3
 └── Worker 4
```

Each worker processes independent jobs.

Node.js processes remain single-threaded for JavaScript execution, while the event loop and **libuv** handle asynchronous I/O. CPU-intensive jobs should be offloaded to `worker_threads` or isolated services to avoid blocking other jobs.

---

## 10. Monitoring

Track:

- queue length
- retry count
- processing latency
- worker crashes
- failure rate
- DLQ size
- throughput
- job age

Alert when:

- retries spike
- queue grows rapidly
- DLQ receives many jobs

---

# Example (JavaScript)

Example using **BullMQ** with retries and exponential backoff.

```javascript
import { Queue, Worker } from "bullmq";

const connection = {
  host: "127.0.0.1",
  port: 6379,
};

const queue = new Queue("emails", { connection });

await queue.add(
  "welcome-email",
  { email: "user@example.com" },
  {
    attempts: 5,
    backoff: {
      type: "exponential",
      delay: 1000,
    },
    removeOnComplete: true,
  },
);

const worker = new Worker(
  "emails",
  async (job) => {
    console.log(`Sending email to ${job.data.email}`);

    // Simulate transient failure
    if (Math.random() < 0.5) {
      throw new Error("Temporary SMTP failure");
    }

    console.log("Email sent successfully");
  },
  { connection },
);

worker.on("completed", (job) => {
  console.log(`Completed job ${job.id}`);
});

worker.on("failed", (job, err) => {
  console.log(`Job ${job?.id} failed: ${err.message}`);
});
```

This example demonstrates:

- durable queue
- automatic retries
- exponential backoff
- worker recovery after crashes
- at-least-once processing

---

# Testing

**Unit testing**

- Mock external services.
- Verify retryable vs. non-retryable errors.
- Test idempotency logic independently.

**Integration testing**

- Run Redis/RabbitMQ in a test container.
- Enqueue jobs and verify:
  - retries occur
  - failed jobs move to DLQ (or failed set)
  - duplicate deliveries do not produce duplicate side effects

Example using Node's built-in test runner:

```bash
node --test
```

Example assertion:

```javascript
import test from "node:test";
import assert from "node:assert/strict";

test("idempotent job executes once", async () => {
  // invoke handler twice with same idempotency key
  assert.equal(processedCount, 1);
});
```

---

# Ops & Monitoring

- Use structured logging (e.g., Pino) with job IDs, correlation IDs, and retry counts.
- Export metrics such as queue depth, processing duration, retries, failures, and DLQ size to Prometheus/Grafana.
- Instrument producers and workers with OpenTelemetry to trace jobs across services.
- Classify errors into retryable and permanent; avoid retrying validation or business logic errors.
- Run workers under a process manager (PM2/systemd) or an orchestrator like Kubernetes, and implement graceful shutdown on `SIGTERM`.

---

# Deployment & Scaling

- Package workers separately from API servers so they can scale independently.
- Use connection pooling for databases and Redis; avoid creating a new connection per job.
- Scale horizontally by adding worker processes or pods, ensuring handlers remain idempotent because duplicate delivery is possible.
- For serverless workers, consider cold-start latency, execution time limits, and queue visibility timeouts.
- Prefer modern LTS Node.js versions (Node.js 20+ or newer LTS) for improved performance and runtime features.

---

# Pitfalls

- **Acknowledging jobs before work completes**, leading to permanent job loss if the worker crashes.
- **Non-idempotent handlers**, causing duplicate emails, payments, or database updates after retries.
- **Retrying every error**, including permanent validation failures, which wastes resources and clogs queues.

## Question 2. How do you implement graceful degradation of services under load?

## Question 3. How do you implement circuit breakers using `opossum` or custom logic?

## Question 4. How do you implement dynamic scaling of Node.js workers?

## Question 5. How do you implement zero-downtime deployments in clustered Node.js apps?

## Question 6. How do you implement memory leak detection in production?

## Question 7. How do you implement CPU profiling using Clinic or 0x?

## Question 8. How do you implement streaming large CSVs from a database to clients?

## Question 9. How do you implement real-time analytics pipelines using Node.js?

## Question 10. How do you implement Pub/Sub messaging patterns for microservices?

## Question 11. How do you implement message retries with dead-letter queues?

## Question 12. How do you implement distributed locks with Redis or etcd?

## Question 13. How do you implement async task pipelines with dependency management?

## Question 14. How do you implement multi-tenant data isolation in Node.js?

## Question 15. How do you implement server push with HTTP/2 in Node.js?

## Question 16. How do you secure HTTP/2 connections for Node.js?

## Question 17. How do you implement real-time collaborative editing using Operational Transforms or CRDTs?

## Question 18. How do you implement mutual TLS (mTLS) for microservices?

## Question 19. How do you implement observability with metrics, logs, and traces?

## Question 20. How do you implement high-availability, fault-tolerant Node.js applications in production?
