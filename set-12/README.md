# Set 12

| S.No. | Question                                                                                                                                                                           |
| ----- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| 1.    | [How do you check if a directory exists in Node.js?](#question-1-how-do-you-check-if-a-directory-exists-in-nodejs)                                                                 |
| 2.    | [How do you create nested directories in Node.js?](#question-2-how-do-you-create-nested-directories-in-nodejs)                                                                     |
| 3.    | [What is the difference between `fs.mkdir` and `fs.mkdirSync`?](#question-3-what-is-the-difference-between-fsmkdir-and-fsmkdirsync)                                                |
| 4.    | [How do you watch a file or directory for changes in Node.js?](#question-4-how-do-you-watch-a-file-or-directory-for-changes-in-nodejs)                                             |
| 5.    | [What is the difference between `require.main === module` and normal `require()`?](#question-5-what-is-the-difference-between-requiremain--module-and-normal-require)              |
| 6.    | [How do you handle uncaught exceptions differently in development vs production?](#question-6-how-do-you-handle-uncaught-exceptions-differently-in-development-vs-production)      |
| 7.    | [How do you log to the console with timestamps in Node.js?](#question-7-how-do-you-log-to-the-console-with-timestamps-in-nodejs)                                                   |
| 8.    | [How do you pause and resume streams in Node.js?](#question-8-how-do-you-pause-and-resume-streams-in-nodejs)                                                                       |
| 9.    | [How do you use template literals in Node.js?](#question-9-how-do-you-use-template-literals-in-nodejs)                                                                             |
| 10.   | [How do you convert a buffer to a string and vice versa?](#question-10-how-do-you-convert-a-buffer-to-a-string-and-vice-versa)                                                     |
| 11.   | [How do you handle text encoding in Node.js?](#question-11-how-do-you-handle-text-encoding-in-nodejs)                                                                              |
| 12.   | [How do you use the `path` module in Node.js?](#question-12-how-do-you-use-the-path-module-in-nodejs)                                                                              |
| 13.   | [How do you join paths across different OS platforms in Node.js?](#question-13-how-do-you-join-paths-across-different-os-platforms-in-nodejs)                                      |
| 14.   | [How do you normalize paths in Node.js?](#question-14-how-do-you-normalize-paths-in-nodejs)                                                                                        |
| 15.   | [How do you resolve absolute paths from relative paths?](#question-15-how-do-you-resolve-absolute-paths-from-relative-paths)                                                       |
| 16.   | [How do you implement rate limiting using `express-rate-limit` for specific routes?](#question-16-how-do-you-implement-rate-limiting-using-express-rate-limit-for-specific-routes) |
| 17.   | [How do you implement JWT token expiration and refresh in Node.js?](#question-17-how-do-you-implement-jwt-token-expiration-and-refresh-in-nodejs)                                  |
| 18.   | [How do you protect routes using JWT authentication in Express.js?](#question-18-how-do-you-protect-routes-using-jwt-authentication-in-expressjs)                                  |
| 19.   | [How do you implement file compression using streams in Node.js?](#question-19-how-do-you-implement-file-compression-using-streams-in-nodejs)                                      |
| 20.   | [How do you use `zlib` module for gzip compression?](#question-20-how-do-you-use-zlib-module-for-gzip-compression)                                                                 |

## Question 1. How do you check if a directory exists in Node.js?

# Short answer

Use `fs.promises.access()` or `fs.promises.stat()` to check whether a directory exists. If you specifically need to verify that the path is a directory (and not just any file), use `stat()` and call `.isDirectory()`. Avoid using deprecated APIs like `fs.exists()` and avoid "check-then-act" patterns when the next step is to create/read the directory because they introduce race conditions.

---

# Explanation

Node.js provides several ways to determine whether a directory exists.

### 1. `fs.promises.stat()` (Recommended)

This is the most reliable approach when you need to know:

- Does the path exist?
- Is it actually a directory?

```text
Path
 ├── exists? → No → ENOENT
 └── Yes
      ├── Directory → isDirectory() === true
      └── File → isDirectory() === false
```

Example flow:

- `await stat(path)`
- If it succeeds, call `stats.isDirectory()`
- If it throws `ENOENT`, the directory doesn't exist.

---

### 2. `fs.promises.access()`

`access()` checks whether a path is accessible.

Example:

```js
await fs.access(path);
```

However, it **doesn't tell you whether the path is a file or directory**. It's useful only when you care about existence or permissions.

---

### 3. Avoid `fs.exists()`

`fs.exists()` is deprecated because its callback API is inconsistent with Node's standard error-first callbacks.

`fs.existsSync()` still exists and is acceptable for:

- startup scripts
- CLI tools
- build scripts

Avoid it in performance-sensitive server request paths because synchronous filesystem calls block the event loop.

---

## Race condition consideration

A common mistake is:

```text
if directory exists
    create file
```

Another process could delete the directory between the check and the file creation.

Instead:

```text
Try the operation
Handle ENOENT if it fails
```

This follows the EAFP (Easier to Ask Forgiveness than Permission) pattern and is the recommended approach in Node.js.

---

# Example (JavaScript)

```javascript
import { stat } from "node:fs/promises";

async function directoryExists(dirPath) {
  try {
    const stats = await stat(dirPath);
    return stats.isDirectory();
  } catch (err) {
    if (err.code === "ENOENT") {
      return false;
    }
    throw err; // Unexpected filesystem error
  }
}

(async () => {
  const exists = await directoryExists("./logs");
  console.log(exists ? "Directory exists" : "Directory does not exist");
})();
```

**Requires:** Node.js 14+ (works best on modern Node.js 18+ with ESM).

---

# Testing

### Unit Testing

Mock `node:fs/promises` using Jest or the built-in Node test runner to simulate:

- existing directory
- existing file
- missing directory (`ENOENT`)
- permission errors (`EACCES`)

Example (Node.js built-in test runner):

```bash
node --test
```

Integration test:

- Create a temporary directory using `fs.mkdtemp()`.
- Verify `directoryExists()` returns `true`.
- Remove it and verify it returns `false`.

---

# Ops & Monitoring

- Log unexpected filesystem errors (`EACCES`, `EPERM`, `EMFILE`) with structured logging.
- Track filesystem latency if directory checks are frequent.
- Instrument filesystem operations with OpenTelemetry spans when debugging I/O bottlenecks.
- Avoid repeated directory checks on every request; cache immutable paths when appropriate.
- Handle transient filesystem failures gracefully instead of crashing the process.

---

# Deployment & Scaling

- Prefer asynchronous filesystem APIs to avoid blocking the event loop.
- In containers, verify mounted volumes exist before startup or create them with `mkdir({ recursive: true })`.
- For serverless environments, remember that writable storage is typically limited (often `/tmp`).
- Avoid excessive existence checks on shared network filesystems, as latency can become significant.
- Use current LTS versions of Node.js (20.x or newer) for the latest performance improvements and API stability.

---

# Pitfalls

- **Don't use `fs.exists()`**—it is deprecated.
- **Don't use synchronous APIs (`existsSync`, `statSync`)** in request handlers because they block the event loop.
- **Don't rely on "check-then-act" logic**; perform the filesystem operation directly and handle `ENOENT` or other expected errors.

## Question 2. How do you create nested directories in Node.js?

## Question 3. What is the difference between `fs.mkdir` and `fs.mkdirSync`?

## Question 4. How do you watch a file or directory for changes in Node.js?

## Question 5. What is the difference between `require.main === module` and normal `require()`?

## Question 6. How do you handle uncaught exceptions differently in development vs production?

## Question 7. How do you log to the console with timestamps in Node.js?

## Question 8. How do you pause and resume streams in Node.js?

## Question 9. How do you use template literals in Node.js?

## Question 10. How do you convert a buffer to a string and vice versa?

## Question 11. How do you handle text encoding in Node.js?

## Question 12. How do you use the `path` module in Node.js?

## Question 13. How do you join paths across different OS platforms in Node.js?

## Question 14. How do you normalize paths in Node.js?

## Question 15. How do you resolve absolute paths from relative paths?

## Question 16. How do you implement rate limiting using `express-rate-limit` for specific routes?

## Question 17. How do you implement JWT token expiration and refresh in Node.js?

## Question 18. How do you protect routes using JWT authentication in Express.js?

## Question 19. How do you implement file compression using streams in Node.js?

## Question 20. How do you use `zlib` module for gzip compression?
