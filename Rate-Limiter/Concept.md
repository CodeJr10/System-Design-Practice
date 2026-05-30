
# 🚦 Rate Limiter

A flexible, production-ready rate limiting library that controls the frequency of operations — protecting your APIs, services, and resources from overload.

---

## Table of Contents

- [Overview](#overview)
- [How It Works](#how-it-works)
- [Algorithms](#algorithms)
- [Installation](#installation)
- [Quick Start](#quick-start)
- [Configuration](#configuration)
- [Usage Examples](#usage-examples)
- [API Reference](#api-reference)
- [Error Handling](#error-handling)
- [Contributing](#contributing)
- [License](#license)

---

## Overview

Rate limiting is a technique used to control how many requests a client can make to a server within a defined time window. This library provides multiple strategies to enforce those limits, helping you:

- Prevent API abuse and denial-of-service attacks
- Ensure fair usage across multiple clients
- Protect downstream services from being overwhelmed
- Comply with third-party API quotas

---

## How It Works

At its core, a rate limiter tracks request counts or timestamps per client (identified by IP, API key, user ID, etc.) and decides whether to **allow** or **reject** each incoming request based on the configured policy.

```
Client → Request → Rate Limiter → Allow ✅ → Service
                               ↘ Reject ❌ → 429 Too Many Requests
```

---

## Algorithms

### 1. Token Bucket
Each client has a "bucket" that fills with tokens at a fixed rate. Each request consumes one token. If the bucket is empty, the request is rejected.

- ✅ Allows short bursts
- ✅ Smooth average rate
- ✅ Simple to implement

### 2. Fixed Window Counter
Counts requests within a fixed time window (e.g., 100 requests per minute). The counter resets at the start of each window.

- ✅ Very fast and memory-efficient
- ⚠️ Vulnerable to burst at window boundaries

### 3. Sliding Window Log
Tracks a log of timestamps for each request. Only counts requests within the rolling window from *now*.

- ✅ Accurate and smooth
- ⚠️ Higher memory usage for high-traffic clients

### 4. Sliding Window Counter
Combines fixed windows with interpolation to approximate a sliding window without storing individual timestamps.

- ✅ Low memory overhead
- ✅ More accurate than fixed window
- ✅ Best balance of accuracy and performance

### 5. Leaky Bucket
Requests are added to a queue and processed at a fixed rate, like water leaking from a bucket.

- ✅ Strictly enforces a constant output rate
- ⚠️ Does not allow bursts

---

## Installation

```bash
# npm
npm install rate-limiter

# yarn
yarn add rate-limiter

# pip
pip install rate-limiter

# go
go get github.com/yourorg/rate-limiter
```

---

## Quick Start

```javascript
import { RateLimiter } from 'rate-limiter';

const limiter = new RateLimiter({
  algorithm: 'sliding-window',
  limit: 100,        // max requests
  windowMs: 60000,   // per 60 seconds
});

// In your request handler
app.use((req, res, next) => {
  const clientId = req.ip;

  if (limiter.allow(clientId)) {
    next();
  } else {
    res.status(429).json({ error: 'Too Many Requests' });
  }
});
```

---

## Configuration

| Option | Type | Default | Description |
|---|---|---|---|
| `algorithm` | `string` | `'token-bucket'` | Rate limiting algorithm to use |
| `limit` | `number` | `100` | Maximum number of allowed requests |
| `windowMs` | `number` | `60000` | Time window in milliseconds |
| `keyPrefix` | `string` | `'rl:'` | Prefix for storage keys |
| `store` | `Store` | In-memory | Storage backend (Redis, memory, etc.) |
| `onRejected` | `function` | `null` | Callback fired when a request is rejected |
| `skipFailedRequests` | `boolean` | `false` | Don't count failed requests against the limit |
| `headers` | `boolean` | `true` | Send `X-RateLimit-*` headers in responses |

---

## Usage Examples

### Express Middleware

```javascript
import express from 'express';
import { RateLimiter } from 'rate-limiter';

const app = express();

const apiLimiter = new RateLimiter({
  algorithm: 'token-bucket',
  limit: 50,
  windowMs: 15 * 60 * 1000, // 15 minutes
});

app.use('/api/', apiLimiter.middleware());
```

### With Redis Store (Distributed)

```javascript
import { RateLimiter, RedisStore } from 'rate-limiter';
import { createClient } from 'redis';

const redis = createClient({ url: 'redis://localhost:6379' });
await redis.connect();

const limiter = new RateLimiter({
  algorithm: 'sliding-window-counter',
  limit: 200,
  windowMs: 60000,
  store: new RedisStore(redis),
});
```

### Custom Key Function (per user, not per IP)

```javascript
const limiter = new RateLimiter({
  limit: 1000,
  windowMs: 3600000, // 1 hour
  keyGenerator: (req) => req.user?.id ?? req.ip,
});
```

### Different Limits per Route

```javascript
const strictLimiter = new RateLimiter({ limit: 5,   windowMs: 60000 });
const looseLimiter  = new RateLimiter({ limit: 500, windowMs: 60000 });

app.post('/auth/login',  strictLimiter.middleware(), loginHandler);
app.get('/api/data',     looseLimiter.middleware(),  dataHandler);
```

---

## API Reference

### `RateLimiter(options)`

Creates a new rate limiter instance.

### `limiter.allow(key: string): boolean`

Returns `true` if the request should be allowed, `false` if it should be rejected.

### `limiter.middleware()`

Returns an Express/Connect-compatible middleware function.

### `limiter.reset(key: string): void`

Manually resets the limit for a given client key.

### `limiter.getInfo(key: string): RateLimitInfo`

Returns current rate limit status for a key.

```typescript
interface RateLimitInfo {
  limit: number;        // Total allowed requests
  remaining: number;    // Remaining requests in window
  resetAt: Date;        // When the window resets
  retryAfter: number;   // Seconds until next allowed request
}
```

### Response Headers

When `headers: true`, the following headers are added to responses:

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 42
X-RateLimit-Reset: 1716900000
Retry-After: 30
```

---

## Error Handling

When a request is rate-limited, the library rejects it with a `RateLimitError`:

```javascript
try {
  limiter.checkOrThrow(clientId);
} catch (err) {
  if (err instanceof RateLimitError) {
    console.log(`Rate limited. Retry after ${err.retryAfter}s`);
  }
}
```

For HTTP APIs, return a `429 Too Many Requests` response with a `Retry-After` header to be standards-compliant (RFC 6585).

---

## Comparison of Algorithms

| Algorithm | Burst Handling | Memory Usage | Accuracy | Best For |
|---|---|---|---|---|
| Token Bucket | ✅ Yes | Low | High | APIs with bursty traffic |
| Fixed Window | ⚠️ Edge bursts | Very Low | Medium | Simple use cases |
| Sliding Window Log | ✅ Yes | High | Highest | Low-traffic, strict limits |
| Sliding Window Counter | ✅ Partial | Low | High | Most production APIs |
| Leaky Bucket | ❌ No | Low | High | Strict throughput control |

---

## Contributing

Contributions are welcome! Please open an issue before submitting a pull request for major changes.

1. Fork the repository
2. Create your feature branch: `git checkout -b feature/my-feature`
3. Commit your changes: `git commit -m 'Add my feature'`
4. Push to the branch: `git push origin feature/my-feature`
5. Open a Pull Request

Please make sure all tests pass: `npm test`

---

## License

MIT © [Your Name](https://github.com/yourname)
