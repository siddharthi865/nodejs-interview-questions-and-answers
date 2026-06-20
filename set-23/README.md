# Set 23

| S.No. | Question                                                                                                                                                                 |
| ----- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| 1.    | [How do you implement token rotation for refresh tokens?](#question-1-how-do-you-implement-token-rotation-for-refresh-tokens)                                            |
| 2.    | [How do you implement OAuth2 token introspection in Node.js?](#question-2-how-do-you-implement-oauth2-token-introspection-in-nodejs)                                     |
| 3.    | [How do you implement real-time collaboration using Socket.IO rooms?](#question-3-how-do-you-implement-real-time-collaboration-using-socketio-rooms)                     |
| 4.    | [How do you implement WebSocket authentication using JWT?](#question-4-how-do-you-implement-websocket-authentication-using-jwt)                                          |
| 5.    | [How do you implement heartbeat/ping-pong mechanism for WebSockets?](#question-5-how-do-you-implement-heartbeatping-pong-mechanism-for-websockets)                       |
| 6.    | [How do you implement dynamic middleware registration in Express.js?](#question-6-how-do-you-implement-dynamic-middleware-registration-in-expressjs)                     |
| 7.    | [How do you implement multi-file uploads with progress tracking?](#question-7-how-do-you-implement-multi-file-uploads-with-progress-tracking)                            |
| 8.    | [How do you implement file streaming to cloud storage like AWS S3 or GCP?](#question-8-how-do-you-implement-file-streaming-to-cloud-storage-like-aws-s3-or-gcp)          |
| 9.    | [How do you implement database connection retries with exponential backoff?](#question-9-how-do-you-implement-database-connection-retries-with-exponential-backoff)      |
| 10.   | [How do you implement automatic reconnection for Redis or MongoDB clients?](#question-10-how-do-you-implement-automatic-reconnection-for-redis-or-mongodb-clients)       |
| 11.   | [How do you implement caching database queries with TTL?](#question-11-how-do-you-implement-caching-database-queries-with-ttl)                                           |
| 12.   | [How do you implement schema migration scripts using `knex` or `sequelize`?](#question-12-how-do-you-implement-schema-migration-scripts-using-knex-or-sequelize)         |
| 13.   | [How do you implement API documentation generation with Swagger?](#question-13-how-do-you-implement-api-documentation-generation-with-swagger)                           |
| 14.   | [How do you implement API throttling per user role?](#question-14-how-do-you-implement-api-throttling-per-user-role)                                                     |
| 15.   | [How do you implement rate limiting with distributed Redis store?](#question-15-how-do-you-implement-rate-limiting-with-distributed-redis-store)                         |
| 16.   | [How do you implement clustered Express apps with sticky sessions?](#question-16-how-do-you-implement-clustered-express-apps-with-sticky-sessions)                       |
| 17.   | [How do you implement automatic request retries with circuit breaker pattern?](#question-17-how-do-you-implement-automatic-request-retries-with-circuit-breaker-pattern) |
| 18.   | [How do you implement background job scheduling using `bull`?](#question-18-how-do-you-implement-background-job-scheduling-using-bull)                                   |
| 19.   | [How do you implement prioritized job queues?](#question-19-how-do-you-implement-prioritized-job-queues)                                                                 |
| 20.   | [How do you implement task dependency chains in background jobs?](#question-20-how-do-you-implement-task-dependency-chains-in-background-jobs)                           |

## Question 1. How do you implement token rotation for refresh tokens?

# Short answer

Refresh token rotation means **issuing a new refresh token every time the client uses one** and immediately invalidating the old token. If an already-used refresh token is presented again, treat it as a **token replay attack**, revoke the entire token family (or all active sessions for that device), and require the user to authenticate again.

---

# Explanation

Refresh token rotation is one of the strongest defenses against stolen refresh tokens. Instead of allowing a refresh token to be reused until it expires, every successful refresh request:

1. Client sends the current refresh token.
2. Server verifies its signature (if JWT) or looks it up (if opaque token).
3. Server checks:
   - Token exists
   - Not expired
   - Not revoked
   - Not previously used

4. Server marks the current refresh token as **used/revoked**.
5. Server generates:
   - New access token
   - New refresh token

6. Server stores the new refresh token and returns it to the client.

Example flow:

```text
Login
  ↓
Refresh Token R1
  ↓
Refresh Request(R1)
  ↓
Invalidate R1
Issue R2
  ↓
Client stores R2
```

If someone later attempts:

```text
Refresh Request(R1)
```

the server knows:

- R1 has already been used
- Someone copied the token
- Possible credential theft

The safest action is:

- Revoke R2 and every descendant token
- Force re-authentication

---

## Token family

Each login session belongs to a **token family**.

```text
Family A

R1
 │
 └──► R2
       │
       └──► R3
             │
             └──► R4
```

If R2 is reused after R3 was issued:

```text
R2 reused ❌
```

then revoke:

```text
R3
R4
```

or even the entire family.

This prevents attackers from racing the legitimate user.

---

## Database schema

A typical table:

```text
refresh_tokens

id
user_id
family_id
token_hash
parent_token_id
expires_at
used_at
revoked_at
created_at
device_id
ip
user_agent
```

Never store refresh tokens in plaintext.

Store:

```text
SHA-256(token)
```

similar to password storage (or HMAC if preferred).

---

## Rotation algorithm

```text
Receive refresh token

↓

Hash token

↓

Lookup DB

↓

Found?

No → Reject

↓

Expired?

Yes → Reject

↓

Revoked?

Yes → Reject

↓

Used already?

Yes
↓
Replay detected
↓
Revoke entire family
↓
Require login

↓

Mark current token used

↓

Generate new refresh token

↓

Store new row

↓

Return new access + refresh tokens
```

---

## JWT vs opaque refresh tokens

### JWT refresh tokens

Pros

- Self-contained
- Signature verification is fast

Cons

- Cannot revoke without server storage
- Rotation still requires a database
- Larger payloads

In practice, JWT refresh tokens still require server-side state for secure rotation.

---

### Opaque refresh tokens

```
Refresh Token

e2f98da12a89...
```

Database:

```text
hash(token)
↓

User
Expiry
Revoked?
Used?
Family
```

Pros

- Easy revocation
- Simple rotation
- Smaller payload
- Recommended for most production systems

---

## Security improvements

### Hash refresh tokens

Never store:

```text
eyJhb...
```

Store:

```text
SHA256(token)
```

---

### Use HttpOnly cookies

Store refresh tokens in:

```
HttpOnly
Secure
SameSite=Lax or Strict
```

Avoid storing long-lived refresh tokens in `localStorage`, where they are more exposed to XSS.

---

### Short access tokens

Example:

```
Access token:
15 minutes

Refresh token:
30 days
```

Only the refresh token needs to persist across sessions.

---

### Device tracking

Store:

- device ID
- browser
- IP address
- last used time

Users can then revoke individual sessions.

---

### Absolute expiration

Even with rotation:

```
Maximum session:
30 days
```

After that:

```
Login again
```

This limits the impact of long-running sessions.

---

# Example

**JavaScript (Node.js + Express + `crypto`)**

```javascript
import crypto from "node:crypto";

// Simulated token store
const refreshStore = new Map();

function hash(token) {
  return crypto.createHash("sha256").update(token).digest("hex");
}

function generateToken() {
  return crypto.randomBytes(32).toString("hex");
}

function issueRefreshToken(userId, familyId = crypto.randomUUID()) {
  const token = generateToken();
  const tokenHash = hash(token);

  refreshStore.set(tokenHash, {
    userId,
    familyId,
    used: false,
    revoked: false,
  });

  return token;
}

function rotateRefreshToken(oldToken) {
  const oldHash = hash(oldToken);
  const record = refreshStore.get(oldHash);

  if (!record || record.revoked) {
    throw new Error("Invalid refresh token");
  }

  if (record.used) {
    // Replay detected: revoke the token family
    for (const token of refreshStore.values()) {
      if (token.familyId === record.familyId) {
        token.revoked = true;
      }
    }
    throw new Error("Refresh token replay detected");
  }

  record.used = true;

  const newToken = issueRefreshToken(record.userId, record.familyId);

  return {
    accessToken: crypto.randomBytes(16).toString("hex"),
    refreshToken: newToken,
  };
}

// Example usage
const refreshToken = issueRefreshToken("user-123");
const tokens = rotateRefreshToken(refreshToken);
console.log(tokens);
```

In production, persist token metadata in a database, execute the "mark used + insert new token" operations in a transaction, and use row-level locking or optimistic concurrency control to avoid race conditions when multiple refresh requests occur simultaneously.

---

# Testing

**Unit testing**

Verify:

- Successful token rotation issues a new refresh token.
- Old refresh token is marked as used.
- Reusing an old refresh token triggers replay detection.
- Revoking a token family prevents further refreshes.

Using the built-in test runner:

```bash
node --test
```

Example assertion:

```javascript
import test from "node:test";
import assert from "node:assert/strict";

// Assert that rotating the same token twice throws an error.
```

**Integration testing**

- Simulate concurrent refresh requests using the same token and verify only one succeeds.
- Test session revocation across multiple devices.
- Validate cookie flags (`HttpOnly`, `Secure`, `SameSite`) in HTTP responses.

---

# Ops & Monitoring

- **Logging:** Log refresh attempts, replay detection, revocations, and session creation without logging raw tokens.
- **Metrics:** Track refresh success rate, replay detections, refresh latency, and active sessions.
- **Tracing:** Instrument authentication flows with OpenTelemetry while excluding sensitive token values.
- **Error handling:** Return generic authentication errors (e.g., HTTP 401) without revealing whether a token existed or was replayed.
- **Process management:** Keep refresh token state in a shared datastore (e.g., PostgreSQL or Redis), not process memory, to support multiple Node.js instances managed by PM2, systemd, or containers.

---

# Deployment & Scaling

- Use a shared database or Redis so all application instances see the same token state.
- Wrap rotation in a database transaction or use atomic Redis operations to prevent concurrent reuse.
- Hash refresh tokens before storage to reduce the impact of database compromise.
- Use connection pooling for the backing datastore to avoid authentication bottlenecks.
- In serverless environments, keep refresh validation state external because execution environments are ephemeral.
- Prefer modern LTS versions of Node.js (Node.js 20+ or newer active LTS releases) for current security updates and runtime improvements.

---

# Pitfalls

- **Allowing refresh token reuse**, which defeats replay detection and increases the impact of token theft.
- **Storing refresh tokens in plaintext** instead of hashed values.
- **Performing rotation non-atomically**, allowing race conditions where two refresh requests both succeed.

## Question 2. How do you implement OAuth2 token introspection in Node.js?

## Question 3. How do you implement real-time collaboration using Socket.IO rooms?

## Question 4. How do you implement WebSocket authentication using JWT?

## Question 5. How do you implement heartbeat/ping-pong mechanism for WebSockets?

## Question 6. How do you implement dynamic middleware registration in Express.js?

## Question 7. How do you implement multi-file uploads with progress tracking?

## Question 8. How do you implement file streaming to cloud storage like AWS S3 or GCP?

## Question 9. How do you implement database connection retries with exponential backoff?

## Question 10. How do you implement automatic reconnection for Redis or MongoDB clients?

## Question 11. How do you implement caching database queries with TTL?

## Question 12. How do you implement schema migration scripts using `knex` or `sequelize`?

## Question 13. How do you implement API documentation generation with Swagger?

## Question 14. How do you implement API throttling per user role?

## Question 15. How do you implement rate limiting with distributed Redis store?

## Question 16. How do you implement clustered Express apps with sticky sessions?

## Question 17. How do you implement automatic request retries with circuit breaker pattern?

## Question 18. How do you implement background job scheduling using `bull`?

## Question 19. How do you implement prioritized job queues?

## Question 20. How do you implement task dependency chains in background jobs?
