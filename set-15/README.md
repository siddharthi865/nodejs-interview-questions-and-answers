# Set 15

| S.No. | Question                                                                                                                                                                        |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you implement streaming of large files to clients efficiently?](#question-1-how-do-you-implement-streaming-of-large-files-to-clients-efficiently)                       |
| 2.    | [How do you handle large JSON payloads without consuming too much memory?](#question-2-how-do-you-handle-large-json-payloads-without-consuming-too-much-memory)                 |
| 3.    | [How do you implement a WebSocket server for real-time notifications?](#question-3-how-do-you-implement-a-websocket-server-for-real-time-notifications)                         |
| 4.    | [How do you implement pub/sub messaging patterns with Node.js?](#question-4-how-do-you-implement-pubsub-messaging-patterns-with-nodejs)                                         |
| 5.    | [How do you implement inter-service communication with gRPC in Node.js?](#question-5-how-do-you-implement-inter-service-communication-with-grpc-in-nodejs)                      |
| 6.    | [How do you implement tracing using OpenTelemetry in Node.js?](#question-6-how-do-you-implement-tracing-using-opentelemetry-in-nodejs)                                          |
| 7.    | [How do you measure Node.js application performance with `perf_hooks`?](#question-7-how-do-you-measure-nodejs-application-performance-with-perf_hooks)                          |
| 8.    | [How do you implement memory profiling in Node.js?](#question-8-how-do-you-implement-memory-profiling-in-nodejs)                                                                |
| 9.    | [How do you detect and fix memory leaks in Node.js?](#question-9-how-do-you-detect-and-fix-memory-leaks-in-nodejs)                                                              |
| 10.   | [How do you implement circuit breakers in Node.js microservices?](#question-10-how-do-you-implement-circuit-breakers-in-nodejs-microservices)                                   |
| 11.   | [How do you implement caching strategies for database queries?](#question-11-how-do-you-implement-caching-strategies-for-database-queries)                                      |
| 12.   | [How do you implement bulk data processing in Node.js without blocking?](#question-12-how-do-you-implement-bulk-data-processing-in-nodejs-without-blocking)                     |
| 13.   | [How do you implement horizontal scaling for WebSocket servers?](#question-13-how-do-you-implement-horizontal-scaling-for-websocket-servers)                                    |
| 14.   | [How do you implement distributed locks for shared resources?](#question-14-how-do-you-implement-distributed-locks-for-shared-resources)                                        |
| 15.   | [How do you implement message queue consumers that scale horizontally?](#question-15-how-do-you-implement-message-queue-consumers-that-scale-horizontally)                      |
| 16.   | [How do you implement async job pipelines with dependency order?](#question-16-how-do-you-implement-async-job-pipelines-with-dependency-order)                                  |
| 17.   | [How do you implement multi-tenancy in a Node.js application?](#question-17-how-do-you-implement-multi-tenancy-in-a-nodejs-application)                                         |
| 18.   | [How do you implement server push with HTTP/2 in Node.js?](#question-18-how-do-you-implement-server-push-with-http2-in-nodejs)                                                  |
| 19.   | [How do you secure WebSocket connections with SSL/TLS?](#question-19-how-do-you-secure-websocket-connections-with-ssltls)                                                       |
| 20.   | [How do you implement monitoring and alerting for Node.js services in production?](#question-20-how-do-you-implement-monitoring-and-alerting-for-nodejs-services-in-production) |

## Question 1. How do you implement streaming of large files to clients efficiently?

# Short answer

Use **Node.js streams** (`fs.createReadStream()` + `stream.pipeline()`) to stream files directly to the HTTP response instead of loading the entire file into memory. Support **backpressure**, **Range requests** (for resumable downloads and media streaming), proper caching headers, and efficient error handling. This minimizes memory usage and maximizes throughput.

---

# Explanation

Streaming is the preferred way to serve large files because Node.js streams transfer data in **small chunks** rather than reading the entire file into RAM.

Typical flow:

```
Disk -> Readable Stream -> (Optional Transform: gzip/encryption) -> HTTP Response
```

### Why streaming is efficient

Instead of:

```js
const data = await fs.promises.readFile("large.iso");
res.end(data);
```

which loads the entire file into memory,

use:

```js
fs.createReadStream("large.iso").pipe(res);
```

Memory usage remains nearly constant regardless of file size.

---

## How backpressure works

Node streams automatically implement **backpressure**.

If:

- the disk reads faster than the network writes,
- Node pauses reading,
- waits until the socket drains,
- resumes automatically.

This prevents:

- excessive RAM usage
- event loop congestion
- unnecessary buffering

This is one of the biggest reasons streams scale well under high concurrency.

---

## Use `stream.pipeline()`

Avoid manually piping streams when possible.

```js
pipeline(readStream, res, callback);
```

Benefits:

- propagates errors correctly
- closes streams automatically
- avoids resource leaks
- recommended by Node.js documentation

---

## Support HTTP Range requests

Large downloads and video streaming typically require:

```
Range: bytes=1000000-
```

The server responds with:

```
206 Partial Content
```

Advantages:

- resume interrupted downloads
- seek within videos
- better CDN compatibility
- improved browser support

---

## Tune `highWaterMark`

Example:

```js
fs.createReadStream(file, {
  highWaterMark: 64 * 1024,
});
```

Larger values:

- fewer system calls
- higher throughput
- slightly more memory

Smaller values:

- lower memory
- more CPU overhead

Default values are usually sufficient.

---

## Zero-copy optimizations

Modern Node.js delegates much of the work to the operating system.

The flow becomes approximately:

```
Disk
 ↓
Kernel buffer
 ↓
Socket
```

Minimal JavaScript involvement means:

- lower CPU usage
- fewer memory copies
- better scalability

---

## Compression

For text files:

```
ReadStream
     ↓
gzip
     ↓
HTTP response
```

Avoid compressing:

- ZIP
- MP4
- JPEG
- PNG

These formats are already compressed.

---

## Production considerations

For very large downloads:

- support `ETag`
- support `Last-Modified`
- support `If-None-Match`
- send `Content-Length`
- support `HEAD`
- support `Accept-Ranges`
- log incomplete downloads
- monitor throughput

---

# Example (JavaScript)

```javascript
import http from "node:http";
import fs from "node:fs";
import { pipeline } from "node:stream";

const server = http.createServer((req, res) => {
  const filePath = "./large-video.mp4";

  fs.stat(filePath, (err, stats) => {
    if (err) {
      res.statusCode = 404;
      return res.end("File not found");
    }

    res.writeHead(200, {
      "Content-Type": "video/mp4",
      "Content-Length": stats.size,
      "Accept-Ranges": "bytes",
    });

    const stream = fs.createReadStream(filePath);

    pipeline(stream, res, (err) => {
      if (err) {
        console.error("Streaming failed:", err);
        if (!res.headersSent) {
          res.statusCode = 500;
          res.end("Internal Server Error");
        }
      }
    });
  });
});

server.listen(3000, () => {
  console.log("Listening on http://localhost:3000");
});
```

This example:

- never loads the whole file into memory
- automatically handles backpressure
- cleans up resources on failure
- scales well for concurrent downloads

---

# Testing

### Unit tests

- Mock `fs.createReadStream()` and verify headers (`Content-Length`, `Accept-Ranges`) are set correctly.
- Simulate stream errors and ensure the server returns an appropriate error response without leaking resources.

### Integration tests

- Download a large test file and verify its checksum matches the source.
- Test interrupted connections and resume behavior if Range support is implemented.

Example using the built-in test runner:

```bash
node --test
```

---

# Ops & Monitoring

- Log request metadata (file, size, duration, client IP, bytes sent) using structured logging (e.g., Pino).
- Monitor throughput, response latency, open file descriptors, and event loop delay.
- Instrument streaming paths with OpenTelemetry spans to identify slow disk or network operations.
- Handle client disconnects (`req.aborted`/response close events) to stop reading and free resources.
- Run under a process manager (PM2/systemd) or containers with health checks and graceful shutdown.

---

# Deployment & Scaling

- Use current LTS Node.js versions for the latest stream improvements.
- In containers, mount large files on fast storage and avoid copying assets into writable layers unnecessarily.
- Offload static large-file delivery to a reverse proxy or object storage/CDN (e.g., NGINX or cloud object storage) when possible; let Node handle authorization and signed URLs.
- Enable connection pooling for upstream services if streams originate from remote storage.
- In serverless environments, avoid proxying very large files through the function when direct object storage downloads are available to reduce cold-start and execution costs.
- Scale horizontally since streaming is largely I/O-bound and benefits from multiple instances behind a load balancer.

---

# Pitfalls

- **Avoid `fs.readFile()`** for large files—it loads the entire file into memory.
- **Don't ignore stream errors**—use `stream.pipeline()` to ensure proper cleanup and error propagation.
- **Implement Range requests** for media and resumable downloads; many clients expect `206 Partial Content` support.

## Question 2. How do you handle large JSON payloads without consuming too much memory?

## Question 3. How do you implement a WebSocket server for real-time notifications?

## Question 4. How do you implement pub/sub messaging patterns with Node.js?

## Question 5. How do you implement inter-service communication with gRPC in Node.js?

## Question 6. How do you implement tracing using OpenTelemetry in Node.js?

## Question 7. How do you measure Node.js application performance with `perf_hooks`?

## Question 8. How do you implement memory profiling in Node.js?

## Question 9. How do you detect and fix memory leaks in Node.js?

## Question 10. How do you implement circuit breakers in Node.js microservices?

## Question 11. How do you implement caching strategies for database queries?

## Question 12. How do you implement bulk data processing in Node.js without blocking?

## Question 13. How do you implement horizontal scaling for WebSocket servers?

## Question 14. How do you implement distributed locks for shared resources?

## Question 15. How do you implement message queue consumers that scale horizontally?

## Question 16. How do you implement async job pipelines with dependency order?

## Question 17. How do you implement multi-tenancy in a Node.js application?

## Question 18. How do you implement server push with HTTP/2 in Node.js?

## Question 19. How do you secure WebSocket connections with SSL/TLS?

## Question 20. How do you implement monitoring and alerting for Node.js services in production?
