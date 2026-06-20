# Set 17

| S.No. | Question                                                                                                                                           |
| ----- | -------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you generate hash digests using the `crypto` module?](#question-1-how-do-you-generate-hash-digests-using-the-crypto-module)                |
| 2.    | [How do you encrypt and decrypt data using Node.js?](#question-2-how-do-you-encrypt-and-decrypt-data-using-nodejs)                                 |
| 3.    | [How do you convert strings to buffers and vice versa?](#question-3-how-do-you-convert-strings-to-buffers-and-vice-versa)                          |
| 4.    | [How do you serialize and deserialize objects in Node.js?](#question-4-how-do-you-serialize-and-deserialize-objects-in-nodejs)                     |
| 5.    | [How do you check the Node.js architecture (32-bit vs 64-bit)?](#question-5-how-do-you-check-the-nodejs-architecture-32-bit-vs-64-bit)             |
| 6.    | [How do you get the current working directory in Node.js?](#question-6-how-do-you-get-the-current-working-directory-in-nodejs)                     |
| 7.    | [How do you change the current working directory programmatically?](#question-7-how-do-you-change-the-current-working-directory-programmatically)  |
| 8.    | [How do you access command-line options passed to a Node.js script?](#question-8-how-do-you-access-command-line-options-passed-to-a-nodejs-script) |
| 9.    | [How do you use `url.parse()` and `url.format()` in Node.js?](#question-9-how-do-you-use-urlparse-and-urlformat-in-nodejs)                         |
| 10.   | [How do you create a simple echo TCP client and server?](#question-10-how-do-you-create-a-simple-echo-tcp-client-and-server)                       |
| 11.   | [How do you create a simple UDP client and server?](#question-11-how-do-you-create-a-simple-udp-client-and-server)                                 |
| 12.   | [How do you implement basic logging using `fs`?](#question-12-how-do-you-implement-basic-logging-using-fs)                                         |
| 13.   | [How do you check Node.js version compatibility for a package?](#question-13-how-do-you-check-nodejs-version-compatibility-for-a-package)          |
| 14.   | [How do you initialize a new npm project with default values?](#question-14-how-do-you-initialize-a-new-npm-project-with-default-values)           |
| 15.   | [How do you uninstall a Node.js package?](#question-15-how-do-you-uninstall-a-nodejs-package)                                                      |
| 16.   | [How do you implement API rate limiting using Redis in Node.js?](#question-16-how-do-you-implement-api-rate-limiting-using-redis-in-nodejs)        |
| 17.   | [How do you implement request throttling for specific endpoints?](#question-17-how-do-you-implement-request-throttling-for-specific-endpoints)     |
| 18.   | [How do you implement request deduplication in Node.js?](#question-18-how-do-you-implement-request-deduplication-in-nodejs)                        |
| 19.   | [How do you implement token-based authentication without sessions?](#question-19-how-do-you-implement-token-based-authentication-without-sessions) |
| 20.   | [How do you implement role-based access control using middleware?](#question-20-how-do-you-implement-role-based-access-control-using-middleware)   |

## Question 1. How do you generate hash digests using the `crypto` module?

# Short answer

Use the built-in `crypto` module's `createHash()` API. Create a hash object, update it with the input data (string, `Buffer`, or stream), and call `digest()` to produce the final hash in a format such as `hex` or `base64`.

---

# Explanation

The `crypto` module provides access to cryptographic hash algorithms like **SHA-256**, **SHA-512**, and **SHA-3** (depending on the OpenSSL version bundled with Node.js).

Basic flow:

1. Create a hash instance with the desired algorithm.
2. Feed data using one or more `update()` calls.
3. Finalize with `digest()`.

```text
Input Data
     │
     ▼
createHash('sha256')
     │
update(data)
     │
digest('hex')
     │
     ▼
Hash Digest
```

### Common algorithms

- `sha256` → General-purpose, widely recommended.
- `sha512` → Larger digest, slightly higher CPU cost.
- `sha1` → Cryptographically broken; avoid for security-sensitive use.
- `md5` → Fast but insecure; only suitable for non-security checksums.

### Important characteristics

- Hashes are **one-way** functions—you cannot recover the original data.
- The same input always produces the same digest.
- Even a one-byte change produces a completely different hash (avalanche effect).

### Streaming large files

For large inputs, avoid loading everything into memory. Pipe the file through a stream and update the hash incrementally.

### Password hashing

Do **not** use `createHash()` for password storage. Instead use dedicated password hashing algorithms such as:

- `crypto.scrypt()`
- `crypto.pbkdf2()`
- Argon2 (third-party library)

These are intentionally slow and resistant to brute-force attacks.

### Performance considerations

- Hashing is CPU work performed by OpenSSL.
- Small payloads are very fast.
- Hashing very large files can consume CPU; streaming avoids excessive memory usage.
- For extremely CPU-intensive workloads, consider using `worker_threads` to prevent blocking the event loop.

---

# Example (JavaScript)

```javascript
import { createHash } from "node:crypto";

const data = "Hello, Node.js!";

const hash = createHash("sha256").update(data, "utf8").digest("hex");

console.log(hash);
// Example:
// 8ef5c0... (64-character SHA-256 digest)
```

### Hashing a file efficiently

```javascript
import { createHash } from "node:crypto";
import { createReadStream } from "node:fs";

async function hashFile(path) {
  return new Promise((resolve, reject) => {
    const hash = createHash("sha256");
    const stream = createReadStream(path);

    stream.on("data", (chunk) => hash.update(chunk));
    stream.on("error", reject);
    stream.on("end", () => resolve(hash.digest("hex")));
  });
}

hashFile("./large-file.zip").then(console.log);
```

---

# Testing

### Unit testing

Verify that known inputs generate expected digests.

Using the built-in Node.js test runner:

```javascript
import test from "node:test";
import assert from "node:assert/strict";
import { createHash } from "node:crypto";

test("SHA-256 hash", () => {
  const digest = createHash("sha256").update("abc").digest("hex");

  assert.equal(
    digest,
    "ba7816bf8f01cfea414140de5dae2223b00361a396177a9cb410ff61f20015ad",
  );
});
```

Run:

```bash
node --test
```

Integration testing should verify file hashing, streamed inputs, and interoperability with hashes produced by other systems or languages.

---

# Ops & Monitoring

- Log the algorithm used (`sha256`, `sha512`) but **never** log sensitive input data.
- Monitor CPU utilization if hashing large files or high request volumes.
- Instrument hashing-heavy endpoints with OpenTelemetry spans to identify latency hotspots.
- Handle stream errors correctly when hashing files.
- Use worker threads for CPU-heavy hashing workloads if they noticeably impact event-loop responsiveness.

---

# Deployment & Scaling

- Use modern Node.js LTS versions (Node.js 20+ recommended) for current OpenSSL support.
- Stream files instead of buffering them entirely in memory.
- Hashing is stateless, making it easy to scale horizontally across containers or instances.
- For serverless environments, avoid hashing unnecessarily large payloads within request time limits.
- Ensure consistent OpenSSL versions across environments if algorithm availability is important.

---

# Pitfalls

- **Don't use MD5 or SHA-1** for security-sensitive applications.
- **Don't use `createHash()` for password storage**—use `scrypt()` or `pbkdf2()` instead.
- **Don't load multi-GB files into memory**; hash them using streams.

## Question 2. How do you encrypt and decrypt data using Node.js?

## Question 3. How do you convert strings to buffers and vice versa?

## Question 4. How do you serialize and deserialize objects in Node.js?

## Question 5. How do you check the Node.js architecture (32-bit vs 64-bit)?

## Question 6. How do you get the current working directory in Node.js?

## Question 7. How do you change the current working directory programmatically?

## Question 8. How do you access command-line options passed to a Node.js script?

## Question 9. How do you use `url.parse()` and `url.format()` in Node.js?

## Question 10. How do you create a simple echo TCP client and server?

## Question 11. How do you create a simple UDP client and server?

## Question 12. How do you implement basic logging using `fs`?

## Question 13. How do you check Node.js version compatibility for a package?

## Question 14. How do you initialize a new npm project with default values?

## Question 15. How do you uninstall a Node.js package?

## Question 16. How do you implement API rate limiting using Redis in Node.js?

## Question 17. How do you implement request throttling for specific endpoints?

## Question 18. How do you implement request deduplication in Node.js?

## Question 19. How do you implement token-based authentication without sessions?

## Question 20. How do you implement role-based access control using middleware?
