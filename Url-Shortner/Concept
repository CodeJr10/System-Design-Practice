<div align="center">

# 🔗 URL Shortener — System Design

A scalable URL shortening service similar to Bitly or TinyURL.

![System Design](https://img.shields.io/badge/System-Design-4f6ef7?style=flat-square)
![Database](https://img.shields.io/badge/Database-PostgreSQL-336791?style=flat-square&logo=postgresql&logoColor=white)
![Cache](https://img.shields.io/badge/Cache-Redis-DC382D?style=flat-square&logo=redis&logoColor=white)
![Queue](https://img.shields.io/badge/Queue-Kafka-231F20?style=flat-square&logo=apachekafka&logoColor=white)

</div>

---

## 📌 Overview

A URL shortener converts long URLs into short, shareable links.

```
Original  →  https://www.example.com/blog/system-design/url-shortener-guide
Short     →  https://short.ly/aZ91Kx
```

When a user opens the short URL, the service redirects them to the original.

---

## ✅ Requirements

<details>
<summary><b>Functional Requirements</b></summary>

- Create short URLs
- Redirect to original URLs
- Support custom aliases
- Optional expiration time
- Analytics tracking

</details>

<details>
<summary><b>Non-Functional Requirements</b></summary>

- High availability
- Low latency
- Scalability
- Fault tolerance
- Durability & Security

</details>

---

## 🏗 High Level Architecture

```
Client
  │
  ▼
Load Balancer
  │
  ├──────────────────────┐
  ▼                      ▼
URL Shortener       Redirect Service
Service                   │
  │                       │
  └──────────┬────────────┘
             ▼
          Database
             │
             ▼
         Redis Cache
```

---

## 🧩 Core Components

<details>
<summary><b>1. URL Shortener Service</b></summary>

Responsible for accepting long URLs, generating short codes, and storing mappings.

```http
POST /api/v1/shorten
```

**Request**
```json
{ "url": "https://example.com/very-long-url" }
```

**Response**
```json
{ "shortUrl": "https://short.ly/abc123" }
```

</details>

<details>
<summary><b>2. Redirect Service</b></summary>

Responsible for fetching the original URL and redirecting users.

```http
GET /abc123
→ 301 Redirect to Original URL
```

</details>

---

## 🗄 Database Design

```sql
CREATE TABLE url_mapping (
    id           BIGINT PRIMARY KEY,
    short_code   VARCHAR(10) UNIQUE,
    original_url TEXT,
    created_at   TIMESTAMP,
    expiry_at    TIMESTAMP,
    click_count  BIGINT
);
```

---

## 🔗 URL Generation

<details>
<summary><b>1. Hashing</b></summary>

```
MD5(long_url) → xYz123
```

| | |
|---|---|
| ✅ Simple & deterministic | ❌ Collision handling required |

</details>

<details>
<summary><b>2. Base62 Encoding (Recommended)</b></summary>

Character set: `[a-zA-Z0-9]`

```
shortCode = encodeBase62(ID)
125 → "cb"
```

- URL friendly — no special characters
- Compact representation
- Efficient encoding

</details>

---

## 🆔 ID Generation

| Strategy | Pros | Cons |
|---|---|---|
| Auto Increment | Simple | Predictable, insecure |
| Snowflake IDs | Distributed, unique | More complex |
| Random + Base62 ✅ | Secure, scalable | Collision handling |

---

## ⚡ Caching Strategy

```
Request Short URL
      │
      ▼
Check Redis Cache
      │
   ┌──┴──┐
  Hit   Miss
   │     │
Return  Query DB
  URL     │
         Store in Cache
          │
        Return URL
```

---

## 📈 Scaling

<details>
<summary><b>Horizontal Scaling</b></summary>

Add more API and redirect servers behind a load balancer.

</details>

<details>
<summary><b>Database Scaling</b></summary>

- **Read Replicas** — for heavy reads and analytics queries
- **Sharding** — shard by user ID or short code range

</details>

<details>
<summary><b>Optimizations</b></summary>

- CDN for lower latency globally
- Hot URL caching — popular URLs stay longer in cache
- Async analytics — never block redirects

</details>

---

## 📊 Analytics Pipeline

```
Redirect Service
      │
      ▼
Message Queue (Kafka)
      │
      ▼
Analytics Workers
      │
      ▼
Analytics Database
```

Tracks: click count · browser · device · geolocation · referrer

---

## 🔒 Security

| Threat | Solution |
|---|---|
| Spam & abuse | Rate limiting (100 req/min/user) |
| Malware links | URL blacklist scanning |
| Phishing | CAPTCHA + URL validation |

Rate limiting uses **Token Bucket** or **Leaky Bucket** with counters in Redis.

---

## 🌐 Redirect Types

| Type | Use Case | Pros | Cons |
|---|---|---|---|
| `301` Permanent | SEO, stable URLs | Browser caching, SEO friendly | Hard to change later |
| `302` Temporary | Analytics, A/B | Flexible | Less caching efficiency |

---

## 📡 API Design

| Method | Endpoint | Description |
|---|---|---|
| `POST` | `/shorten` | Create a short URL |
| `GET` | `/{shortCode}` | Redirect to original URL |
| `GET` | `/analytics/{shortCode}` | Get click analytics |

---

## ⚖ Trade-Offs

| Choice | Advantage | Drawback |
|---|---|---|
| SQL Database | Strong consistency | Harder to scale |
| NoSQL Database | High scalability | Eventual consistency |
| Sequential IDs | Simple | Predictable & insecure |
| Random IDs | Secure | Collision handling needed |

---

## 🛠 Tech Stack

| Layer | Technology |
|---|---|
| Backend | Node.js / Go / Java |
| Database | PostgreSQL |
| Cache | Redis |
| Queue | Kafka |
| Load Balancer | NGINX |
| Monitoring | Prometheus + Grafana |

---

## 📚 Interview Discussion Points

- How to avoid hash collisions?
- How to scale redirect traffic?
- SQL vs NoSQL — which to choose?
- Cache invalidation strategy?
- How to handle billions of URLs?
- Hot partition handling?
- Analytics scaling without blocking redirects?

---

## ✅ Conclusion

A URL shortener is a classic distributed systems problem. Key concepts:

- **Base62 encoding** — compact, URL-safe short codes
- **Distributed ID generation** — secure and non-predictable
- **Redis caching** — low-latency redirects at scale
- **Async analytics** — never block the redirect path
- **Horizontal scaling** — handle billions of URLs reliably
