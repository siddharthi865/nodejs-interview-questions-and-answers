# Set 24

| S.No. | Question                                                                                                                                                               |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you implement automated cleanup of failed jobs in queues?](#question-1-how-do-you-implement-automated-cleanup-of-failed-jobs-in-queues)                        |
| 2.    | [How do you implement monitoring of background jobs with dashboards?](#question-2-how-do-you-implement-monitoring-of-background-jobs-with-dashboards)                  |
| 3.    | [How do you implement structured logs with JSON format?](#question-3-how-do-you-implement-structured-logs-with-json-format)                                            |
| 4.    | [How do you implement log rotation in Node.js applications?](#question-4-how-do-you-implement-log-rotation-in-nodejs-applications)                                     |
| 5.    | [How do you implement request tracing with correlation IDs?](#question-5-how-do-you-implement-request-tracing-with-correlation-ids)                                    |
| 6.    | [How do you implement dynamic configuration reload in running applications?](#question-6-how-do-you-implement-dynamic-configuration-reload-in-running-applications)    |
| 7.    | [How do you implement multi-environment configuration management?](#question-7-how-do-you-implement-multi-environment-configuration-management)                        |
| 8.    | [How do you implement input sanitization to prevent NoSQL injection?](#question-8-how-do-you-implement-input-sanitization-to-prevent-nosql-injection)                  |
| 9.    | [How do you implement API key authentication and management?](#question-9-how-do-you-implement-api-key-authentication-and-management)                                  |
| 10.   | [How do you implement rate-limited file download endpoints?](#question-10-how-do-you-implement-rate-limited-file-download-endpoints)                                   |
| 11.   | [How do you implement distributed cache invalidation strategies?](#question-11-how-do-you-implement-distributed-cache-invalidation-strategies)                         |
| 12.   | [How do you implement horizontal scaling for background job workers?](#question-12-how-do-you-implement-horizontal-scaling-for-background-job-workers)                 |
| 13.   | [How do you implement message ordering guarantees with Kafka or RabbitMQ?](#question-13-how-do-you-implement-message-ordering-guarantees-with-kafka-or-rabbitmq)       |
| 14.   | [How do you implement distributed tracing with OpenTelemetry for Node.js?](#question-14-how-do-you-implement-distributed-tracing-with-opentelemetry-for-nodejs)        |
| 15.   | [How do you implement event-driven architecture with Node.js microservices?](#question-15-how-do-you-implement-event-driven-architecture-with-nodejs-microservices)    |
| 16.   | [How do you implement backpressure handling in streams with async iterators?](#question-16-how-do-you-implement-backpressure-handling-in-streams-with-async-iterators) |
| 17.   | [How do you implement hot reloading in production without downtime?](#question-17-how-do-you-implement-hot-reloading-in-production-without-downtime)                   |
| 18.   | [How do you implement zero-downtime deployments with PM2 or Kubernetes?](#question-18-how-do-you-implement-zero-downtime-deployments-with-pm2-or-kubernetes)           |
| 19.   | [How do you implement cluster-wide session sharing?](#question-19-how-do-you-implement-cluster-wide-session-sharing)                                                   |
| 20.   | [How do you implement circuit breaker patterns with fallback strategies?](#question-20-how-do-you-implement-circuit-breaker-patterns-with-fallback-strategies)         |

## Question 1. How do you implement automated cleanup of failed jobs in queues?

# Short answer

Implement automated cleanup by configuring the queue to automatically remove completed and failed jobs after a retention period or count, while retaining enough failed jobs for debugging. For production systems, combine this with retry policies, dead-letter queues (DLQs), periodic cleanup tasks, and monitoring to prevent Redis/database growth.

---

# Explanation

Most Node.js queue systems (e.g., **BullMQ**, Bull, Bee-Queue, Agenda) persist job metadata. If failed jobs are never removed:

- Redis/database size continuously grows.
- Queue operations become slower.
- Memory usage increases.
- Dashboard queries become expensive.

A production strategy usually consists of the following.

### 1. Automatic cleanup

Configure workers to remove jobs automatically.

- Remove successful jobs immediately or keep only the last N jobs.
- Retain failed jobs for a limited period or limited count.
- This is the preferred approach over manually deleting jobs.

Example policies:

- Keep last **1,000 completed** jobs.
- Keep failed jobs for **7 days**.
- Remove everything older automatically.

---

### 2. Retry before cleanup

Don't immediately delete failed jobs.

Use:

- exponential backoff
- retry limits
- jitter

Example:

- Retry 5 times.
- Backoff: 2s → 4s → 8s → 16s → 32s.
- Remove only after retries are exhausted.

---

### 3. Dead Letter Queue (DLQ)

Critical jobs should be moved to another queue instead of being deleted.

Example:

```
Payment Queue
        │
        ▼
Failed after retries
        │
        ▼
Dead Letter Queue
        │
        ▼
Investigation / Manual replay
```

This prevents losing important business events.

---

### 4. Scheduled cleanup

Some queue libraries expose cleanup APIs.

Typical scheduled cleanup:

- every hour
- every day
- remove failed jobs older than X days
- remove completed jobs older than X hours

This keeps storage predictable.

---

### 5. Monitoring

Track:

- failed jobs
- retry count
- queue depth
- Redis memory
- cleanup duration

OpenTelemetry, Prometheus, and Grafana can alert when failed jobs accumulate unexpectedly.

---

### 6. Performance considerations

Keeping millions of completed jobs causes:

- slower Redis scans
- larger persistence files
- longer startup times
- slower queue dashboards

Automatic cleanup dramatically reduces Redis memory usage.

---

# Example (TypeScript - BullMQ)

```typescript
import { Queue, Worker } from "bullmq";

const queue = new Queue("emails", {
  connection: {
    host: "localhost",
    port: 6379,
  },
});

new Worker(
  "emails",
  async (job) => {
    console.log(`Sending email to ${job.data.email}`);

    // Simulate failure
    throw new Error("SMTP unavailable");
  },
  {
    connection: {
      host: "localhost",
      port: 6379,
    },

    // Retry policy
    attempts: 5,

    backoff: {
      type: "exponential",
      delay: 2000,
    },

    // Cleanup policy
    removeOnComplete: {
      count: 1000,
    },

    removeOnFail: {
      age: 7 * 24 * 60 * 60, // 7 days
      count: 5000,
    },
  },
);

await queue.add("welcome-email", {
  email: "user@example.com",
});
```

This configuration:

- retries failed jobs five times,
- uses exponential backoff,
- keeps only the latest 1,000 successful jobs,
- removes failed jobs after 7 days while capping retained failures at 5,000.

---

# Testing

**Unit tests**

- Mock the processor to throw errors.
- Verify retry behavior.
- Verify cleanup options are configured correctly.

**Integration tests**

- Run against a real Redis instance.
- Enqueue failing jobs.
- Confirm jobs transition to the failed state, respect retry limits, and are eventually removed according to the configured retention policy.

Example using the built-in Node.js test runner:

```bash
node --test
```

---

# Ops & Monitoring

- Log job ID, queue name, attempt number, and error stack for each failure.
- Export metrics such as queue depth, failed job count, retry count, processing latency, and cleanup duration.
- Instrument workers with **OpenTelemetry** for distributed tracing.
- Configure alerts when failed jobs exceed expected thresholds or cleanup falls behind.
- Run workers under a process manager (PM2, systemd, or Kubernetes) with graceful shutdown so active jobs finish or are re-queued safely.

---

# Deployment & Scaling

- Use Redis persistence settings appropriate for your durability requirements.
- Scale workers horizontally; queue libraries coordinate work through Redis.
- Separate queues for CPU-intensive and I/O-intensive workloads to avoid resource contention.
- Tune Redis memory limits and eviction policies, but rely on queue cleanup rather than eviction to manage growth.
- Use a current LTS version of Node.js (20.x or later) for improved performance and stability.

---

# Pitfalls

- **Never remove failed jobs immediately** if they are needed for debugging or compliance.
- **Avoid unlimited retention**, which can eventually exhaust Redis memory.
- **Don't rely solely on retries**; use a DLQ for permanently failing or business-critical jobs.

## Question 2. How do you implement monitoring of background jobs with dashboards?

## Question 3. How do you implement structured logs with JSON format?

## Question 4. How do you implement log rotation in Node.js applications?

## Question 5. How do you implement request tracing with correlation IDs?

## Question 6. How do you implement dynamic configuration reload in running applications?

## Question 7. How do you implement multi-environment configuration management?

## Question 8. How do you implement input sanitization to prevent NoSQL injection?

## Question 9. How do you implement API key authentication and management?

## Question 10. How do you implement rate-limited file download endpoints?

## Question 11. How do you implement distributed cache invalidation strategies?

## Question 12. How do you implement horizontal scaling for background job workers?

## Question 13. How do you implement message ordering guarantees with Kafka or RabbitMQ?

## Question 14. How do you implement distributed tracing with OpenTelemetry for Node.js?

## Question 15. How do you implement event-driven architecture with Node.js microservices?

## Question 16. How do you implement backpressure handling in streams with async iterators?

## Question 17. How do you implement hot reloading in production without downtime?

## Question 18. How do you implement zero-downtime deployments with PM2 or Kubernetes?

## Question 19. How do you implement cluster-wide session sharing?

## Question 20. How do you implement circuit breaker patterns with fallback strategies?
