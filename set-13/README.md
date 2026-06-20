# Set 13

| S.No. | Question                                                                                                                                                      |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you implement gzip response compression in Express.js?](#question-1-how-do-you-implement-gzip-response-compression-in-expressjs)                      |
| 2.    | [How do you implement streaming CSV or JSON responses?](#question-2-how-do-you-implement-streaming-csv-or-json-responses)                                     |
| 3.    | [How do you implement streaming large datasets from a database?](#question-3-how-do-you-implement-streaming-large-datasets-from-a-database)                   |
| 4.    | [How do you use the `events.once()` method in Node.js?](#question-4-how-do-you-use-the-eventsonce-method-in-nodejs)                                           |
| 5.    | [How do you implement promises with the EventEmitter pattern?](#question-5-how-do-you-implement-promises-with-the-eventemitter-pattern)                       |
| 6.    | [How do you use `Promise.allSettled()` in Node.js?](#question-6-how-do-you-use-promiseallsettled-in-nodejs)                                                   |
| 7.    | [How do you handle concurrency with `Promise.race()`?](#question-7-how-do-you-handle-concurrency-with-promiserace)                                            |
| 8.    | [How do you use `async_hooks` module in Node.js?](#question-8-how-do-you-use-async_hooks-module-in-nodejs)                                                    |
| 9.    | [How do you implement structured logging using `pino`?](#question-9-how-do-you-implement-structured-logging-using-pino)                                       |
| 10.   | [How do you implement request correlation IDs for tracing?](#question-10-how-do-you-implement-request-correlation-ids-for-tracing)                            |
| 11.   | [How do you handle session persistence across clustered Node.js servers?](#question-11-how-do-you-handle-session-persistence-across-clustered-nodejs-servers) |
| 12.   | [How do you implement sticky sessions with `express-session`?](#question-12-how-do-you-implement-sticky-sessions-with-express-session)                        |
| 13.   | [How do you handle CORS preflight requests?](#question-13-how-do-you-handle-cors-preflight-requests)                                                          |
| 14.   | [How do you prevent cross-site scripting (XSS) in Node.js applications?](#question-14-how-do-you-prevent-cross-site-scripting-xss-in-nodejs-applications)     |
| 15.   | [How do you validate input to prevent SQL/NoSQL injection attacks?](#question-15-how-do-you-validate-input-to-prevent-sqlnosql-injection-attacks)             |
| 16.   | [How do you implement rate limiting to prevent brute-force attacks?](#question-16-how-do-you-implement-rate-limiting-to-prevent-brute-force-attacks)          |
| 17.   | [How do you implement multi-factor authentication in Node.js?](#question-17-how-do-you-implement-multi-factor-authentication-in-nodejs)                       |
| 18.   | [How do you implement OAuth2 login with Google/Facebook in Node.js?](#question-18-how-do-you-implement-oauth2-login-with-googlefacebook-in-nodejs)            |
| 19.   | [How do you implement file streaming to S3 or cloud storage?](#question-19-how-do-you-implement-file-streaming-to-s3-or-cloud-storage)                        |
| 20.   | [How do you set request body size limits in Express.js?](#question-20-how-do-you-set-request-body-size-limits-in-expressjs)                                   |

## Question 1. How do you implement gzip response compression in Express.js?

# Short answer

In Express.js, the standard way to enable gzip (and Brotli where supported) response compression is to use the `compression` middleware. It compresses responses based on the client's `Accept-Encoding` header, reducing bandwidth and improving page/API response times at the cost of some CPU usage.

---

# Explanation

The `compression` middleware transparently compresses HTTP responses using Node.js's built-in `zlib` module.

### How it works

1. The client sends:

```http
Accept-Encoding: gzip, br, deflate
```

2. Express checks whether the response is compressible (based on `Content-Type`).

3. If compression is appropriate:
   - Response data is streamed through `zlib`.
   - Express sets:

```http
Content-Encoding: gzip
Vary: Accept-Encoding
```

4. The browser or HTTP client automatically decompresses the response.

---

### Installation

```bash
npm install compression
```

---

### Production Example (JavaScript)

```javascript
import express from "express";
import compression from "compression";

const app = express();

// Enable gzip/Brotli compression
app.use(
  compression({
    level: 6, // Good balance between CPU and compression
    threshold: 1024, // Compress responses larger than 1 KB
  }),
);

app.get("/users", async (req, res) => {
  const users = Array.from({ length: 10000 }, (_, i) => ({
    id: i,
    name: `User ${i}`,
  }));

  res.json(users);
});

app.listen(3000, () => {
  console.log("Server running on http://localhost:3000");
});
```

Test it:

```bash
curl -H "Accept-Encoding: gzip" -I http://localhost:3000/users
```

Expected:

```http
Content-Encoding: gzip
Vary: Accept-Encoding
```

---

### When responses are compressed

Typically compressed:

- JSON APIs
- HTML
- CSS
- JavaScript
- SVG
- XML
- Text files

Usually **not** compressed:

- JPEG
- PNG
- GIF
- WebP
- MP4
- ZIP
- PDF

These formats are already compressed, so gzip wastes CPU with little or no size reduction.

---

### Custom filtering

You can decide which responses should be compressed.

```javascript
import compression from "compression";

app.use(
  compression({
    filter(req, res) {
      if (req.headers["x-no-compression"]) {
        return false;
      }

      return compression.filter(req, res);
    },
  }),
);
```

This allows clients to disable compression:

```http
GET /users
X-No-Compression: true
```

---

### Performance considerations

**Advantages**

- Smaller payloads
- Faster page loads
- Lower bandwidth costs
- Better performance over slower networks

**Trade-offs**

- Uses CPU for compression
- High compression levels increase latency
- Minimal benefit for already-compressed assets

A compression level of **5–7** is generally a good production balance.

---

### Reverse proxy considerations

In production, many deployments terminate compression at a reverse proxy such as:

- NGINX
- Apache HTTP Server
- Envoy

Reasons include:

- Better CPU utilization
- Centralized compression configuration
- Easier caching
- Consistent behavior across services

If your proxy already compresses responses, disable Express compression to avoid unnecessary work.

---

### Brotli support

Modern Node.js versions support Brotli (`br`) via `zlib`.

If the client advertises:

```http
Accept-Encoding: br, gzip
```

the middleware can serve Brotli, which often achieves 15–25% better compression than gzip for text assets, though compression may require more CPU.

---

# Testing

### Unit testing

Use **Jest** or Node's built-in test runner with **Supertest** to verify the middleware.

Example:

```javascript
import request from "supertest";

test("returns gzip response", async () => {
  const res = await request(app).get("/users").set("Accept-Encoding", "gzip");

  expect(res.headers["content-encoding"]).toBe("gzip");
});
```

Run:

```bash
npm test
```

or

```bash
node --test
```

### Integration testing

Verify:

- `Content-Encoding` is present when appropriate.
- Small responses below the threshold are not compressed.
- Binary responses are not compressed.
- Responses remain valid after decompression.

---

# Ops & Monitoring

- Log response sizes before and after compression to measure bandwidth savings.
- Track CPU usage, as compression is CPU-intensive under heavy load.
- Export metrics such as compression ratio and response latency using OpenTelemetry.
- Monitor request latency (P95/P99) to ensure compression doesn't become a bottleneck.
- Handle compression errors gracefully by allowing responses to fall back to uncompressed output if streaming compression fails.

---

# Deployment & Scaling

- Prefer enabling compression at a reverse proxy or load balancer when serving many applications.
- In containerized deployments, ensure CPU limits are sufficient, since compression consumes CPU.
- Horizontal scaling can offset CPU overhead introduced by compression.
- Serverless environments should consider cold-start CPU budgets; compress only responses large enough to justify the cost.
- Use a current LTS version of Node.js (20.x or newer) for the latest `zlib` performance improvements and Brotli support.

---

# Pitfalls

- **Don't compress already compressed files** (images, videos, ZIPs).
- **Avoid very high compression levels (8–9)** unless bandwidth savings outweigh increased CPU and latency.
- **Ensure the `Vary: Accept-Encoding` header is present** so caches store separate compressed and uncompressed variants.

## Question 2. How do you implement streaming CSV or JSON responses?

## Question 3. How do you implement streaming large datasets from a database?

## Question 4. How do you use the `events.once()` method in Node.js?

## Question 5. How do you implement promises with the EventEmitter pattern?

## Question 6. How do you use `Promise.allSettled()` in Node.js?

## Question 7. How do you handle concurrency with `Promise.race()`?

## Question 8. How do you use `async_hooks` module in Node.js?

## Question 9. How do you implement structured logging using `pino`?

## Question 10. How do you implement request correlation IDs for tracing?

## Question 11. How do you handle session persistence across clustered Node.js servers?

## Question 12. How do you implement sticky sessions with `express-session`?

## Question 13. How do you handle CORS preflight requests?

## Question 14. How do you prevent cross-site scripting (XSS) in Node.js applications?

## Question 15. How do you validate input to prevent SQL/NoSQL injection attacks?

## Question 16. How do you implement rate limiting to prevent brute-force attacks?

## Question 17. How do you implement multi-factor authentication in Node.js?

## Question 18. How do you implement OAuth2 login with Google/Facebook in Node.js?

## Question 19. How do you implement file streaming to S3 or cloud storage?

## Question 20. How do you set request body size limits in Express.js?
