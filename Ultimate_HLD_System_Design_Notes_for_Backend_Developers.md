# Ultimate High-Level Design (HLD) / System Design Notes — From Single Server to Millions of Users

> **Who is this for?** Backend developers who can build an API on a single server but freeze when asked: "How would you scale this to 10 million users?", "What happens when the database becomes the bottleneck?", or "Design Twitter's news feed." After reading this, system design interviews will feel like conversations, not interrogations.

---

## Table of Contents

**Part 1 — Foundations of Scale**

1. [What Is HLD and How Is It Different from LLD?](#1-what-is-hld-and-how-is-it-different-from-lld)
2. [Scaling 101 — Vertical vs Horizontal](#2-scaling-101--vertical-vs-horizontal)
3. [Load Balancing — Distributing the Traffic](#3-load-balancing--distributing-the-traffic)
4. [Caching — The Single Biggest Performance Win](#4-caching--the-single-biggest-performance-win)
5. [CDN — Bringing Content Closer to Users](#5-cdn--bringing-content-closer-to-users)
6. [Proxy & Reverse Proxy](#6-proxy--reverse-proxy)

**Part 2 — Data Layer**

7. [Database Fundamentals — SQL vs NoSQL](#7-database-fundamentals--sql-vs-nosql)
8. [Database Indexing — Why Your Queries Are Slow](#8-database-indexing--why-your-queries-are-slow)
9. [Database Replication — Copies for Safety and Speed](#9-database-replication--copies-for-safety-and-speed)
10. [Database Sharding — Splitting Data Across Machines](#10-database-sharding--splitting-data-across-machines)
11. [CAP Theorem & Consistency Models](#11-cap-theorem--consistency-models)
12. [Consistent Hashing — The Key to Distributed Systems](#12-consistent-hashing--the-key-to-distributed-systems)

**Part 3 — Communication & Processing**

13. [Message Queues & Async Processing](#13-message-queues--async-processing)
14. [Event-Driven Architecture](#14-event-driven-architecture)
15. [Microservices vs Monolith](#15-microservices-vs-monolith)
16. [API Gateway — The Front Door](#16-api-gateway--the-front-door)
17. [Service Discovery & Communication](#17-service-discovery--communication)

**Part 4 — Reliability & Performance**

18. [Rate Limiting at Scale](#18-rate-limiting-at-scale)
19. [Circuit Breaker — Failing Gracefully](#19-circuit-breaker--failing-gracefully)
20. [Monitoring, Logging & Observability](#20-monitoring-logging--observability)
21. [Data Redundancy, Backup & Disaster Recovery](#21-data-redundancy-backup--disaster-recovery)

**Part 5 — Real System Design Case Studies**

22. [How to Approach Any System Design Problem](#22-how-to-approach-any-system-design-problem)
23. [Design a URL Shortener (like bit.ly)](#23-design-a-url-shortener-like-bitly)
24. [Design a Chat System (like WhatsApp)](#24-design-a-chat-system-like-whatsapp)
25. [Design a News Feed (like Twitter/Instagram)](#25-design-a-news-feed-like-twitterinstagram)
26. [Design a Notification System](#26-design-a-notification-system)
27. [Design a Rate Limiter](#27-design-a-rate-limiter)
28. [Design a File Storage System (like Google Drive)](#28-design-a-file-storage-system-like-google-drive)
29. [Design an E-Commerce System](#29-design-an-e-commerce-system)

**Part 6 — Interview Questions**

30. [System Design Interview Questions (80+ Questions)](#30-system-design-interview-questions-80-questions)

---

# PART 1 — FOUNDATIONS OF SCALE

---

## 1. What Is HLD and How Is It Different from LLD?

### LLD vs HLD — The Building Analogy

**LLD (Low-Level Design)** is the interior of a building — how rooms are laid out, where the wiring goes, which materials to use. It's about classes, patterns, and code structure inside a single service.

**HLD (High-Level Design)** is the city plan — where buildings are located, how roads connect them, where the power plant is, how water flows. It's about servers, databases, caches, load balancers, message queues, and how they all connect.

| | LLD | HLD |
|-|-----|-----|
| **Scope** | Inside one service | Across the entire system |
| **Concerns** | Classes, patterns, SOLID | Servers, databases, caching, scaling |
| **Question** | "How should I structure this code?" | "How does this system handle 10M users?" |
| **Tools** | Design patterns, clean architecture | Load balancers, CDNs, message queues, sharding |
| **Interview** | "Design a parking lot system (classes)" | "Design Twitter (architecture)" |

### The Journey from One User to Millions

Every system starts the same way — one server doing everything:

```
Stage 1: One server
┌─────────────────────────┐
│   Your App + Database   │ ← Everything on one machine
│   (Node.js + MongoDB)   │
└─────────────────────────┘
         ↑
    100 users
```

As traffic grows, you hit bottlenecks. HLD is about knowing which bottleneck you're hitting and which tool solves it.

```
Stage 5: Millions of users
                         ┌─── CDN (static files, images)
                         │
Users → DNS → Load Balancer → App Server 1 ──→ Cache (Redis)
                           → App Server 2 ──→ Read Replica DB
                           → App Server 3 ──→ Primary DB
                           → App Server N      │
                                               ↓
                              Message Queue → Worker Servers
                                               │
                                          Search Engine
                                          (Elasticsearch)
```

---

## 2. Scaling 101 — Vertical vs Horizontal

### Vertical Scaling (Scale Up) — Bigger Machine

Buy a more powerful server — more CPU, more RAM, bigger disk.

**Analogy:** Your restaurant is full. Vertical scaling means knocking down walls and making the restaurant bigger — bigger kitchen, more tables in one building.

```
Before: 4 CPU, 8 GB RAM, 100 GB disk
After:  64 CPU, 512 GB RAM, 2 TB disk
```

**Pros:** Simple (no code changes), no distributed system complexity.
**Cons:** There's a ceiling (you can't buy a server with 10,000 CPUs). Single point of failure (if that one server dies, everything dies). Expensive at the top end.

### Horizontal Scaling (Scale Out) — More Machines

Add more servers and distribute the load.

**Analogy:** Instead of making one restaurant bigger, you open 10 branches across the city. Each branch handles some customers.

```
Before: 1 server
After:  10 servers behind a load balancer
```

**Pros:** No ceiling (keep adding servers). Redundancy (if one dies, others handle traffic). Often cheaper (many commodity servers vs one supercomputer).
**Cons:** Requires code changes (stateless design), introduces distributed system complexity (data consistency, network failures).

### What to Scale First?

```
Step 1: Vertical scale (buy better hardware) — cheapest and fastest fix
Step 2: Separate DB from app server — isolate the bottleneck
Step 3: Add a cache (Redis) — reduce DB load dramatically
Step 4: Horizontal scale app servers — add more behind a load balancer
Step 5: Database read replicas — handle read-heavy traffic
Step 6: Database sharding — when even replicas aren't enough
Step 7: Message queues — handle async/heavy processing
Step 8: Microservices — when the team/codebase is too big for one service
```

**Rule of thumb:** Don't jump to step 8 when step 3 would solve your problem. Most apps never need sharding. Many never need microservices. Scale in response to real bottlenecks, not hypothetical ones.

---

## 3. Load Balancing — Distributing the Traffic

### The Problem

You have 5 app servers. How do you distribute incoming requests evenly across them? And what happens when one server crashes?

### The Analogy

A bank with 5 counters. Without a load balancer, all customers crowd at counter 1. With a load balancer (the queue manager), customers are directed to whichever counter is available.

### How It Works

```
                    ┌─── App Server 1  (healthy ✓)
                    │
Client → Load   ───┼─── App Server 2  (healthy ✓)
         Balancer   │
                    ├─── App Server 3  (DOWN ✗) ← LB stops sending here
                    │
                    └─── App Server 4  (healthy ✓)
```

The load balancer:
1. Receives all incoming requests
2. Chooses a healthy server to forward to
3. Returns the server's response to the client
4. Health-checks servers periodically — removes unhealthy ones

### Load Balancing Algorithms

| Algorithm | How It Works | Best For |
|-----------|-------------|----------|
| **Round Robin** | Server 1, 2, 3, 4, 1, 2, 3, 4... | All servers are identical |
| **Weighted Round Robin** | Server 1 (weight 3), Server 2 (weight 1) → S1 gets 3x traffic | Servers with different capacities |
| **Least Connections** | Send to the server with fewest active connections | Requests with varying processing time |
| **IP Hash** | Hash the client's IP to always send them to the same server | When you need session stickiness |
| **Random** | Pick a random server | Simple, works well at scale |

### Layer 4 vs Layer 7 Load Balancing

| | Layer 4 (Transport) | Layer 7 (Application) |
|-|---------------------|----------------------|
| **Operates on** | TCP/UDP packets | HTTP requests |
| **Knows about** | IP addresses, ports | URLs, headers, cookies, body |
| **Speed** | Faster (less processing) | Slower (reads HTTP content) |
| **Smart routing** | No (can't read URL) | Yes (route /api to API servers, /images to CDN) |
| **Example** | AWS NLB | AWS ALB, Nginx, HAProxy |

**Layer 7 is what you'll use most** — it can route `/api/users` to user servers and `/api/orders` to order servers.

### Real-World Load Balancers

| Tool | Type | Used By |
|------|------|---------|
| **Nginx** | Software LB (L7) | Nearly everyone |
| **HAProxy** | Software LB (L4/L7) | High-performance setups |
| **AWS ALB** | Managed LB (L7) | AWS users |
| **AWS NLB** | Managed LB (L4) | AWS, ultra-low latency |
| **Cloudflare** | CDN + LB | Global traffic distribution |

### Stateless Design — The Key to Horizontal Scaling

For load balancing to work, your servers must be **stateless** — any server can handle any request.

```javascript
// BAD — stateful (session stored in server memory)
app.post('/login', (req, res) => {
  req.session.user = { id: 42, name: 'Ravi' };  // Stored in THIS server's memory
  // If the next request goes to a different server, the session is lost!
});

// GOOD — stateless (session stored externally)
app.post('/login', (req, res) => {
  const token = jwt.sign({ id: 42, name: 'Ravi' }, SECRET);
  res.json({ token });
  // Token goes to the client. Any server can verify it.
});

// Alternative: Store sessions in Redis (shared across all servers)
const session = require('express-session');
const RedisStore = require('connect-redis').default;
app.use(session({
  store: new RedisStore({ client: redisClient }),  // Shared session store
  secret: 'your-secret',
}));
```

---

## 4. Caching — The Single Biggest Performance Win

### The Problem

Your database is the bottleneck. Every request queries the database. With 10,000 requests per second, your database is drowning. Most of these requests fetch the same data — the same product page, the same user profile, the same config.

### The Analogy

You're a student. Every time you need a formula, you walk to the library, find the book, find the page, read it, walk back, and use it. Caching is writing the formula on a sticky note on your desk. Next time, you glance at the sticky note (nanoseconds) instead of walking to the library (minutes).

### Cache Hit vs Cache Miss

```
Request → Check Cache → Found? (HIT) → Return cached data      ← Fast (1-5ms)
                      → Not found? (MISS) → Query database      ← Slow (50-500ms)
                                          → Store in cache
                                          → Return data
```

### Where to Cache

```
Client (Browser) ← Browser cache, localStorage
       ↓
CDN ← Static files, images, API responses
       ↓
Load Balancer
       ↓
Application ← In-memory cache (Redis, Memcached)
       ↓
Database ← Query cache, buffer pool
```

### Application Caching with Redis

Redis is the most popular caching solution. It's an in-memory data store — blazing fast because data never hits disk (for caching).

```javascript
const redis = require('redis');
const client = redis.createClient({ url: 'redis://localhost:6379' });

// Cache-Aside Pattern (most common)
async function getUser(userId) {
  // Step 1: Check cache
  const cached = await client.get(`user:${userId}`);
  if (cached) {
    return JSON.parse(cached);  // CACHE HIT — return immediately
  }

  // Step 2: Cache miss — query database
  const user = await db.query('SELECT * FROM users WHERE id = ?', userId);

  // Step 3: Store in cache for next time (expire after 1 hour)
  await client.setEx(`user:${userId}`, 3600, JSON.stringify(user));

  return user;
}
```

### Caching Strategies

#### 1. Cache-Aside (Lazy Loading)
Application checks cache first. On miss, queries DB and updates cache. Most common pattern.

```
Read:  App → Cache → (miss) → DB → write to Cache → return
Write: App → DB → invalidate/update Cache
```

**Pros:** Only caches data that's actually requested. Cache failure doesn't break the app.
**Cons:** First request is always slow (cold cache). Data can become stale.

#### 2. Write-Through
Every write goes to both cache AND database simultaneously.

```
Write: App → Cache + DB (both updated together)
Read:  App → Cache → (always hit, in theory)
```

**Pros:** Cache is always up-to-date. Reads are always fast.
**Cons:** Write latency is higher (writing twice). Caches data that might never be read.

#### 3. Write-Behind (Write-Back)
Write to cache immediately, then asynchronously write to DB later.

```
Write: App → Cache → (async, later) → DB
Read:  App → Cache → (always hit)
```

**Pros:** Extremely fast writes. Great for write-heavy workloads.
**Cons:** Risk of data loss if cache crashes before writing to DB. Complex.

#### 4. Read-Through
Cache itself is responsible for loading data from DB on miss. The application only talks to the cache.

```
Read: App → Cache → (miss) → Cache loads from DB → returns to App
```

### Cache Invalidation — The Hardest Problem in CS

> "There are only two hard things in Computer Science: cache invalidation and naming things." — Phil Karlton

When the underlying data changes, the cache must be updated or removed. Get this wrong and users see stale data.

| Strategy | How | When |
|----------|-----|------|
| **TTL (Time-To-Live)** | Cache expires after X seconds | Data that's okay to be slightly stale (product listings) |
| **Event-Based** | Invalidate cache when data changes | Data that must be fresh (user profile after update) |
| **Write-Through** | Update cache on every write | When reads far outnumber writes |

```javascript
// Event-based invalidation
async function updateUser(userId, data) {
  await db.query('UPDATE users SET ... WHERE id = ?', userId);

  // Invalidate the cache — next read will fetch fresh data from DB
  await client.del(`user:${userId}`);
}
```

### What to Cache vs What Not to Cache

| Cache This | Don't Cache This |
|-----------|-----------------|
| Database query results | Rapidly changing data (stock prices in a trading app) |
| API responses from external services | User-specific sensitive data (without proper isolation) |
| Computed/aggregated data (leaderboards) | Data that MUST be real-time consistent |
| Session data | Large blobs that change frequently |
| Configuration and feature flags | Data with unpredictable access patterns |

### Cache Stampede — When the Cache Expires, Everyone Storms the DB

A popular cache key expires. 10,000 requests simultaneously hit the database for the same data.

**Fix 1: Locking** — Only one request queries the DB. Others wait for the cache to be repopulated.

```javascript
async function getUserWithLock(userId) {
  const cached = await client.get(`user:${userId}`);
  if (cached) return JSON.parse(cached);

  // Try to acquire a lock
  const lockAcquired = await client.set(`lock:user:${userId}`, '1', 'NX', 'EX', 10);

  if (lockAcquired) {
    // I got the lock — I'll fetch from DB and repopulate cache
    const user = await db.query('SELECT * FROM users WHERE id = ?', userId);
    await client.setEx(`user:${userId}`, 3600, JSON.stringify(user));
    await client.del(`lock:user:${userId}`);
    return user;
  } else {
    // Someone else is fetching — wait and retry
    await sleep(100);
    return getUserWithLock(userId);
  }
}
```

**Fix 2: Stale-while-revalidate** — Serve slightly stale data while refreshing in the background.

---

## 5. CDN — Bringing Content Closer to Users

### The Analogy

Amazon doesn't ship everything from one warehouse. They have warehouses in every major city. When you order something, it ships from the warehouse closest to you. A CDN does the same for digital content — it stores copies of your files on servers around the world.

### How It Works

```
Without CDN:
User in Mumbai → Request → Server in US → Response (200ms latency)

With CDN:
User in Mumbai → Request → CDN Edge in Mumbai → Response (20ms latency)
                              ↑
                     (CDN fetches from US server once, then serves locally)
```

### What Goes on a CDN

| CDN This | Don't CDN This |
|----------|---------------|
| Images, videos, audio | Dynamic API responses (usually) |
| CSS, JavaScript bundles | Personalised content |
| Static HTML pages | Real-time data |
| Fonts | Authenticated API calls |
| API responses that are public and cacheable | Frequently changing data |

### Popular CDNs

| CDN | Notes |
|-----|-------|
| **CloudFront** | AWS's CDN, integrates with S3 |
| **Cloudflare** | CDN + security + DNS + DDoS protection |
| **Akamai** | Enterprise, massive network |
| **Fastly** | Edge computing, real-time purging |

---

## 6. Proxy & Reverse Proxy

### Forward Proxy — Acts on Behalf of the Client

```
Client → Forward Proxy → Internet → Server
```

The server doesn't know who the real client is. Used for: anonymity, bypassing restrictions, content filtering in offices/schools.

### Reverse Proxy — Acts on Behalf of the Server

```
Client → Reverse Proxy → Server 1
                       → Server 2
                       → Server 3
```

The client doesn't know which server handled the request. Used for: load balancing, SSL termination, caching, compression, security.

**Analogy:** A forward proxy is like a VPN — hides your identity from the server. A reverse proxy is like a receptionist — hides which employee (server) actually handled your request.

**Nginx is the most popular reverse proxy.** It sits in front of your app servers and handles:
- Load balancing (distributing requests)
- SSL termination (handles HTTPS so your app doesn't have to)
- Static file serving (serves images/CSS directly, faster than your app)
- Compression (gzip responses)
- Rate limiting
- Request buffering

---

# PART 2 — DATA LAYER

---

## 7. Database Fundamentals — SQL vs NoSQL

### SQL (Relational Databases)

Data is stored in **tables** with fixed schemas, related through foreign keys.

```sql
-- Structured, normalised, relationships
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100) NOT NULL,
    email VARCHAR(255) UNIQUE NOT NULL
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    total DECIMAL(10,2),
    status VARCHAR(20)
);
```

**Examples:** PostgreSQL, MySQL, Oracle, SQL Server.

### NoSQL (Non-Relational Databases)

| Type | How It Stores Data | Example | Best For |
|------|-------------------|---------|----------|
| **Document** | JSON-like documents | MongoDB, CouchDB | Flexible schemas, rapid development |
| **Key-Value** | Simple key → value | Redis, DynamoDB | Caching, sessions, simple lookups |
| **Column-Family** | Columns grouped in families | Cassandra, HBase | Time-series, analytics, write-heavy |
| **Graph** | Nodes and relationships | Neo4j, Amazon Neptune | Social networks, recommendations |

```javascript
// MongoDB document — flexible, nested, no fixed schema
{
  "_id": "user_42",
  "name": "Ravi Kumar",
  "email": "ravi@example.com",
  "addresses": [
    { "type": "home", "city": "Delhi", "pincode": "110001" },
    { "type": "work", "city": "Gurugram", "pincode": "122001" }
  ],
  "preferences": {
    "language": "en",
    "notifications": true
  }
}
```

### When to Use Which

| Use SQL When | Use NoSQL When |
|-------------|---------------|
| Data has clear relationships (users → orders → items) | Schema changes frequently |
| You need ACID transactions | You need horizontal scaling |
| Data integrity is critical (banking, inventory) | Data is hierarchical or document-like |
| You need complex joins and aggregations | Read/write speed is more important than relationships |
| Your schema is well-defined and stable | You're dealing with massive scale (millions of writes/sec) |

**The Honest Answer for Interviews:** Most applications should start with PostgreSQL. It handles 90% of use cases, supports JSON for flexibility, has excellent performance, and provides strong consistency. Move to NoSQL when you have a specific reason — massive write throughput, highly variable schemas, or graph-like data.

### ACID vs BASE

| ACID (SQL) | BASE (NoSQL) |
|-----------|-------------|
| **A**tomicity — all or nothing | **B**asically **A**vailable — system is always available |
| **C**onsistency — data is always valid | **S**oft state — state may change over time |
| **I**solation — concurrent transactions don't interfere | **E**ventual consistency — will be consistent eventually |
| **D**urability — committed data survives crashes | |

**Analogy:** ACID is like a bank transfer — either both accounts update or neither does. BASE is like social media likes — if you see 999 likes instead of 1000 for a few seconds, nobody notices or cares.

---

## 8. Database Indexing — Why Your Queries Are Slow

### The Analogy

A textbook without an index. To find "photosynthesis," you read every page from page 1. That's what a database does without an index — it scans every row (**full table scan**). With an index, you go to the back of the book, find "photosynthesis → page 247," and jump directly there.

### How Indexes Work

Most databases use a **B-Tree** index — a sorted tree structure that enables binary search.

```
Without index: SELECT * FROM users WHERE email = 'ravi@example.com'
→ Scan all 10 million rows → 5 seconds

With index on email: Same query
→ B-Tree lookup → 3 rows checked → 5 milliseconds
```

### When to Create Indexes

```sql
-- Columns you search by frequently
CREATE INDEX idx_users_email ON users(email);

-- Columns you use in WHERE clauses
CREATE INDEX idx_orders_status ON orders(status);

-- Columns you sort by
CREATE INDEX idx_orders_created ON orders(created_at DESC);

-- Composite index (multiple columns — order matters!)
CREATE INDEX idx_orders_user_status ON orders(user_id, status);
-- This index works for:
--   WHERE user_id = 42
--   WHERE user_id = 42 AND status = 'pending'
-- But NOT for:
--   WHERE status = 'pending' (leftmost column must be included)
```

### Index Trade-offs

| Benefit | Cost |
|---------|------|
| Reads are dramatically faster | Writes are slower (index must be updated) |
| Sorting is faster | Extra disk space |
| Lookups are O(log n) instead of O(n) | Too many indexes slow down inserts/updates |

**Rule of thumb:** Index columns that appear in WHERE, JOIN, ORDER BY, and GROUP BY. Don't index columns that change constantly or have low cardinality (like a boolean `is_active` with only true/false).

---

## 9. Database Replication — Copies for Safety and Speed

### The Analogy

You have one notebook with all your exam notes. If you lose it, you're done. Solution: make copies. Your friend has a copy, your study group has a copy. Even if your notebook is destroyed, the knowledge survives. Database replication is the same — maintain copies of your data on multiple servers.

### Primary-Replica Architecture

```
Writes → Primary (Master) DB
              │
              ├── Replication ──→ Replica 1 (Slave) ← Reads
              ├── Replication ──→ Replica 2 (Slave) ← Reads
              └── Replication ──→ Replica 3 (Slave) ← Reads
```

- **All writes** go to the primary
- **Reads** are distributed across replicas
- Replicas are copies of the primary, updated via replication

**Why?** Most applications are read-heavy (90% reads, 10% writes). By having 4 replicas handle reads, you've increased read capacity by 4x without touching the primary.

### Synchronous vs Asynchronous Replication

| | Synchronous | Asynchronous |
|-|-------------|-------------|
| **How** | Primary waits for replica to confirm write | Primary writes and moves on; replica catches up later |
| **Consistency** | Strong — replicas always up-to-date | Eventual — replicas may be slightly behind |
| **Speed** | Slower writes (waiting for confirmation) | Faster writes |
| **Data loss risk** | Zero (if replica confirmed) | Small window of potential loss |
| **Use when** | Financial data, inventory | Social media, analytics, logs |

### Replication Lag — The Gotcha

With async replication, a user updates their profile (write to primary) and immediately views it (read from replica). The replica hasn't received the update yet — the user sees old data.

**Fixes:**
- **Read-after-write consistency:** After a write, read from the primary for a short window
- **Sticky sessions:** Route a user to the same replica consistently
- **Minimum replication lag:** Monitor and alert if lag exceeds a threshold

---

## 10. Database Sharding — Splitting Data Across Machines

### The Problem

Your single database has 500 million rows. Even with indexes and replicas, it's too big. Queries are slow, backups take hours, and you're running out of disk space.

### The Analogy

One filing cabinet has 10 million documents. Finding anything takes forever. Solution: buy 10 filing cabinets and put documents A-C in cabinet 1, D-F in cabinet 2, and so on. Each cabinet is smaller and faster to search. That's **sharding** — splitting data across multiple database servers.

### How Sharding Works

```
                    ┌─── Shard 1: Users A-G     (DB Server 1)
                    │
App → Router ───────┼─── Shard 2: Users H-N     (DB Server 2)
  (shard key:       │
   first letter     ├─── Shard 3: Users O-T     (DB Server 3)
   of username)     │
                    └─── Shard 4: Users U-Z     (DB Server 4)
```

### Sharding Strategies

#### 1. Range-Based Sharding
Split by ranges of the shard key (e.g., user IDs 1-1M on shard 1, 1M-2M on shard 2).

**Problem:** Hotspots. If new users (highest IDs) get more traffic, the last shard is overloaded while older shards sit idle.

#### 2. Hash-Based Sharding
Hash the shard key and use modulo to pick a shard: `shard = hash(userId) % numShards`.

**Pros:** Even distribution.
**Cons:** Adding a new shard requires rehashing and redistributing data (unless you use consistent hashing).

#### 3. Directory-Based Sharding
A lookup table maps each key to a shard. Flexible but the directory itself becomes a bottleneck and single point of failure.

### Sharding Challenges

| Challenge | Why It's Hard |
|-----------|--------------|
| **Cross-shard queries** | Joining data across shards is expensive — need to query multiple DBs and merge results |
| **Transactions across shards** | ACID transactions across multiple databases are extremely complex |
| **Resharding** | Adding/removing shards means moving massive amounts of data |
| **Hotspots** | One shard getting more traffic than others |
| **Operational complexity** | 10 database servers instead of 1 — 10x the backups, monitoring, upgrades |

**Rule of thumb:** Don't shard until you absolutely must. Try indexes, caching, read replicas, and query optimisation first. Most applications never need sharding. When you do need it, use a database that supports it natively (MongoDB, CockroachDB, Vitess for MySQL).

---

## 11. CAP Theorem & Consistency Models

### The CAP Theorem

In any distributed system, you can have at most **two out of three**:

| Letter | Stands For | Meaning |
|--------|-----------|---------|
| **C** | Consistency | Every read returns the most recent write |
| **A** | Availability | Every request gets a response (even if not the latest data) |
| **P** | Partition Tolerance | System works even if network connections between nodes fail |

**The reality:** Network partitions WILL happen (you can't avoid P). So the real choice is between **Consistency** and **Availability** during a partition.

### The Analogy

You and your friend have copies of the same spreadsheet. The internet goes down (partition). Someone calls you to update a value.

- **CP (Consistency + Partition Tolerance):** You refuse to update until you can sync with your friend. The caller gets no answer (unavailable) but data is always consistent.
- **AP (Availability + Partition Tolerance):** You update your copy immediately. The caller gets an answer fast. But your friend's copy is now different (inconsistent). You'll sync later.

### Where Real Databases Fall

| System | Priority | Example |
|--------|----------|---------|
| **PostgreSQL, MySQL** | CP (Consistency) | Banking — never show wrong balance |
| **Cassandra, DynamoDB** | AP (Availability) | Social media — always respond, eventual consistency is fine |
| **MongoDB** | CP by default | Configurable — can tune toward AP |
| **Redis** | AP | Caching — availability matters more |

### Consistency Models

| Model | What It Guarantees | Example |
|-------|-------------------|---------|
| **Strong** | Every read sees the latest write | Bank balance after transfer |
| **Eventual** | Reads will eventually see the latest write | Social media likes count |
| **Causal** | If A causes B, everyone sees A before B | Chat messages in order |
| **Read-your-writes** | You always see your own writes | Profile update |

---

## 12. Consistent Hashing — The Key to Distributed Systems

### The Problem with Regular Hashing

You have 4 cache servers and use `server = hash(key) % 4` to decide which server stores each key. This works until you add a 5th server — now it's `hash(key) % 5`. Almost every key maps to a different server. Your entire cache is invalidated. Massive database load.

### The Analogy

Imagine a clock. Your servers are placed at specific hours (12, 3, 6, 9). Each data item is also placed at an hour (based on its hash). A data item goes to the first server clockwise from its position. If you remove the server at 3, only the data between 12 and 3 moves to the next server (at 6). Everything else stays put.

### How It Works

```
            12 (Server A)
           /              \
         /                  \
       9                      3 (Server B)
   (Server D)                  
         \                  /
           \              /
            6 (Server C)

Data item "user:42" hashes to position 2 → goes to Server B (next clockwise)
Data item "user:99" hashes to position 7 → goes to Server D (next clockwise)

Remove Server B? Only data between 12-3 moves to Server C.
All other data stays on its current server.
```

### Where It's Used

- **Distributed caches** (Redis Cluster, Memcached)
- **Load balancers** (consistent routing)
- **CDNs** (routing content to edge servers)
- **Database sharding** (distributing data across shards)
- **Distributed storage** (Amazon DynamoDB, Apache Cassandra)

---

# PART 3 — COMMUNICATION & PROCESSING

---

## 13. Message Queues & Async Processing

### The Problem

A user places an order. Your system needs to: process payment (2s), update inventory (500ms), send email (1s), send SMS (800ms), update analytics (300ms), generate invoice (1s). Synchronously, the user waits 5.6 seconds for a response.

### The Analogy

At a restaurant, when you order food, the waiter doesn't stand in the kitchen watching the chef cook. The waiter writes the order on a slip (the **message**), pins it on the board (the **queue**), and goes to serve other tables. The kitchen (the **worker/consumer**) picks up slips and processes them in order.

### How Message Queues Work

```
Producer → [Message Queue] → Consumer

Order API → [ Payment Job | Email Job | SMS Job | Analytics Job ] → Workers
              (queued)       (queued)    (queued)   (queued)
```

The user gets a response immediately after the order is created. Everything else happens asynchronously via the queue.

```javascript
// Producer: Order API
app.post('/orders', async (req, res) => {
  const order = await OrderRepository.save(req.body);

  // Don't do these synchronously — push to queue
  await queue.publish('order.created', {
    orderId: order.id,
    userId: order.userId,
    items: order.items,
    total: order.total,
  });

  // User gets response immediately (order is saved)
  res.status(201).json(order);
});

// Consumer: Email Worker (separate process/server)
queue.subscribe('order.created', async (message) => {
  const { orderId, userId } = message;
  const user = await UserRepository.findById(userId);
  await EmailService.sendConfirmation(user.email, orderId);
});

// Consumer: Inventory Worker (separate process/server)
queue.subscribe('order.created', async (message) => {
  const { items } = message;
  for (const item of items) {
    await InventoryService.deduct(item.productId, item.quantity);
  }
});
```

### Key Concepts

| Concept | What It Means | Analogy |
|---------|--------------|---------|
| **Producer** | Sends messages to the queue | The waiter writing order slips |
| **Consumer** | Reads and processes messages from the queue | The kitchen cooking orders |
| **Queue** | Ordered buffer of messages | The order board in the kitchen |
| **Topic/Exchange** | Routes messages to specific queues | Separating food orders from drink orders |
| **Dead Letter Queue (DLQ)** | Where failed messages go after max retries | A "problem orders" pile |
| **Acknowledgment** | Consumer confirms it processed the message | Kitchen marks the slip as "done" |

### Popular Message Queues

| Tool | Best For | Used By |
|------|---------|---------|
| **RabbitMQ** | Task queues, work distribution, routing | General purpose |
| **Apache Kafka** | Event streaming, high throughput, log aggregation | LinkedIn, Netflix, Uber |
| **AWS SQS** | Simple queue with zero management | AWS users |
| **Redis (Bull/BullMQ)** | Job queues in Node.js apps | Small-medium apps |

### RabbitMQ vs Kafka — When to Use Which

| | RabbitMQ | Kafka |
|-|----------|-------|
| **Model** | Message broker (messages consumed and deleted) | Event log (messages stored and replayable) |
| **Speed** | Fast (thousands/sec) | Very fast (millions/sec) |
| **Message retention** | Deleted after consumption | Kept for configured period (days/weeks) |
| **Replay** | Can't replay consumed messages | Can replay from any point in time |
| **Use case** | Task queues, notifications, work distribution | Event streaming, analytics, real-time data pipelines |
| **Analogy** | Post office (letter delivered, then gone) | Newspaper archive (can read any old edition) |

---

## 14. Event-Driven Architecture

### Events vs Commands

| | Command | Event |
|-|---------|-------|
| **Tense** | Imperative ("send email") | Past tense ("order placed") |
| **Intent** | Tells someone to do something | Announces that something happened |
| **Coupling** | Sender knows the receiver | Sender doesn't know who listens |
| **Example** | `SendEmailCommand` | `OrderPlacedEvent` |

### Event-Driven Patterns

#### 1. Event Notification
When something happens, publish an event. Interested services react.

```
Order Service: "OrderPlaced" → Email Service picks it up
                              → Inventory Service picks it up
                              → Analytics Service picks it up
```

#### 2. Event Sourcing
Instead of storing current state, store every event that ever happened. Current state is derived by replaying events.

```
Events for Order #789:
1. OrderCreated { items: [...], total: 5000 }
2. PaymentReceived { amount: 5000 }
3. ItemShipped { trackingId: "TRK123" }
4. ItemDelivered { timestamp: "2024-01-20" }

Current state = replay all events → Order is "delivered"
```

**Why?** Complete audit trail, can reconstruct state at any point in time, can derive new views by replaying events. Used in banking, e-commerce, and any domain where "why did this happen?" matters.

#### 3. CQRS (Command Query Responsibility Segregation)
Separate the read model from the write model. Writes go to one database (optimised for writes). Reads go to a different database (optimised for reads).

```
Write side:   Command → Validate → Save to Write DB → Publish Event
Read side:    Event → Update Read DB (denormalised, pre-computed)
              Query → Read from Read DB → Return fast
```

**Example:** An e-commerce product page. Writes (update price, stock) go to a normalised PostgreSQL. An event updates a denormalised MongoDB document with pre-computed data (product + reviews + ratings + seller info). Reads serve this single document — fast, no joins.

---

## 15. Microservices vs Monolith

### Monolith — One Big Application

Everything is in one codebase, one deployable, one database.

```
┌──────────────────────────────────┐
│         Monolith App              │
│  ┌──────┐ ┌──────┐ ┌──────────┐ │
│  │Users │ │Orders│ │Payments  │ │
│  └──┬───┘ └──┬───┘ └────┬─────┘ │
│     └────────┼──────────┘        │
│           Database               │
└──────────────────────────────────┘
```

### Microservices — Many Small Applications

Each business domain is a separate service with its own database, deployed independently.

```
┌──────────┐  ┌──────────┐  ┌────────────┐
│User       │  │Order      │  │Payment      │
│Service    │  │Service    │  │Service      │
│   + DB    │  │   + DB    │  │   + DB      │
└──────────┘  └──────────┘  └────────────┘
     │              │              │
     └──────── Message Queue ──────┘
```

### Honest Comparison

| Aspect | Monolith | Microservices |
|--------|----------|---------------|
| **Complexity** | Simple to develop, deploy, test | Complex networking, deployment, debugging |
| **Scaling** | Scale everything together | Scale each service independently |
| **Team** | Works for small teams (< 10 devs) | Needed for large teams (10+ devs per service) |
| **Deploy** | One deployment | Many independent deployments |
| **Technology** | One stack | Each service can use different tech |
| **Data** | One database, easy joins | Each service owns its data, no cross-service joins |
| **Failure** | One bug can crash everything | One service failing doesn't crash others (ideally) |
| **Debugging** | Stack trace shows everything | Need distributed tracing across services |
| **Cost** | Lower infrastructure cost | Higher (more servers, monitoring, networking) |

### When to Use Which

**Start with a Monolith when:**
- Small team (under 10 developers)
- New product / startup (requirements change rapidly)
- Simple domain
- You haven't figured out service boundaries yet

**Move to Microservices when:**
- Team is big enough that they step on each other in the monolith
- Different parts of the system need different scaling (chat needs 100 servers, admin needs 1)
- You need independent deployments (payment team deploys without waiting for the feed team)
- You need different technology stacks for different domains

**The Golden Rule:** If you can't build a well-structured monolith, you can't build microservices. Microservices don't fix bad design — they amplify it. A poorly designed monolith becomes 50 poorly designed services that can't talk to each other.

---

## 16. API Gateway — The Front Door

### What It Does

An API Gateway sits between clients and your backend services. All traffic goes through it.

```
Mobile App  ─→
Web App     ─→  API Gateway  ─→ User Service
Partner API ─→       │        ─→ Order Service
                     │        ─→ Payment Service
                     │        ─→ Search Service
                     │
                Handles:
                - Authentication
                - Rate Limiting
                - Routing
                - Load Balancing
                - Logging
                - SSL Termination
                - Response Caching
                - Request/Response Transformation
```

### Why Not Just Let Clients Call Services Directly?

Without a gateway, the mobile app needs to know the address of every service, handle auth for each, manage failures for each, and deal with different protocols. The gateway centralises all of that.

### Popular API Gateways

| Tool | Type |
|------|------|
| **Kong** | Open-source, plugin-based |
| **AWS API Gateway** | Managed, serverless |
| **Nginx** | Can act as a simple API gateway |
| **Traefik** | Container-native, auto-discovery |
| **Spring Cloud Gateway** | Java/Spring ecosystem |

---

## 17. Service Discovery & Communication

### The Problem

In a microservices world, services need to find each other. But services spin up and die constantly (auto-scaling, deployments, failures). Hardcoding IP addresses doesn't work.

### Service Discovery

| Type | How | Example |
|------|-----|---------|
| **Client-Side Discovery** | Client queries a registry, picks a server | Netflix Eureka |
| **Server-Side Discovery** | Load balancer/router queries registry, routes client | AWS ALB, Kubernetes Services |
| **DNS-Based** | Services register DNS names, clients resolve | Consul, CoreDNS |

### How Services Talk

| Method | When to Use |
|--------|-------------|
| **REST/HTTP** | Synchronous, request-response, simple |
| **gRPC** | Synchronous, high-performance, binary protocol, strong typing |
| **Message Queue** | Asynchronous, fire-and-forget, decoupled |
| **Events** | Asynchronous, one-to-many, pub-sub |

**Rule of thumb:** Use synchronous (REST/gRPC) when the caller needs the response immediately. Use asynchronous (queues/events) when the caller can continue without waiting.

---

# PART 4 — RELIABILITY & PERFORMANCE

---

## 18. Rate Limiting at Scale

### Algorithms

#### 1. Token Bucket
Imagine a bucket that holds 100 tokens. Each request costs 1 token. Tokens refill at 10 per second. If the bucket is empty, requests are rejected. Allows bursts up to the bucket size.

#### 2. Sliding Window
Count requests in a rolling time window (e.g., last 60 seconds). More accurate than fixed windows, smoother than token bucket.

#### 3. Fixed Window Counter
Count requests per fixed time window (e.g., per minute). Simple but allows bursts at window boundaries.

### Distributed Rate Limiting

With multiple app servers, rate limits must be shared. Use Redis as the central counter.

```javascript
async function isRateLimited(userId, limit = 100, windowSecs = 60) {
  const key = `ratelimit:${userId}`;
  const current = await redis.incr(key);

  if (current === 1) {
    await redis.expire(key, windowSecs);
  }

  return current > limit;
}
```

---

## 19. Circuit Breaker — Failing Gracefully

### The Analogy

An electrical circuit breaker in your house. When there's a power surge, the breaker trips and cuts the circuit — preventing a fire. You flip it back when the issue is resolved. A software circuit breaker does the same — when a service keeps failing, the circuit breaker "trips" and stops sending requests to it.

### The Three States

```
CLOSED (Normal) ──── failures exceed threshold ──── OPEN (Failing)
     ↑                                                  │
     │                                                   │
     └── success ── HALF-OPEN (Testing) ← timeout expires ┘
```

- **CLOSED:** Everything is normal. Requests go through. Track failures.
- **OPEN:** Too many failures. Stop sending requests immediately (return error or fallback). Wait for a timeout.
- **HALF-OPEN:** After timeout, let a few test requests through. If they succeed, go back to CLOSED. If they fail, go back to OPEN.

```javascript
class CircuitBreaker {
  constructor(fn, options = {}) {
    this.fn = fn;
    this.failureThreshold = options.failureThreshold || 5;
    this.resetTimeout = options.resetTimeout || 30000;
    this.state = 'CLOSED';
    this.failureCount = 0;
    this.lastFailureTime = null;
  }

  async call(...args) {
    if (this.state === 'OPEN') {
      if (Date.now() - this.lastFailureTime > this.resetTimeout) {
        this.state = 'HALF-OPEN';
      } else {
        throw new Error('Circuit is OPEN — service unavailable');
      }
    }

    try {
      const result = await this.fn(...args);
      this.onSuccess();
      return result;
    } catch (error) {
      this.onFailure();
      throw error;
    }
  }

  onSuccess() {
    this.failureCount = 0;
    this.state = 'CLOSED';
  }

  onFailure() {
    this.failureCount++;
    this.lastFailureTime = Date.now();
    if (this.failureCount >= this.failureThreshold) {
      this.state = 'OPEN';
    }
  }
}

// Usage
const paymentBreaker = new CircuitBreaker(
  (amount) => stripeClient.charge(amount),
  { failureThreshold: 5, resetTimeout: 30000 }
);

try {
  await paymentBreaker.call(5000);
} catch (err) {
  // Return a fallback or queued response
}
```

---

## 20. Monitoring, Logging & Observability

### The Three Pillars

| Pillar | What It Answers | Tools |
|--------|----------------|-------|
| **Metrics** | How is the system performing right now? | Prometheus, Grafana, Datadog, CloudWatch |
| **Logging** | What happened? (detailed event records) | ELK Stack, Loki, CloudWatch Logs |
| **Tracing** | How did a request flow through the system? | Jaeger, Zipkin, AWS X-Ray |

### Key Metrics to Track (RED Method)

| Metric | What |
|--------|------|
| **R**ate | Requests per second |
| **E**rrors | Error rate (% of failed requests) |
| **D**uration | Response time (p50, p95, p99) |

**Why p99 matters more than average:** If your average response time is 100ms but p99 is 5s, that means 1 in 100 users waits 5 seconds. At 10,000 users per minute, 100 people per minute have a terrible experience. Averages hide the pain.

### Structured Logging

```javascript
// BAD — unstructured
console.log('User 42 placed order 789 for $150');

// GOOD — structured (parseable by log aggregation tools)
logger.info({
  event: 'order_placed',
  userId: 42,
  orderId: 789,
  total: 150,
  paymentMethod: 'credit_card',
  requestId: req.id,
  duration: 235,
});
```

### Distributed Tracing

In microservices, one user request might hit 5 services. If it's slow, which service is the bottleneck? Distributed tracing assigns a **trace ID** to each request and passes it through every service.

```
Request (trace-id: abc-123)
  → API Gateway (50ms)
    → User Service (30ms)
    → Order Service (200ms) ← BOTTLENECK
      → Inventory Service (15ms)
      → Payment Service (150ms) ← SECONDARY BOTTLENECK
```

---

## 21. Data Redundancy, Backup & Disaster Recovery

### Backup Strategies

| Strategy | How Often | What It Captures |
|----------|----------|-----------------|
| **Full backup** | Weekly/Daily | Everything |
| **Incremental** | Hourly | Only changes since last backup |
| **Point-in-time recovery** | Continuous | Can restore to any second |

### Disaster Recovery Metrics

| Metric | What It Means |
|--------|--------------|
| **RPO** (Recovery Point Objective) | How much data can you afford to lose? (1 hour = you accept losing the last hour of data) |
| **RTO** (Recovery Time Objective) | How long can you be down? (5 minutes = you must be back in 5 minutes) |

### Multi-Region Deployment

For critical systems, deploy across multiple geographic regions:

```
Region 1 (Mumbai)  ←→  Region 2 (Singapore)  ←→  Region 3 (Frankfurt)
  App Servers           App Servers               App Servers
  Database              Database Replica          Database Replica
  Cache                 Cache                     Cache
```

If Mumbai goes down (natural disaster, data center fire), traffic routes to Singapore. Users experience minimal disruption.

---

# PART 5 — REAL SYSTEM DESIGN CASE STUDIES

---

## 22. How to Approach Any System Design Problem

### The Framework (Use This in Every Interview)

**Step 1: Requirements Clarification (2-3 minutes)**
Ask what features to focus on. Clarify scale (users, data volume, requests per second). Understand read vs write ratio. Identify latency requirements.

**Step 2: Capacity Estimation (3-5 minutes)**
Back-of-the-envelope math: total users, daily active users, requests per second, storage needed, bandwidth.

**Step 3: High-Level Architecture (5-10 minutes)**
Draw the major components: clients, load balancers, app servers, databases, caches, queues. Show how data flows.

**Step 4: Deep Dive (10-15 minutes)**
Dig into the most important/challenging component. Database schema, caching strategy, real-time delivery, etc.

**Step 5: Identify Bottlenecks & Scale (5 minutes)**
Where will the system break under load? How do you address it? Single points of failure?

### Estimation Cheat Sheet

| Metric | Rough Number |
|--------|-------------|
| 1 day | 86,400 seconds (~100K for estimation) |
| 1 month | ~2.5 million seconds |
| 1 year | ~30 million seconds |
| 1 char | 1 byte |
| 1 image (medium) | ~200 KB |
| 1 video (1 min, compressed) | ~10 MB |
| Average tweet/post | ~300 bytes |
| SSD read | ~0.1 ms |
| Memory read | ~0.0001 ms (100 ns) |
| Network round trip (same datacenter) | ~0.5 ms |
| Network round trip (cross-continent) | ~150 ms |

---

## 23. Design a URL Shortener (like bit.ly)

### Requirements
- Given a long URL, generate a short URL
- Given a short URL, redirect to the original URL
- Optional: analytics (click counts), custom short URLs, expiry

### Capacity Estimation
- 100M new URLs per month = ~40 URLs/second (write)
- 10:1 read-to-write ratio = 400 redirects/second (read)
- Each URL record ~500 bytes
- 100M × 500 bytes × 12 months × 5 years = ~300 GB storage

### Architecture

```
Client → Load Balancer → App Servers → Cache (Redis)
                                     → Database (PostgreSQL/NoSQL)

Write Flow:
1. Client sends long URL
2. App generates unique short code (Base62 encoding of auto-increment ID or hash)
3. Store mapping { shortCode → longURL } in DB + Cache
4. Return short URL

Read Flow:
1. Client accesses short URL
2. App checks Cache for mapping (fast path)
3. Cache miss → query DB → populate cache
4. Return 301/302 redirect to long URL
```

### Short Code Generation

```
Approach 1: Counter + Base62
- Auto-increment counter: 1, 2, 3, ...
- Convert to Base62: 1 → "1", 62 → "10", 3844 → "100"
- 6 characters of Base62 = 62^6 = ~57 billion unique codes
- Use a distributed counter (Redis INCR) to avoid collisions

Approach 2: MD5/SHA Hash
- Hash the URL, take first 6-7 characters
- Handle collisions (check if code already exists)

Approach 3: Pre-generated codes
- Generate millions of unique codes offline
- When a new URL comes in, grab the next available code from the pool
```

### Database Schema

```sql
CREATE TABLE urls (
    id BIGSERIAL PRIMARY KEY,
    short_code VARCHAR(10) UNIQUE NOT NULL,
    original_url TEXT NOT NULL,
    created_at TIMESTAMP DEFAULT NOW(),
    expires_at TIMESTAMP,
    click_count BIGINT DEFAULT 0
);

CREATE INDEX idx_short_code ON urls(short_code);
```

---

## 24. Design a Chat System (like WhatsApp)

### Requirements
- One-on-one messaging
- Group chats (up to 500 members)
- Online/offline status
- Read receipts
- Message history

### Architecture

```
Client → Load Balancer → WebSocket Servers → Message Router
                              ↓                    ↓
                        Connection Registry    Message Queue
                           (Redis)           (Kafka/RabbitMQ)
                                                   ↓
                                             Message Store
                                            (Cassandra/MongoDB)
                                                   ↓
                                          Push Notification Service
                                          (for offline users)
```

### Key Design Decisions

**1. How are messages delivered in real-time?**
WebSocket connections. Each user maintains a persistent WebSocket to a server. The server pushes messages through this connection instantly.

**2. What if the recipient is connected to a different server?**
Use a Connection Registry (Redis) that maps `userId → serverId`. When Server A needs to send a message to a user on Server B, it routes through the message queue.

**3. What if the recipient is offline?**
Store the message in the database. When the user comes online, the server fetches undelivered messages and sends them. Also send a push notification.

**4. How to store messages?**
Use a NoSQL database like Cassandra. Partition by `chatId`, cluster by `timestamp`. This gives fast writes and efficient retrieval of recent messages for a chat.

```
Table: messages
Partition key: chat_id
Clustering key: timestamp (DESC)

| chat_id | timestamp           | sender_id | content        | status    |
|---------|---------------------|-----------|----------------|-----------|
| chat_1  | 2024-01-15 10:30:01 | user_42   | "Hey!"         | delivered |
| chat_1  | 2024-01-15 10:30:15 | user_43   | "Hi! How are?" | sent      |
```

**5. Group messages?**
Fan-out on write: when a user sends a message to a group, the server writes one copy to the group's message store and sends it to each online member's WebSocket. For offline members, queue a push notification.

---

## 25. Design a News Feed (like Twitter/Instagram)

### The Core Problem

When User A opens their feed, they should see posts from everyone they follow, sorted by recency or relevance. With millions of users and billions of follow relationships, this is non-trivial.

### Two Approaches

#### Approach 1: Fan-Out on Write (Push Model)
When a user creates a post, immediately write it to every follower's feed.

```
User A posts → Write to Follower 1's feed
             → Write to Follower 2's feed
             → Write to Follower N's feed
```

**Pros:** Reading the feed is instant (pre-computed).
**Cons:** Writing is expensive for popular users (a celebrity with 10M followers = 10M writes per post). Wasted work if most followers don't check their feed.

#### Approach 2: Fan-Out on Read (Pull Model)
When a user opens their feed, query posts from all users they follow at that moment.

```
User opens feed → Get list of users they follow
                → Query recent posts from each followed user
                → Merge and sort by timestamp
                → Return top N posts
```

**Pros:** Writing is cheap (just store the post once). No wasted work.
**Cons:** Reading is slow — need to query and merge posts from potentially thousands of followed users.

#### Approach 3: Hybrid (What Twitter/Instagram Actually Do)
- **For regular users** (< 10K followers): fan-out on write. Pre-compute their followers' feeds.
- **For celebrities** (> 10K followers): fan-out on read. When a user opens their feed, mix the pre-computed feed with fresh posts from celebrities they follow.

### Architecture

```
Post Service → (fan-out) → Feed Cache (Redis) ← Feed Service ← User
                   ↓
            Message Queue → Feed Workers (build feeds)
                   ↓
            Post Database (store all posts)

Celebrity posts: fetched at read time and merged with cached feed
```

---

## 26. Design a Notification System

### Requirements
- Support email, SMS, push notifications
- Millions of notifications per day
- Retry failed notifications
- Rate limiting (don't spam users)
- User preferences (opt-in/opt-out per channel)

### Architecture

```
Events (order placed, etc.)
       ↓
Notification Service → User Preferences DB → Skip if opted out
       ↓
Priority Queue (Kafka/SQS)
       ↓
┌──────────────┬──────────────┬──────────────┐
│ Email Worker  │ SMS Worker    │ Push Worker    │
│ (SendGrid)    │ (Twilio)      │ (FCM/APNS)    │
└──────────────┴──────────────┴──────────────┘
       ↓
Notification Log (tracking + analytics)
       ↓
Dead Letter Queue (for failed notifications → retry)
```

---

## 27. Design a Rate Limiter

### Requirements
- Limit API requests per user/IP
- Distributed across multiple servers
- Low latency (can't slow down every request)

### Architecture

```
Client → API Gateway → Rate Limiter → App Server
                           ↓
                     Redis (counters)
```

### Sliding Window with Redis

```
For each request:
1. Get current timestamp
2. Key = "ratelimit:{userId}:{currentMinute}"
3. INCR the key in Redis
4. If count > limit → reject (429)
5. Set TTL on the key (auto-cleanup)
```

---

## 28. Design a File Storage System (like Google Drive)

### Requirements
- Upload and download files
- Share files with other users
- Sync across devices
- Version history
- Support large files (GBs)

### Architecture

```
Client → API Gateway → Metadata Service → Metadata DB (PostgreSQL)
                     → Upload Service → Object Storage (S3)
                     → Sync Service (WebSocket) → Notification Queue

Upload Flow:
1. Client requests upload URL (pre-signed S3 URL)
2. Client uploads directly to S3 (bypasses your servers)
3. S3 notifies your service (webhook/event)
4. Metadata Service records file info (name, size, owner, version)
5. Sync Service notifies other devices of the user

Key Decisions:
- Chunked uploads for large files (split into 4MB chunks)
- Deduplication (if the same chunk exists, don't store twice)
- Block-level sync (only sync changed blocks, not entire files)
```

---

## 29. Design an E-Commerce System

### Core Services

```
┌─────────────┐  ┌──────────────┐  ┌──────────────┐
│ User Service │  │ Product      │  │ Cart Service  │
│              │  │ Catalog      │  │              │
│   + Auth     │  │ Service      │  │   + Redis    │
└─────────────┘  └──────────────┘  └──────────────┘
        │               │                  │
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Order Service │  │ Payment      │  │ Inventory    │
│              │  │ Service      │  │ Service      │
│  + Order DB  │  │ (Stripe)     │  │  + DB        │
└──────────────┘  └──────────────┘  └──────────────┘
        │               │                  │
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│ Notification │  │ Search       │  │ Shipping     │
│ Service      │  │ Service      │  │ Service      │
│ (email/SMS)  │  │ (Elastic)    │  │              │
└──────────────┘  └──────────────┘  └──────────────┘
```

### The Checkout Flow

```
1. User clicks "Checkout"
2. Cart Service → get cart items
3. Inventory Service → verify stock (optimistic locking)
4. Pricing Service → calculate total (discounts, tax, shipping)
5. Payment Service → charge (with idempotency key)
6. Order Service → create order record
7. Inventory Service → reserve stock
8. Notification Service → send confirmation (async via queue)
9. Return order confirmation to user

Key: Steps 5-7 should be in a SAGA pattern for distributed transactions
```

### Saga Pattern for Distributed Transactions

In a monolith, you use a database transaction: either everything succeeds or everything rolls back. In microservices, there's no single database. The Saga pattern orchestrates a sequence of local transactions with compensating actions.

```
Step 1: Reserve Inventory    → Compensate: Release Inventory
Step 2: Process Payment      → Compensate: Refund Payment
Step 3: Create Order         → Compensate: Cancel Order
Step 4: Send Notification    → (no compensation needed)

If Step 2 fails → run Step 1 compensation (release inventory)
If Step 3 fails → run Step 2 compensation (refund) + Step 1 compensation (release)
```

---

# PART 6 — INTERVIEW QUESTIONS

---

## 30. System Design Interview Questions (80+ Questions)

### Fundamentals

**Q1: What is the difference between horizontal and vertical scaling?**
Vertical scaling means adding more power to a single machine (more CPU, RAM). Horizontal scaling means adding more machines and distributing the load. Vertical is simpler but has a ceiling and is a single point of failure. Horizontal scales infinitely but introduces distributed system complexity — you need stateless design, load balancers, and shared data stores.

**Q2: What is a load balancer? What algorithms does it use?**
A load balancer distributes incoming requests across multiple servers. Common algorithms: Round Robin (even distribution), Least Connections (route to the least busy server), IP Hash (consistent routing for same client), and Weighted (send more traffic to more powerful servers). It also performs health checks, removing failed servers from the pool. Layer 4 operates on TCP; Layer 7 operates on HTTP and can route based on URLs.

**Q3: What is caching? When should you NOT use a cache?**
Caching stores frequently accessed data in a faster storage layer (Redis, in-memory) to reduce database load and improve response time. Don't cache: data that changes more frequently than it's read, data that must be perfectly consistent (payment balances), data with very low access frequency (caching wastes memory), and large blobs that change often. The key question is: is this data read more often than it's written?

**Q4: Explain the CAP theorem in simple terms.**
In a distributed system, when a network partition (communication failure between nodes) occurs, you must choose between Consistency (all nodes see the same data) and Availability (all requests get a response). You can't have both during a partition. Banks choose Consistency (never show wrong balance). Social media chooses Availability (always show something, even if slightly stale). In normal operation (no partition), you can have both.

**Q5: What is database replication? What problem does it solve?**
Replication maintains copies of a database on multiple servers. It solves two problems: reliability (if the primary crashes, a replica takes over — no data loss) and performance (reads are distributed across replicas, multiplying read capacity). The trade-off is replication lag — replicas might be slightly behind the primary with async replication.

**Q6: What is database sharding? When do you need it?**
Sharding splits data across multiple database servers (shards) based on a shard key. You need it when: the data is too large for a single server, write throughput exceeds what one server can handle, or you need to reduce query scope (smaller dataset per shard = faster queries). Most applications never need sharding — try indexing, caching, and read replicas first.

**Q7: What is a CDN and how does it work?**
A Content Delivery Network is a network of servers distributed globally that cache and serve content from locations near users. When a user in Mumbai requests an image, the CDN serves it from a server in Mumbai (20ms) instead of from a server in the US (200ms). It works by caching content at edge locations. First request fetches from the origin server; subsequent requests are served from the edge.

**Q8: What is a message queue? Why use one?**
A message queue is a buffer that decouples producers (who send messages) from consumers (who process them). Use one when: you want to process tasks asynchronously (email sending, image processing), you need to handle traffic spikes (queue absorbs bursts), you want to decouple services (producer doesn't care how many consumers exist), or you need reliable processing with retries.

**Q9: What is the difference between SQL and NoSQL databases?**
SQL databases store data in tables with fixed schemas, support ACID transactions, and excel at complex joins and relationships. NoSQL databases support flexible schemas, horizontal scaling, and high throughput, but with weaker consistency guarantees. Use SQL when data integrity and relationships are critical (e-commerce, banking). Use NoSQL when you need flexibility and scale (social media feeds, IoT data, caching).

**Q10: What is a reverse proxy?**
A server that sits in front of backend servers, forwarding client requests to them. Clients don't know which backend server handled their request. It provides: load balancing, SSL termination, caching, compression, security (hiding backend topology), and request routing. Nginx is the most popular reverse proxy.

---

### Architecture & Scaling

**Q11: How would you scale a system from 100 users to 10 million?**
Stage 1 (100 users): single server with app + DB. Stage 2 (1K): separate DB from app server. Stage 3 (10K): add Redis cache, reduce DB load. Stage 4 (100K): horizontal scale app servers behind a load balancer, add DB read replicas. Stage 5 (1M): CDN for static content, message queues for async tasks, database indexing optimisation. Stage 6 (10M): database sharding, microservices for independent scaling, multi-region deployment.

**Q12: What is database indexing and when should you use it?**
An index is a data structure (usually B-Tree) that speeds up data retrieval at the cost of slower writes and extra storage. Use indexes on columns in WHERE, JOIN, ORDER BY, and GROUP BY clauses. Don't index: columns that change constantly, columns with low cardinality (boolean), or tables with very few rows. A missing index is the #1 cause of slow queries in production.

**Q13: Explain the difference between strong and eventual consistency.**
Strong consistency: every read returns the most recent write. After updating your profile, you immediately see the change. Eventual consistency: reads may return stale data for a brief period, but will eventually reflect the latest write. Social media likes might show 999 instead of 1000 for a few seconds. Strong consistency is safer but slower; eventual consistency is faster and more available.

**Q14: What are microservices? When should you use them?**
Microservices split an application into small, independently deployable services, each owning its own data. Use them when: teams are large enough that they block each other in a monolith, different components need different scaling, or you need independent deployment cycles. Don't use them for: small teams, new products with unclear requirements, or simple applications. Start monolithic, extract services when pain points are clear.

**Q15: What is an API Gateway?**
A single entry point for all client requests in a microservices architecture. It handles cross-cutting concerns: authentication, rate limiting, routing, logging, SSL termination, and response caching. Without it, every client needs to know every service's address and handle auth individually. It's like a reception desk — all visitors check in through one place.

**Q16: What is the circuit breaker pattern?**
A fault tolerance pattern that prevents cascading failures. When a service fails repeatedly, the circuit breaker "opens" and stops sending requests to it (returning errors or fallbacks immediately). After a timeout, it allows a few test requests through. If they succeed, the circuit "closes" and normal traffic resumes. This prevents one failing service from bringing down the entire system.

**Q17: Explain event-driven architecture.**
Instead of services calling each other directly (tight coupling), services publish events when something happens ("order placed") and other services listen and react. Benefits: loose coupling (publisher doesn't know who listens), easy to add new reactions (just add a new listener), services evolve independently. Challenges: debugging across services, eventual consistency, event ordering.

**Q18: What is a dead letter queue?**
A queue where messages go after they've failed processing multiple times (exceeded max retries). Instead of losing the message, it's moved to the DLQ for investigation. Engineers can inspect failed messages, fix the issue, and replay them. Essential for reliability — you never lose data, even when processing fails.

**Q19: What is consistent hashing?**
A hashing technique for distributed systems where adding or removing a server only affects a small portion of keys (approximately 1/n of the keys, where n is the number of servers). Regular hashing (key % n) redistributes nearly all keys when n changes. Consistent hashing maps servers and keys onto a ring — each key goes to the nearest server clockwise. Used in distributed caches, CDNs, and database sharding.

**Q20: What are read replicas and when do you use them?**
Read replicas are copies of your primary database that handle read queries. Use them when: your application is read-heavy (90%+ reads), the primary database is struggling with load, or you need to run expensive analytics queries without impacting the main application. All writes go to the primary; reads are distributed across replicas. The trade-off is replication lag — replicas may be slightly behind.

---

### Deep Dive Questions

**Q21: How do you handle distributed transactions?**
Distributed transactions across microservices can't use traditional ACID transactions. Use the Saga pattern: a sequence of local transactions where each step has a compensating action. If step 3 fails, run compensations for steps 2 and 1. Two approaches: choreography (each service listens for events and reacts) or orchestration (a central coordinator manages the sequence). Orchestration is easier to understand and debug.

**Q22: How does database sharding affect your application?**
Application needs to know which shard to query (shard-aware routing). Cross-shard queries are expensive — need to query all shards and merge results. Cross-shard transactions are extremely complex. Resharding (adding more shards) requires data migration. Auto-increment IDs collide across shards — use UUIDs or distributed ID generators. Some queries that were easy (global sort, count) become hard.

**Q23: What is the difference between Kafka and RabbitMQ?**
Kafka is an event log — messages are persisted and can be replayed, supports millions of messages per second, ideal for event streaming and data pipelines. RabbitMQ is a message broker — messages are consumed and deleted, supports complex routing (fanout, topic, headers), ideal for task queues and work distribution. Kafka for "what happened" (events). RabbitMQ for "please do this" (commands).

**Q24: How do you ensure exactly-once processing in a distributed system?**
True exactly-once is nearly impossible. The practical approach is "at-least-once delivery + idempotent processing." The queue delivers messages at least once (might deliver duplicates). The consumer uses an idempotency key to detect duplicates — if it's already processed a message with that ID, it skips it. Store processed message IDs in a database or Redis with a TTL.

**Q25: What is the SAGA pattern? Orchestration vs Choreography?**
Orchestration: a central Saga orchestrator tells each service what to do and handles compensations. Like a conductor leading an orchestra. Easier to understand, single point of failure. Choreography: each service listens for events and knows what to do next. Like dancers responding to music. More decoupled, harder to debug, no single point of failure.

**Q26: How do you handle hot partitions/hotspots?**
A hot partition occurs when one shard gets disproportionately more traffic (e.g., a celebrity's data shard). Fixes: choose a shard key with high cardinality, add a random suffix to spread writes (shard key = userId + randomSuffix), use caching to absorb read hotspots, split hot partitions further, or use consistent hashing with virtual nodes for better distribution.

**Q27: What is a write-ahead log (WAL)?**
Before any change is written to the database files, it's first written to a log (the WAL). If the database crashes mid-write, the WAL is replayed on recovery to bring the database back to a consistent state. Every relational database uses WAL. It's also the foundation of database replication — replicas replay the primary's WAL.

**Q28: How do you design for fault tolerance?**
Redundancy at every level (multiple app servers, database replicas, multi-AZ deployment). Health checks and automatic failover. Circuit breakers to prevent cascading failures. Retry with exponential backoff. Graceful degradation (show cached data when the database is down). Dead letter queues for failed processing. Chaos engineering (intentionally break things in production to find weaknesses).

**Q29: What is the difference between latency and throughput?**
Latency is how long one request takes (measured in ms). Throughput is how many requests the system handles per unit of time (measured in requests/second). You can have low latency but low throughput (one fast server), or high throughput but high latency (batch processing). Optimising for one doesn't necessarily optimise the other.

**Q30: Explain CQRS and when to use it.**
Command Query Responsibility Segregation separates read and write operations into different models, potentially different databases. Writes go to a normalised write store; events update a denormalised read store optimised for queries. Use when: read and write patterns are very different, reads need data from multiple services (pre-join into one read store), or you need different scaling for reads vs writes. Overkill for simple CRUD apps.

---

### Scenario-Based Questions

**Q31: How would you design Twitter?**
Core features: tweet, follow, news feed. Architecture: Tweet Service (write tweets to DB), Fan-out Service (push tweets to followers' feeds), Feed Service (read pre-computed feeds from cache), User Service (profiles, follows). For celebrities with millions of followers, use fan-out on read instead of write. Store feeds in Redis (sorted sets by timestamp). Use Kafka for async fan-out. WebSocket for real-time updates.

**Q32: How would you design YouTube?**
Upload Service: accept video, store in blob storage (S3), queue transcoding. Transcoding Service: convert to multiple resolutions. CDN: serve videos from edge locations near users. Metadata Service: title, description, views. Recommendation Service: ML-based. Key challenges: massive storage (petabytes), global delivery (CDN), real-time view counting (Redis counters, async DB writes).

**Q33: How would you design a payment system?**
Idempotency (every payment has a unique key — retries don't double-charge). ACID transactions for money movement. Saga pattern for distributed transactions. Separate ledger for audit trail. Rate limiting on payment endpoints. Fraud detection service. PCI compliance (never store raw card numbers). Retry with exponential backoff. Dead letter queue for failed payments. Event sourcing for complete payment history.

**Q34: How would you design an autocomplete/search suggestion system?**
Use a Trie (prefix tree) data structure. Store it in memory for speed. When a user types "hel", traverse the trie to find all words starting with "hel" and rank by popularity. At scale: shard the trie by first character. Update popularity counts asynchronously. Use Elasticsearch for full-text search. Cache top suggestions for popular prefixes. Data collection: aggregate user search queries with MapReduce.

**Q35: How would you design a ride-sharing system (like Uber)?**
Location Service: drivers send GPS coordinates every few seconds, stored in a geospatial index (Redis with geospatial commands or QuadTree). Matching Service: when a rider requests, find nearby drivers using geospatial queries. Trip Service: manage trip lifecycle (requested → matched → in-progress → completed). Pricing Service: surge pricing based on supply/demand. ETA Service: real-time routing. Notification: push notifications for match updates.

**Q36: How would you design a distributed cache?**
Use consistent hashing to distribute keys across cache nodes. Each node stores a subset of keys. When a node fails, only its keys are redistributed. Add replication (each key stored on 2-3 nodes) for fault tolerance. Use LRU eviction when memory is full. Cache invalidation via TTL and event-based invalidation. Client-side library handles routing. Monitor hit ratio, latency, and memory usage.

**Q37: How would you design a web crawler?**
URL Frontier (priority queue of URLs to crawl). Fetcher (downloads pages, respects robots.txt, rate limits per domain). Parser (extracts text and new URLs). URL Filter (dedup, filter unwanted domains). Storage (raw pages in blob storage, parsed content in search index). Politeness: don't overload any single website. Scale: distribute URLs across multiple crawlers using consistent hashing on domain.

**Q38: How would you design a distributed logging system?**
Agents on each server collect logs and forward them. Message queue (Kafka) ingests log streams from all agents. Stream processors filter and transform logs. Storage tier: hot storage (Elasticsearch for recent, searchable logs) and cold storage (S3 for archival). Query layer: Kibana or Grafana for searching and visualising. Key: handle massive volume, don't lose logs, support real-time search.

**Q39: How would you design a metrics/monitoring system?**
Agents on each server collect metrics (CPU, memory, custom application metrics) and push to a collector. Time-series database (InfluxDB, Prometheus) stores metrics. Dashboards (Grafana) visualise trends. Alerting engine checks thresholds and sends alerts. Key challenges: high write throughput (thousands of metrics per second per server), efficient storage (compression, downsampling old data), fast queries for dashboards.

**Q40: How would you design a global ID generator?**
Requirements: globally unique, roughly sortable by time, high throughput. Twitter Snowflake approach: 64-bit ID = timestamp (41 bits) + machine ID (10 bits) + sequence number (12 bits). Each machine generates up to 4096 IDs per millisecond without coordination. IDs are roughly time-ordered. No single point of failure. Alternative: UUID v4 (random, not sortable), UUID v7 (timestamp-based, sortable).

---

### Trade-off Questions

**Q41: Consistency vs Availability — how do you decide?**
Depends on the business domain. Financial transactions need consistency — showing wrong balance is unacceptable. Social media can tolerate eventual consistency — likes being a few seconds behind is fine. User-facing features often need read-your-writes consistency (you see your own changes immediately). Analytics can be eventually consistent. Ask: "What's the business cost of showing stale data?"

**Q42: SQL vs NoSQL — how do you choose?**
Start with SQL (PostgreSQL) unless you have a specific reason not to. Choose NoSQL when: schema is highly variable (document store), you need extreme write throughput (Cassandra), data is graph-shaped (Neo4j), you need simple key-value lookups at massive scale (DynamoDB), or you need horizontal scaling without complex sharding setup.

**Q43: Push vs Pull for real-time updates?**
Push (WebSocket/SSE): server sends updates immediately. Good for chat, notifications, live scores. Higher server resource usage (maintaining connections). Pull (polling): client periodically asks for updates. Simpler, works everywhere, but wastes bandwidth and adds latency. Long polling: client makes a request, server holds it until there's an update. Middle ground. Choose push when latency matters.

**Q44: Monolith vs Microservices — how do you decide?**
Start monolithic. Move to microservices when: team size makes coordination painful, different components need different scaling, you need independent deployments, or different components need different technology stacks. Red flags that you moved to microservices too early: more time on infrastructure than features, debugging takes hours across services, data consistency issues everywhere.

**Q45: Synchronous vs Asynchronous communication?**
Synchronous (REST/gRPC): when the caller needs the response to continue (get user data, check inventory). Asynchronous (queues/events): when the caller doesn't need to wait (send email, update analytics, process image). Rule of thumb: if the user is waiting for the response, use sync. If it can happen in the background, use async.

---

### Quick-Fire Round

**Q46: What is a single point of failure?** A component whose failure brings down the entire system. Eliminate by adding redundancy (multiple servers, DB replicas, multi-AZ).

**Q47: What is data partitioning?** Splitting data across multiple storage units. Horizontal partitioning (sharding) splits rows. Vertical partitioning splits columns into different tables.

**Q48: What is a bloom filter?** A space-efficient probabilistic data structure that tells you "possibly yes" or "definitely no." Used to avoid expensive lookups — e.g., check if a username is taken before querying the DB.

**Q49: What is the thundering herd problem?** When a cache entry expires and many simultaneous requests hit the database for the same data. Fix with locking (only one request queries DB) or stale-while-revalidate.

**Q50: What is leader election?** In a distributed system, one node is chosen as the "leader" to coordinate actions (writes in DB replication, task distribution). Algorithms: Raft, Paxos. Tools: ZooKeeper, etcd.

**Q51: What is back pressure?** When a system can't keep up with incoming requests, it slows down the producers rather than crashing. Like a sink — if water flows faster than it drains, you reduce the tap flow, not let the sink overflow.

**Q52: What is geo-replication?** Replicating data across geographically distant data centers. Provides disaster recovery and lower latency for global users. Challenge: cross-region replication lag.

**Q53: What is data denormalisation?** Duplicating data to avoid expensive joins. In a normalised DB, you join `orders` with `users` to get the username. Denormalised: store `user_name` directly in the orders table. Faster reads, harder writes (must update in multiple places).

**Q54: What is the write-ahead log (WAL)?** A sequential log where every change is written before it's applied to the database. Ensures durability — if the system crashes, the WAL is replayed to recover.

**Q55: What is a quorum?** The minimum number of nodes that must agree for an operation to be considered successful. In a cluster of 5 nodes, a quorum of 3 means at least 3 must confirm a write. Ensures consistency in distributed systems.

**Q56: What is graceful degradation?** When part of the system fails, the rest continues working with reduced functionality. If the recommendation service is down, show trending products instead. If the cache is down, serve from the database (slower but functional).

**Q57: What is a health check endpoint?** An API endpoint (usually /health) that returns the status of the service and its dependencies. Load balancers, orchestrators, and monitoring tools use it to determine if the service is alive and healthy.

**Q58: What is database connection pooling?** Maintaining a pool of reusable database connections instead of creating a new one for each request. Creating connections is expensive (TCP handshake, authentication). A pool of 20 connections can serve hundreds of requests per second by reusing them.

**Q59: What is the sidecar pattern?** Deploying a helper container alongside your main application container. The sidecar handles cross-cutting concerns (logging, monitoring, networking) without modifying the application. Like a motorcycle sidecar — attached to the main vehicle, adds functionality without changing the bike.

**Q60: What is blue-green deployment?** Running two identical production environments (blue and green). Traffic goes to blue. Deploy new version to green. Test green. Switch traffic from blue to green. If something goes wrong, switch back. Zero-downtime deployments with instant rollback.

---

### Architecture Pattern Questions

**Q61: What is the Strangler Fig pattern?**
A migration strategy for replacing a legacy system. Instead of rewriting everything at once, you gradually build new services that handle specific features, routing traffic from the old system to the new one feature by feature. Named after strangler fig trees that grow around a host tree and eventually replace it. Used when migrating from monolith to microservices.

**Q62: What is event sourcing? When would you use it?**
Instead of storing current state, store every event that led to the current state. Current state is derived by replaying events. Use when: you need a complete audit trail (banking, e-commerce), you need to reconstruct state at any point in time, or you need to derive new views from historical data. Drawback: complexity, storage growth, eventual consistency.

**Q63: What is the Bulkhead pattern?**
Isolating different parts of a system so that failure in one doesn't bring down others. Like bulkheads in a ship — flooding in one compartment doesn't sink the ship. Implementation: separate thread pools, separate databases, separate circuit breakers for different services. If the payment service thread pool is exhausted, the product listing service still works fine.

**Q64: What is the Outbox pattern?**
A pattern for reliably publishing events when a database transaction occurs. Instead of writing to the DB and publishing to a message queue (two separate systems that can fail independently), write the event to an "outbox" table in the same DB transaction. A separate process reads the outbox table and publishes to the queue. Guarantees that the event is published if and only if the transaction commits.

**Q65: Explain the BFF (Backend for Frontend) pattern.**
A separate backend service for each frontend type (web BFF, mobile BFF, admin BFF). Each BFF aggregates and transforms data from microservices into exactly what its frontend needs. The web BFF might return rich HTML-friendly data. The mobile BFF returns minimal payloads for bandwidth efficiency. Prevents a "one size fits all" API that serves no client well.

---

### Estimation Questions

**Q66: Estimate the storage needed for a service that stores 1 billion tweets.**
Average tweet: ~300 bytes (140 chars + metadata). 1 billion × 300 bytes = 300 GB for text. With media (images average 200KB, 10% of tweets have images): 100M × 200KB = 20 TB. With indexes and replication (3x): total ~60-100 TB. Daily growth: ~500M tweets/day × 300 bytes = 150 GB/day for text alone.

**Q67: How many servers do you need for 10,000 requests per second?**
A typical Node.js server handles ~1,000-2,000 simple API requests per second. For 10,000 RPS: 5-10 servers behind a load balancer. But it depends heavily on: request complexity (simple reads vs heavy computation), database query time, external API calls, and response size. Always benchmark your actual application.

**Q68: Estimate the bandwidth needed for a video streaming service with 10 million daily active users.**
If each user watches 30 minutes/day at 5 Mbps (HD quality): 10M × 30 min × 60 sec × 5 Mbps = 90 Pbps total per day. Peak concurrent viewers might be 1M at any time: 1M × 5 Mbps = 5 Tbps peak bandwidth. This is why CDNs are essential — no single data center can handle this.

---

### Final Rapid Fire

**Q69: What does "five nines" availability mean?** 99.999% uptime = ~5 minutes of downtime per year.

**Q70: What is a reverse index?** Maps words to the documents that contain them. Foundation of search engines and Elasticsearch.

**Q71: What is tail latency?** The latency at high percentiles (p99, p999). Even if average latency is 50ms, p99 might be 2 seconds. Important because at scale, many users experience tail latency.

**Q72: What is a gossip protocol?** Nodes share information by randomly "gossiping" with each other, like rumors spreading in a group. Used for failure detection and membership in distributed systems (Cassandra uses it).

**Q73: What is a content-addressable storage?** Data is retrieved based on its content hash, not its location. Same content always produces the same hash. Used in: Git, IPFS, Docker image layers. Enables deduplication.

**Q74: What is the difference between active-passive and active-active replication?** Active-passive: one node handles all traffic, the other is standby for failover. Active-active: both nodes handle traffic simultaneously. Active-active gives higher throughput but requires conflict resolution.

**Q75: What is database connection pooling?** Maintaining pre-created database connections that are reused across requests instead of creating new ones per request. Creating a connection takes ~50ms. Reusing from pool takes ~1ms.

**Q76: What is a time-series database?** A database optimised for timestamped data — metrics, IoT sensor data, stock prices. Examples: InfluxDB, TimescaleDB. Optimisations: time-based partitioning, compression, downsampling.

**Q77: What is the ambassador pattern?** A helper service that handles network communication on behalf of the application — retries, circuit breaking, TLS. Like the sidecar pattern but specifically for outgoing network calls.

**Q78: What is idempotency and why is it critical in distributed systems?** An operation that produces the same result no matter how many times it's executed. Critical because in distributed systems, messages can be delivered multiple times (network retries, queue redelivery). Without idempotency, retrying a payment could charge the user twice.

**Q79: What is a distributed lock?** A lock mechanism that works across multiple servers. When only one server should process a task at a time (e.g., scheduled job), they compete for a distributed lock (often stored in Redis or ZooKeeper). The winner holds the lock, others skip.

**Q80: What is the difference between a proxy and a gateway?** A proxy forwards requests transparently — the client may not know it exists. A gateway acts as a single entry point that adds value — authentication, routing, aggregation, transformation. All gateways are a form of proxy, but not all proxies are gateways.

---

> **Final Advice for System Design Interviews:**
>
> Interviewers aren't looking for the "perfect" architecture. They want to see your thinking process: can you identify the right bottleneck, propose a solution, and discuss trade-offs? Always start with requirements and scale. Mention what you'd monitor. Discuss failure scenarios. And remember — every architectural decision is a trade-off. The best answer isn't "use Kafka" — it's "I'd use Kafka here because we need event replay and high throughput, but RabbitMQ would be simpler if we only need task queuing."
