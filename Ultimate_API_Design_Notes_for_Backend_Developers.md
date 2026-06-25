# Ultimate API Design Notes — From Zero to Pro

> **Who is this for?** Backend developers who want to deeply understand API design — REST, GraphQL, authentication, versioning, error handling, pagination, rate limiting, and everything in between. After reading this, you'll design APIs that other developers love using, and you'll crush API-related interview questions.

---

## Table of Contents

1. [What Is an API and Why Should You Care?](#1-what-is-an-api-and-why-should-you-care)
2. [The Anatomy of an HTTP Request & Response](#2-the-anatomy-of-an-http-request--response)
3. [REST — The Foundation](#3-rest--the-foundation)
4. [Designing RESTful URLs — The Art of Naming](#4-designing-restful-urls--the-art-of-naming)
5. [HTTP Methods — Choosing the Right Verb](#5-http-methods--choosing-the-right-verb)
6. [HTTP Status Codes — Speaking the Right Language](#6-http-status-codes--speaking-the-right-language)
7. [Request & Response Design](#7-request--response-design)
8. [Pagination — Handling Large Data Sets](#8-pagination--handling-large-data-sets)
9. [Filtering, Sorting & Searching](#9-filtering-sorting--searching)
10. [API Versioning — Evolving Without Breaking](#10-api-versioning--evolving-without-breaking)
11. [Authentication & Authorization](#11-authentication--authorization)
12. [Error Handling — The Make or Break of Developer Experience](#12-error-handling--the-make-or-break-of-developer-experience)
13. [Rate Limiting & Throttling](#13-rate-limiting--throttling)
14. [HATEOAS & Richardson Maturity Model](#14-hateoas--richardson-maturity-model)
15. [Idempotency — Safe Retries](#15-idempotency--safe-retries)
16. [Caching in APIs](#16-caching-in-apis)
17. [File Upload & Download](#17-file-upload--download)
18. [Webhooks — APIs in Reverse](#18-webhooks--apis-in-reverse)
19. [Real-Time APIs — WebSockets & SSE](#19-real-time-apis--websockets--sse)
20. [GraphQL — The Complete Guide](#20-graphql--the-complete-guide)
21. [REST vs GraphQL — The Honest Comparison](#21-rest-vs-graphql--the-honest-comparison)
22. [API Documentation — If It's Not Documented, It Doesn't Exist](#22-api-documentation--if-its-not-documented-it-doesnt-exist)
23. [API Security Checklist](#23-api-security-checklist)
24. [API Design Patterns in the Real World](#24-api-design-patterns-in-the-real-world)
25. [Interview Questions (80+ Questions)](#25-interview-questions-80-questions)

---

## 1. What Is an API and Why Should You Care?

### The Restaurant Analogy

Imagine you're at a restaurant. You (the **client**) sit at a table. The kitchen (the **server/database**) prepares food. But you never walk into the kitchen yourself. Instead, a **waiter** takes your order, brings it to the kitchen, and comes back with your food.

**The waiter is the API.**

- **You** = the frontend app, mobile app, or another service
- **The menu** = the API documentation
- **Your order** = the API request
- **The food** = the API response
- **The waiter** = the API itself
- **The kitchen** = the server and database

An API (Application Programming Interface) is a contract between two pieces of software. It says: "If you send me *this* kind of request, I'll give you *this* kind of response."

### Why APIs Matter for Backend Developers

As a backend developer, building APIs is likely 70-80% of your job. You're building the waiter, deciding what's on the menu, and making sure the kitchen can handle the orders. A well-designed API means:

- Frontend developers can build UIs without constantly asking you questions
- Mobile developers can integrate without workarounds
- Third-party developers can extend your platform
- Your team can move faster because the contract is clear
- Your system can evolve without breaking existing clients

---

## 2. The Anatomy of an HTTP Request & Response

Before designing APIs, you need to understand HTTP deeply. Every REST API rides on top of HTTP.

### HTTP Request — What the Client Sends

```
POST /api/v1/users HTTP/1.1          ← Request Line (Method + Path + Protocol)
Host: api.myapp.com                   ← Headers start here
Content-Type: application/json
Authorization: Bearer eyJhbGciOi...
Accept: application/json
User-Agent: MyApp/2.0

{                                     ← Body (payload)
  "name": "Ravi Kumar",
  "email": "ravi@example.com",
  "role": "developer"
}
```

**Five parts of every request:**

| Part | What It Is | Analogy |
|------|-----------|---------|
| **Method** (POST) | What action you want to do | "I'd like to ORDER something" |
| **URL** (/api/v1/users) | Which resource you're acting on | "From the MAIN COURSE section" |
| **Headers** | Metadata about the request | "I'm vegetarian, I speak Hindi, here's my membership card" |
| **Query Parameters** (?page=2&limit=10) | Filters and options | "Page 2 of the menu, show me 10 items" |
| **Body** (JSON payload) | The actual data you're sending | "Here's exactly what I want" |

### HTTP Response — What the Server Sends Back

```
HTTP/1.1 201 Created                  ← Status Line (Protocol + Status Code + Reason)
Content-Type: application/json
Location: /api/v1/users/42
X-Request-Id: abc-123-def
X-RateLimit-Remaining: 98
Cache-Control: no-cache

{                                     ← Response Body
  "id": 42,
  "name": "Ravi Kumar",
  "email": "ravi@example.com",
  "role": "developer",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

### Common Request Headers You Should Know

| Header | Purpose | Example |
|--------|---------|---------|
| `Content-Type` | Format of the request body | `application/json` |
| `Accept` | What format you want the response in | `application/json` |
| `Authorization` | Your credentials | `Bearer eyJhbGci...` |
| `User-Agent` | Who's making the request | `MyApp/2.0` |
| `X-Request-Id` | Unique request identifier (for tracing) | `uuid-here` |
| `If-None-Match` | For caching (ETags) | `"abc123"` |
| `Accept-Language` | Preferred language | `en-US, hi-IN` |

### Common Response Headers You Should Know

| Header | Purpose | Example |
|--------|---------|---------|
| `Content-Type` | Format of the response body | `application/json` |
| `Location` | URL of newly created resource | `/api/v1/users/42` |
| `X-RateLimit-Limit` | Max requests allowed | `100` |
| `X-RateLimit-Remaining` | Requests remaining | `98` |
| `Retry-After` | When to retry after rate limiting | `30` (seconds) |
| `Cache-Control` | Caching instructions | `max-age=3600` |
| `ETag` | Version identifier for caching | `"abc123"` |

---

## 3. REST — The Foundation

### What REST Actually Means

REST stands for **RE**presentational **S**tate **T**ransfer. It was defined by Roy Fielding in his PhD dissertation in 2000. It's not a protocol, not a standard, not a library — it's an **architectural style** (a set of constraints/rules).

**Analogy:** REST is like the rules of grammar in English. You can technically write "me go store yesterday" and people might understand you. But following grammar rules ("I went to the store yesterday") makes communication clearer and more predictable. REST is grammar for APIs.

### The 6 Constraints of REST

#### 1. Client-Server Separation
The client (frontend) and server (backend) are independent. The client doesn't know how data is stored. The server doesn't know how data is displayed. They communicate only through the API.

**Why it matters:** Your React frontend and your Node.js backend can evolve independently. Change the database from MongoDB to PostgreSQL? The frontend doesn't care. Rebuild the frontend in Vue? The backend doesn't care.

#### 2. Statelessness
Every request from the client must contain ALL the information the server needs. The server doesn't remember anything about previous requests.

**Analogy:** Imagine calling a help desk where they have no memory of your previous calls. Every time you call, you say: "My name is Ravi, my account number is 12345, and here's my problem." Annoying for you, but the help desk can be run by anyone, anywhere, without needing to remember your history.

**Why it matters:** Any server in your cluster can handle any request. This makes scaling trivial — just add more servers behind a load balancer.

```
# BAD (stateful) — server remembers "current user" from login
GET /api/my-orders          ← Who is "my"? Depends on server session state.

# GOOD (stateless) — everything needed is in the request
GET /api/users/42/orders
Authorization: Bearer eyJhbGci...    ← Token contains user identity
```

#### 3. Cacheability
Responses must say whether they can be cached. If yes, clients and intermediaries (CDNs) can reuse the response for identical requests.

```
# Cacheable response
Cache-Control: public, max-age=3600   ← Cache for 1 hour
ETag: "v1-abc123"

# Non-cacheable response
Cache-Control: no-store, no-cache
```

#### 4. Uniform Interface
Every resource is accessed the same way: through URLs, HTTP methods, and standard media types. You don't need different protocols for users, orders, and products — they all follow the same pattern.

#### 5. Layered System
The client doesn't know if it's talking directly to the server, or to an intermediary (load balancer, CDN, API gateway). Each layer only knows about the layer it directly interacts with.

```
Client → CDN → Load Balancer → API Gateway → Your API Server → Database
```

#### 6. Code on Demand (Optional)
The server can send executable code to the client (like JavaScript). This is the only optional constraint and is rarely discussed in API design.

### What Is a Resource?

In REST, everything is a **resource**. A resource is any concept that can be named and addressed.

| Resource | URL | What It Represents |
|----------|-----|-------------------|
| A single user | `/users/42` | The user with ID 42 |
| All users | `/users` | The collection of all users |
| A user's orders | `/users/42/orders` | All orders belonging to user 42 |
| A specific order | `/orders/789` | The order with ID 789 |
| A product image | `/products/5/image` | The image for product 5 |

**Think of resources as nouns, not verbs.** The URL tells you *what* you're working with. The HTTP method tells you *what you're doing* with it.

---

## 4. Designing RESTful URLs — The Art of Naming

### Golden Rules

#### Rule 1: Use Nouns, Not Verbs

```
# GOOD — nouns (resources)
GET    /users
POST   /users
GET    /users/42
PUT    /users/42
DELETE /users/42

# BAD — verbs (actions)
GET    /getUsers
POST   /createUser
GET    /getUserById?id=42
POST   /updateUser
POST   /deleteUser
```

The HTTP method IS the verb. Adding another verb is redundant — it's like saying "I'm running quickly fast."

#### Rule 2: Use Plural Nouns

```
# GOOD — plural and consistent
/users
/users/42
/products
/products/5

# BAD — mixing singular and plural
/user
/user/42
/getProduct
```

Even for a single resource, the URL path uses plural because it represents the collection. `/users/42` means "from the users collection, get number 42."

#### Rule 3: Use Kebab-Case for Multi-Word Resources

```
# GOOD
/order-items
/user-profiles
/payment-methods

# BAD
/orderItems        ← camelCase
/order_items       ← snake_case
/OrderItems        ← PascalCase
```

URLs are case-sensitive, and kebab-case is the web standard.

#### Rule 4: Nest Resources to Show Relationships

```
# A user's orders
GET /users/42/orders

# A specific order for a user
GET /users/42/orders/789

# Items in an order
GET /orders/789/items

# A specific item in an order
GET /orders/789/items/3
```

**But don't nest too deep** — more than 2 levels gets confusing:

```
# TOO DEEP — hard to read and maintain
GET /users/42/orders/789/items/3/reviews/12

# BETTER — flatten after 2 levels
GET /order-items/3/reviews/12
```

**Rule of thumb:** If a resource can exist independently (like an order item has its own ID), give it its own top-level endpoint.

#### Rule 5: Use Query Parameters for Filtering, Not URL Paths

```
# GOOD — filters as query parameters
GET /products?category=electronics&min_price=100&in_stock=true

# BAD — filters baked into the URL path
GET /products/electronics/price-above-100/in-stock
```

#### Real-World URL Design Examples

**E-Commerce API:**
```
GET    /products                    # List all products
GET    /products/42                 # Get product 42
POST   /products                    # Create a product
PUT    /products/42                 # Update product 42
DELETE /products/42                 # Delete product 42

GET    /products/42/reviews         # Get reviews for product 42
POST   /products/42/reviews         # Add a review to product 42

GET    /cart                        # Get current user's cart
POST   /cart/items                  # Add item to cart
DELETE /cart/items/15               # Remove item 15 from cart

POST   /orders                     # Place an order
GET    /orders/789                  # Get order details
GET    /orders/789/tracking         # Get tracking info
```

**Social Media API:**
```
GET    /users/42                    # Get user profile
GET    /users/42/posts              # Get user's posts
GET    /users/42/followers          # Get user's followers
GET    /users/42/following          # Get users they follow

POST   /posts                       # Create a post
GET    /posts/100                    # Get a post
POST   /posts/100/likes             # Like a post
DELETE /posts/100/likes             # Unlike a post
POST   /posts/100/comments          # Comment on a post

GET    /feed                         # Get authenticated user's feed
GET    /search/users?q=ravi          # Search users
GET    /search/posts?q=docker        # Search posts
```

### Actions That Don't Fit CRUD

Sometimes you need actions that aren't simple create/read/update/delete. Here's how to handle them:

```
# Approach 1: Treat the action as a sub-resource
POST /orders/789/cancel            # Cancel an order
POST /users/42/deactivate          # Deactivate a user
POST /payments/123/refund          # Refund a payment

# Approach 2: Use a status update (more RESTful)
PATCH /orders/789
{ "status": "cancelled" }

# Approach 3: Use a dedicated action resource
POST /password-reset-requests      # Request a password reset
POST /email-verifications          # Request email verification
```

---

## 5. HTTP Methods — Choosing the Right Verb

### The Main Five

| Method | Purpose | Analogy | Has Body? | Idempotent? | Safe? |
|--------|---------|---------|-----------|-------------|-------|
| `GET` | Read data | Looking at the menu | No | Yes | Yes |
| `POST` | Create new data | Placing an order | Yes | No | No |
| `PUT` | Replace entire resource | Rewriting your entire order | Yes | Yes | No |
| `PATCH` | Update part of a resource | Changing one item in your order | Yes | Yes* | No |
| `DELETE` | Remove data | Cancelling your order | Optional | Yes | No |

**Safe** = Doesn't change anything on the server. GET is like looking at a book — you don't alter it by reading.

**Idempotent** = Doing it multiple times has the same effect as doing it once. DELETE is idempotent: deleting user 42 ten times still results in user 42 being deleted. POST is NOT idempotent: posting an order ten times creates ten orders.

### GET — Read

```bash
# Get all users (list)
GET /api/v1/users?page=1&limit=20

# Get a specific user
GET /api/v1/users/42

# Get with filters
GET /api/v1/products?category=electronics&sort=-price&fields=name,price
```

**Rules:**
- Never use GET to modify data
- Never put sensitive data in the URL (it gets logged, cached, shared)
- Always support filtering, sorting, pagination for list endpoints
- Should be cacheable

### POST — Create

```bash
# Create a new user
POST /api/v1/users
Content-Type: application/json

{
  "name": "Ravi Kumar",
  "email": "ravi@example.com"
}

# Response: 201 Created
{
  "id": 42,
  "name": "Ravi Kumar",
  "email": "ravi@example.com",
  "createdAt": "2024-01-15T10:30:00Z"
}
```

**Rules:**
- Return `201 Created` with the new resource
- Include a `Location` header pointing to the new resource
- The server assigns the ID, not the client
- Not idempotent — calling twice creates two resources

### PUT — Full Replace

```bash
# Replace user 42 entirely
PUT /api/v1/users/42
Content-Type: application/json

{
  "name": "Ravi Kumar",
  "email": "ravi.new@example.com",
  "role": "senior-developer"
}
```

**Rules:**
- Send the ENTIRE resource, not just changed fields
- If you omit a field, it should be set to null/default
- Idempotent — sending the same PUT ten times has the same result
- Return `200 OK` with the updated resource

**Analogy:** PUT is like writing a letter. You don't send just the corrected paragraph — you send the entire new version of the letter.

### PATCH — Partial Update

```bash
# Update only the email
PATCH /api/v1/users/42
Content-Type: application/json

{
  "email": "ravi.new@example.com"
}
```

**Rules:**
- Send ONLY the fields you want to change
- Unmentioned fields remain unchanged
- Return `200 OK` with the full updated resource
- More practical than PUT for most real-world use cases

**Analogy:** PATCH is like editing a Google Doc — you change one sentence, not the entire document.

### DELETE — Remove

```bash
# Delete user 42
DELETE /api/v1/users/42

# Response: 204 No Content (nothing to return)
```

**Rules:**
- Return `204 No Content` (preferred) or `200 OK` with a confirmation message
- Idempotent — deleting an already-deleted resource returns `404 Not Found` or `204`
- Consider soft deletes in production (set a `deletedAt` timestamp instead of actually removing)

### PUT vs PATCH — When to Use Which

```json
// Original user
{
  "id": 42,
  "name": "Ravi Kumar",
  "email": "ravi@example.com",
  "role": "developer",
  "bio": "I love coding"
}
```

**PUT** (full replace) — you must send everything:
```json
PUT /users/42
{
  "name": "Ravi Kumar",
  "email": "ravi.new@example.com",
  "role": "developer",
  "bio": "I love coding"
}
// If you forget "bio", it becomes null
```

**PATCH** (partial update) — send only what changed:
```json
PATCH /users/42
{
  "email": "ravi.new@example.com"
}
// Everything else stays the same
```

**In practice:** Most APIs use PATCH for updates because it's simpler and less error-prone. PUT is technically more RESTful but impractical when resources have many fields.

---

## 6. HTTP Status Codes — Speaking the Right Language

### The Five Families

| Range | Category | Analogy |
|-------|----------|---------|
| **1xx** | Informational | "Hold on, I'm working on it..." |
| **2xx** | Success | "Here you go, all good!" |
| **3xx** | Redirection | "It's not here anymore, try over there" |
| **4xx** | Client Error | "You messed up your request" |
| **5xx** | Server Error | "I messed up, sorry" |

### Success Codes (2xx) — Use the Right One

| Code | Name | When to Use | Example |
|------|------|-------------|---------|
| `200` | OK | General success, data returned | GET request, PUT/PATCH update |
| `201` | Created | New resource created | POST that creates a user |
| `202` | Accepted | Request accepted, processing later | Async job submitted |
| `204` | No Content | Success, but nothing to return | DELETE, or PUT with no response body |

```javascript
// Express.js examples
app.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  res.status(200).json(user);              // 200: here's the data
});

app.post('/users', (req, res) => {
  const user = createUser(req.body);
  res.status(201).json(user);              // 201: created successfully
});

app.delete('/users/:id', (req, res) => {
  deleteUser(req.params.id);
  res.status(204).send();                  // 204: deleted, nothing to return
});

app.post('/reports/generate', (req, res) => {
  queueReportJob(req.body);
  res.status(202).json({                   // 202: accepted, will process later
    message: "Report generation started",
    jobId: "job-789",
    statusUrl: "/api/v1/jobs/job-789"
  });
});
```

### Client Error Codes (4xx) — The Client Did Something Wrong

| Code | Name | When to Use | Example |
|------|------|-------------|---------|
| `400` | Bad Request | Invalid input, malformed JSON | Missing required field, wrong data type |
| `401` | Unauthorized | Not authenticated (who are you?) | Missing or expired token |
| `403` | Forbidden | Authenticated but not allowed | User trying to access admin endpoint |
| `404` | Not Found | Resource doesn't exist | GET /users/99999 when no such user |
| `405` | Method Not Allowed | Right URL, wrong method | PUT on a read-only endpoint |
| `409` | Conflict | Action conflicts with current state | Creating user with duplicate email |
| `422` | Unprocessable Entity | Valid JSON but invalid data | Email format is wrong, age is -5 |
| `429` | Too Many Requests | Rate limit exceeded | More than 100 requests per minute |

**The 401 vs 403 Confusion — Cleared Up Forever:**

- **401 Unauthorized** = "I don't know who you are." You haven't logged in, your token expired, or your credentials are wrong. *Solution:* Log in or refresh your token.
- **403 Forbidden** = "I know who you are, but you can't do this." You're logged in as a regular user trying to access admin features. *Solution:* Get more permissions.

**Analogy:** 401 is a bouncer asking for your ID. 403 is a bouncer who checked your ID but says "VIP area only."

### Server Error Codes (5xx) — The Server Messed Up

| Code | Name | When to Use |
|------|------|-------------|
| `500` | Internal Server Error | Unhandled exception, unexpected bug |
| `502` | Bad Gateway | Upstream service returned an invalid response |
| `503` | Service Unavailable | Server is overloaded or under maintenance |
| `504` | Gateway Timeout | Upstream service didn't respond in time |

**Rule:** 5xx errors should NEVER expose internal details (stack traces, database errors, file paths) to the client. Log them internally, return a generic message externally.

```javascript
// BAD — leaking internals
res.status(500).json({
  error: "MongoServerError: E11000 duplicate key error collection: mydb.users index: email_1 dup key: { email: \"ravi@example.com\" }"
});

// GOOD — user-friendly + logged internally
logger.error('Duplicate email error', { email: req.body.email, stack: err.stack });
res.status(409).json({
  error: {
    code: "DUPLICATE_EMAIL",
    message: "A user with this email already exists."
  }
});
```

---

## 7. Request & Response Design

### Request Body Best Practices

#### Use camelCase for JSON Keys (Most Common Convention)
```json
{
  "firstName": "Ravi",
  "lastName": "Kumar",
  "emailAddress": "ravi@example.com",
  "dateOfBirth": "1995-03-15",
  "isActive": true
}
```

Some APIs use snake_case (like GitHub, Stripe, Twitter). Pick one convention and stick with it across your entire API.

#### Accept Only What You Need
```javascript
// BAD — accepting the entire body without validation
app.post('/users', (req, res) => {
  const user = await User.create(req.body);  // Dangerous! Client could send isAdmin: true
});

// GOOD — whitelist specific fields
app.post('/users', (req, res) => {
  const { name, email, password } = req.body;
  const user = await User.create({ name, email, password });
});
```

#### Validate Everything
```javascript
// Using a validation library (like Joi or Zod)
const createUserSchema = z.object({
  name: z.string().min(2).max(100),
  email: z.string().email(),
  password: z.string().min(8).max(128),
  role: z.enum(['user', 'admin']).default('user'),
  age: z.number().int().min(13).max(150).optional(),
});

app.post('/users', (req, res) => {
  const result = createUserSchema.safeParse(req.body);
  if (!result.success) {
    return res.status(422).json({
      error: {
        code: "VALIDATION_ERROR",
        message: "Invalid input",
        details: result.error.issues
      }
    });
  }
  // proceed with result.data
});
```

### Response Body Best Practices

#### Consistent Envelope (Wrapper) Pattern

**Approach 1: Direct Response (simpler, used by GitHub, Stripe)**
```json
// Single resource
{
  "id": 42,
  "name": "Ravi Kumar",
  "email": "ravi@example.com"
}

// Collection
[
  { "id": 42, "name": "Ravi Kumar" },
  { "id": 43, "name": "Priya Singh" }
]
```

**Approach 2: Envelope/Wrapper (used by Google, many internal APIs)**
```json
// Single resource
{
  "data": {
    "id": 42,
    "name": "Ravi Kumar",
    "email": "ravi@example.com"
  },
  "meta": {
    "requestId": "abc-123"
  }
}

// Collection
{
  "data": [
    { "id": 42, "name": "Ravi Kumar" },
    { "id": 43, "name": "Priya Singh" }
  ],
  "meta": {
    "totalCount": 150,
    "page": 1,
    "perPage": 20,
    "totalPages": 8
  }
}
```

**Pick one pattern and use it everywhere.** Mixing is confusing — "sometimes the user object is the response, sometimes it's inside a `data` wrapper."

#### Always Return the Resource After Create/Update
```javascript
// GOOD — client gets the full resource back (with server-generated fields)
app.post('/users', async (req, res) => {
  const user = await User.create(req.body);
  res.status(201).json(user);  // includes id, createdAt, defaults
});

// BAD — client has to make another GET request
app.post('/users', async (req, res) => {
  await User.create(req.body);
  res.status(201).json({ message: "User created" });  // Where's the ID? The createdAt?
});
```

#### Use ISO 8601 for Dates — Always

```json
{
  "createdAt": "2024-01-15T10:30:00Z",
  "updatedAt": "2024-06-20T14:45:30Z",
  "expiresAt": "2025-01-15T00:00:00Z"
}
```

Never use Unix timestamps, "January 15, 2024", or any other format in your API responses. ISO 8601 is unambiguous, timezone-aware, sortable, and universally parseable.

#### Null vs Absent Fields

```json
// Option A: Include null fields (explicit — client knows the field exists but has no value)
{
  "id": 42,
  "name": "Ravi Kumar",
  "bio": null,
  "avatarUrl": null
}

// Option B: Omit null fields (leaner — client checks for field presence)
{
  "id": 42,
  "name": "Ravi Kumar"
}
```

Pick a convention. Option A is more explicit and easier for frontend developers. Option B produces smaller payloads. Most modern APIs go with Option A.

---

## 8. Pagination — Handling Large Data Sets

### Why Paginate?

Imagine a library with 10 million books. If you ask "give me all the books," the librarian would spend years loading carts. Instead, you say "give me books 1-20 from the fiction shelf." That's pagination.

Without pagination, your API returns all records, your database crashes, your network chokes, and your client runs out of memory.

### Strategy 1: Offset-Based Pagination (Most Common)

```
GET /api/v1/products?page=2&limit=20
```

```json
{
  "data": [ ... 20 products ... ],
  "meta": {
    "currentPage": 2,
    "perPage": 20,
    "totalItems": 543,
    "totalPages": 28,
    "hasNextPage": true,
    "hasPrevPage": true
  }
}
```

**How it works:** `OFFSET = (page - 1) * limit`. For page 2 with limit 20: `SELECT * FROM products LIMIT 20 OFFSET 20`.

**Pros:** Simple, easy to jump to any page, easy to show "Page 2 of 28."
**Cons:** Slow on large datasets (database scans through all skipped rows), inconsistent if data is added/removed between pages (you might see duplicates or miss items).

**Analogy:** Like reading a book by page number. If someone inserts a page before page 50, everything shifts. When you go to page 50, you're now on the wrong content.

### Strategy 2: Cursor-Based Pagination (Better for Large/Real-Time Data)

```
GET /api/v1/posts?limit=20&cursor=eyJpZCI6MTAwfQ==
```

```json
{
  "data": [ ... 20 posts ... ],
  "meta": {
    "nextCursor": "eyJpZCI6MTIwfQ==",
    "prevCursor": "eyJpZCI6MTAxfQ==",
    "hasMore": true
  }
}
```

**How it works:** Instead of "give me page 5," you say "give me 20 items after this specific point." The cursor is typically a Base64-encoded ID or timestamp.

```sql
-- Instead of OFFSET (slow)
SELECT * FROM posts WHERE id > 100 ORDER BY id ASC LIMIT 20;
```

**Pros:** Fast even on millions of rows (uses index), consistent results even when data changes, works perfectly for infinite scroll.
**Cons:** Can't jump to "page 5" directly, harder to implement.

**Analogy:** Like a bookmark. You don't need to know the page number — you continue exactly where you left off, even if someone added pages before your bookmark.

**When to use which:**
- **Offset:** Admin panels, dashboards, or any UI with page numbers
- **Cursor:** Social feeds, chat messages, real-time data, large datasets, public APIs

### Strategy 3: Keyset Pagination (Best Performance)

A simpler variant of cursor-based pagination using actual column values:

```
GET /api/v1/products?limit=20&created_before=2024-01-15T10:30:00Z
```

This translates directly to an indexed query: `WHERE created_at < '2024-01-15T10:30:00Z' ORDER BY created_at DESC LIMIT 20`.

---

## 9. Filtering, Sorting & Searching

### Filtering

Let clients narrow down results using query parameters:

```
# Simple equality filters
GET /products?category=electronics
GET /products?status=active&brand=apple

# Range filters
GET /products?min_price=100&max_price=500
GET /orders?created_after=2024-01-01&created_before=2024-06-30

# Multiple values (OR condition)
GET /products?category=electronics,clothing,books
# OR
GET /products?category[]=electronics&category[]=clothing

# Boolean filters
GET /products?in_stock=true
GET /users?verified=true
```

### Sorting

```
# Sort by single field
GET /products?sort=price          # ascending (default)
GET /products?sort=-price         # descending (prefix with -)

# Sort by multiple fields
GET /products?sort=-rating,price  # highest rated first, then cheapest

# Alternative syntax (more explicit)
GET /products?sort_by=price&sort_order=desc
```

**Convention:** The `-` prefix for descending is widely used (GitHub, JSON:API spec). It's concise and readable.

### Searching

```
# Full-text search
GET /products?q=wireless+headphones

# Search specific fields
GET /users?search=ravi&search_fields=name,email

# Combined with filters
GET /products?q=headphones&category=electronics&min_price=50&sort=-rating
```

### Field Selection (Sparse Fieldsets)

Let clients choose which fields to return — saves bandwidth:

```
# Only return name and price
GET /products?fields=name,price

# Return specific nested fields
GET /users/42?fields=name,email,address.city
```

**Real example:** Mobile clients on slow networks might only need `name` and `thumbnail`, while desktop clients want everything.

---

## 10. API Versioning — Evolving Without Breaking

### Why Version?

Your API is a contract. Clients (mobile apps, third-party integrations, frontend apps) depend on the shape of your requests and responses. If you rename a field from `userName` to `displayName`, every client breaks.

**Analogy:** Versioning is like software releases. When Microsoft releases Windows 11, Windows 10 doesn't stop working. Both versions coexist while users migrate.

### Strategy 1: URL Path Versioning (Most Common)

```
GET /api/v1/users
GET /api/v2/users
```

**Pros:** Dead simple, obvious, easy to route, easy to test.
**Cons:** URL changes, technically not RESTful (the resource is the same, only the representation changed).

**Used by:** Twitter, Stripe, Google Maps, Facebook.

### Strategy 2: Header Versioning

```
GET /api/users
Accept: application/vnd.myapi.v2+json
```

Or with a custom header:
```
GET /api/users
X-API-Version: 2
```

**Pros:** Clean URLs, more RESTful.
**Cons:** Harder to test (can't just change the URL in a browser), easy to forget.

**Used by:** GitHub (Accept header).

### Strategy 3: Query Parameter Versioning

```
GET /api/users?version=2
```

**Pros:** Easy to switch versions.
**Cons:** Easy to forget, pollutes the query string, can interfere with caching.

### Which Should You Use?

**URL path versioning (/v1/users)** — unless you have a strong reason not to. It's the most widely used, easiest to understand, easiest to implement, and easiest to test. Most companies use it.

### When to Create a New Version

**DO version when:**
- Removing a field from the response
- Renaming a field
- Changing a field's data type
- Changing the structure of the response
- Removing an endpoint

**DON'T version when:**
- Adding a new optional field to the response
- Adding a new endpoint
- Fixing a bug
- Adding a new optional query parameter

**Rule of thumb:** Adding is safe, removing/changing is breaking.

### Deprecation Strategy

```
# Response header to warn clients
Sunset: Sat, 01 Jan 2025 00:00:00 GMT
Deprecation: true
Link: <https://api.myapp.com/v2/docs>; rel="successor-version"
```

Provide a migration guide, give at least 6-12 months notice, monitor v1 usage, and don't turn it off until usage is near zero.

---

## 11. Authentication & Authorization

### Authentication vs Authorization — Know the Difference

| | Authentication (AuthN) | Authorization (AuthZ) |
|-|----------------------|---------------------|
| **Question** | Who are you? | What can you do? |
| **Analogy** | Showing your ID at the door | Checking if your ticket allows VIP access |
| **When** | Before anything else | After authentication |
| **Failure** | 401 Unauthorized | 403 Forbidden |

### Strategy 1: API Keys

The simplest form of authentication. A long random string that identifies the client.

```
GET /api/v1/weather?city=delhi
X-API-Key: ak_live_7f8g9h0j1k2l3m4n5o6p
```

**Pros:** Simple, easy to implement, good for server-to-server communication.
**Cons:** No user identity (identifies the app, not the user), if leaked, full access until rotated, typically sent in every request.

**Use for:** Third-party API access, service-to-service communication, public APIs with rate limiting.

**Analogy:** An API key is like a building access card. It gets you in the door, but it doesn't say WHO you are — just that you're allowed in.

### Strategy 2: JWT (JSON Web Tokens) — The Most Common

A JWT is a self-contained token that holds user information. It has three parts separated by dots:

```
eyJhbGciOiJIUzI1NiJ9.eyJ1c2VySWQiOjQyLCJyb2xlIjoiYWRtaW4iLCJleHAiOjE3MDU0MjAwMDB9.signature_here
```

**Decoded:**

```
HEADER (algorithm info)
{
  "alg": "HS256",
  "typ": "JWT"
}

PAYLOAD (claims — the actual data)
{
  "userId": 42,
  "role": "admin",
  "email": "ravi@example.com",
  "exp": 1705420000,
  "iat": 1705333600
}

SIGNATURE (verification)
HMACSHA256(base64(header) + "." + base64(payload), secret_key)
```

**How it works:**

```
1. Client sends credentials:
   POST /auth/login { email, password }

2. Server validates, creates JWT, sends it back:
   { "accessToken": "eyJ...", "refreshToken": "dGhpcyBpcy..." }

3. Client sends JWT in every subsequent request:
   GET /api/v1/users/me
   Authorization: Bearer eyJhbGciOi...

4. Server verifies the signature (no database lookup needed!)
   → If valid, extract userId from the payload
   → If expired/tampered, return 401
```

**Why JWTs are popular for APIs:**
- **Stateless** — the server doesn't need to store sessions. The token itself contains all the info.
- **Scalable** — any server can verify the token without hitting a shared session store.
- **Cross-domain** — works across different services and domains.

**Critical rules:**
- Never store sensitive data in the payload (it's Base64-encoded, not encrypted — anyone can decode it)
- Always set an expiration (`exp` claim) — short-lived (15 min to 1 hour for access tokens)
- Use HTTPS always — tokens in transit can be stolen
- Store tokens securely on the client (HttpOnly cookies > localStorage)

### Access Token + Refresh Token Pattern

```
Access Token:  Short-lived (15 min). Used for API requests.
Refresh Token: Long-lived (7-30 days). Used only to get new access tokens.

Flow:
1. Login → get access token (15 min) + refresh token (7 days)
2. Use access token for API requests
3. Access token expires → use refresh token to get a new access token
4. Refresh token expires → user must login again
```

```
POST /auth/refresh
{
  "refreshToken": "dGhpcyBpcy..."
}

Response:
{
  "accessToken": "new-eyJ...",
  "expiresIn": 900
}
```

**Why two tokens?** If an access token is stolen, the attacker has only 15 minutes. The refresh token is stored more securely and only sent to one endpoint (/auth/refresh), reducing exposure.

**Analogy:** The access token is a day pass at a theme park (short-lived, used constantly). The refresh token is the membership card in your wallet (long-lived, only shown at the registration desk to get a new day pass).

### Strategy 3: OAuth 2.0 — Delegated Authorization

OAuth is NOT authentication — it's authorization. It lets users grant limited access to their data on one service to another service, without sharing their password.

**The Classic Example:**
"Sign in with Google" — you click the button, Google asks "do you want to let MyApp access your name and email?", you approve, MyApp gets a token to access only what you allowed.

**OAuth 2.0 Roles:**
- **Resource Owner** = the user (you)
- **Client** = the app wanting access (MyApp)
- **Authorization Server** = the login provider (Google)
- **Resource Server** = where the data lives (Google's API)

**Authorization Code Flow (Most Secure — for Server-Side Apps):**

```
1. User clicks "Login with Google" on MyApp
2. MyApp redirects user to Google:
   https://accounts.google.com/authorize?
     client_id=MYAPP_ID&
     redirect_uri=https://myapp.com/callback&
     response_type=code&
     scope=email+profile

3. User logs into Google, approves access
4. Google redirects back to MyApp with a code:
   https://myapp.com/callback?code=AUTH_CODE_HERE

5. MyApp's SERVER exchanges the code for tokens (never exposed to browser):
   POST https://oauth2.googleapis.com/token
   {
     client_id: "MYAPP_ID",
     client_secret: "MYAPP_SECRET",
     code: "AUTH_CODE_HERE",
     grant_type: "authorization_code",
     redirect_uri: "https://myapp.com/callback"
   }

6. Google returns access_token + refresh_token
7. MyApp uses access_token to call Google APIs on behalf of the user
```

**Analogy:** You want a friend to pick up your laundry. Instead of giving them your house key, you call the laundry service and say "let my friend pick up my clothes." The service gives your friend a one-time slip (token) that only works for picking up YOUR laundry, nothing else.

### Strategy 4: Session-Based Authentication (Traditional)

```
1. Client sends credentials: POST /login { email, password }
2. Server creates a session, stores it in memory/database/Redis
3. Server sends back a session ID in a cookie
4. Browser automatically sends the cookie with every request
5. Server looks up the session ID to identify the user
```

**Pros:** Simple, well-understood, easy to invalidate (delete the session).
**Cons:** Stateful (server must store sessions), harder to scale (need shared session store), doesn't work well for APIs consumed by mobile apps or third-party services.

**When to use:** Traditional web apps (server-rendered HTML). For APIs, prefer JWT.

### Role-Based Access Control (RBAC)

```javascript
// Define roles and permissions
const permissions = {
  admin:   ['read', 'write', 'delete', 'manage-users'],
  editor:  ['read', 'write'],
  viewer:  ['read']
};

// Middleware to check permissions
function authorize(...requiredPermissions) {
  return (req, res, next) => {
    const userRole = req.user.role;
    const userPermissions = permissions[userRole];

    const hasPermission = requiredPermissions.every(
      perm => userPermissions.includes(perm)
    );

    if (!hasPermission) {
      return res.status(403).json({
        error: { code: "FORBIDDEN", message: "Insufficient permissions" }
      });
    }
    next();
  };
}

// Usage in routes
app.get('/users', authenticate, authorize('read'), getUsers);
app.post('/users', authenticate, authorize('write'), createUser);
app.delete('/users/:id', authenticate, authorize('delete'), deleteUser);
```

---

## 12. Error Handling — The Make or Break of Developer Experience

### A Consistent Error Format

Pick one format and use it everywhere — errors are the #1 thing that determines developer experience with your API.

```json
{
  "error": {
    "code": "VALIDATION_ERROR",
    "message": "The request body contains invalid fields.",
    "details": [
      {
        "field": "email",
        "message": "Must be a valid email address.",
        "rejectedValue": "not-an-email"
      },
      {
        "field": "age",
        "message": "Must be at least 13.",
        "rejectedValue": -5
      }
    ],
    "timestamp": "2024-01-15T10:30:00Z",
    "path": "/api/v1/users",
    "requestId": "req-abc-123"
  }
}
```

### Error Code Catalog

Create a catalog of machine-readable error codes. Human-readable messages can change; codes are the contract.

```javascript
const ErrorCodes = {
  // Authentication
  AUTH_REQUIRED:       { status: 401, message: "Authentication required." },
  TOKEN_EXPIRED:       { status: 401, message: "Your session has expired. Please log in again." },
  TOKEN_INVALID:       { status: 401, message: "Invalid authentication token." },
  INSUFFICIENT_PERMS:  { status: 403, message: "You don't have permission to perform this action." },

  // Validation
  VALIDATION_ERROR:    { status: 422, message: "The request contains invalid data." },
  MISSING_FIELD:       { status: 422, message: "A required field is missing." },

  // Resources
  NOT_FOUND:           { status: 404, message: "The requested resource was not found." },
  ALREADY_EXISTS:      { status: 409, message: "A resource with this identifier already exists." },

  // Rate limiting
  RATE_LIMITED:        { status: 429, message: "Too many requests. Please try again later." },

  // Server
  INTERNAL_ERROR:      { status: 500, message: "An unexpected error occurred. Please try again." },
  SERVICE_UNAVAILABLE: { status: 503, message: "The service is temporarily unavailable." },
};
```

### Global Error Handler (Express.js Example)

```javascript
// Error handler middleware (must be last middleware)
app.use((err, req, res, next) => {
  // Log the full error internally
  logger.error({
    message: err.message,
    stack: err.stack,
    path: req.path,
    method: req.method,
    requestId: req.id,
    userId: req.user?.id,
  });

  // Known operational errors
  if (err.isOperational) {
    return res.status(err.statusCode).json({
      error: {
        code: err.code,
        message: err.message,
        details: err.details || undefined,
        requestId: req.id,
      }
    });
  }

  // Unknown errors — never expose internals
  res.status(500).json({
    error: {
      code: "INTERNAL_ERROR",
      message: "An unexpected error occurred. Please try again.",
      requestId: req.id,
    }
  });
});
```

### Common Mistake: Using 200 for Everything

```javascript
// TERRIBLE — how most beginners do it
app.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  if (!user) {
    return res.status(200).json({    // 200?! But there's an error!
      success: false,
      error: "User not found"
    });
  }
  res.status(200).json({ success: true, data: user });
});

// CORRECT — use proper status codes
app.get('/users/:id', (req, res) => {
  const user = findUser(req.params.id);
  if (!user) {
    return res.status(404).json({
      error: { code: "NOT_FOUND", message: "User not found." }
    });
  }
  res.status(200).json(user);
});
```

HTTP status codes exist for a reason. Clients, proxies, CDNs, monitoring tools, and load balancers all use them. Returning 200 for errors breaks all of that.

---

## 13. Rate Limiting & Throttling

### Why Rate Limit?

- Prevent abuse (someone scraping your entire database)
- Protect your server from overload
- Ensure fair usage across clients
- Guard against DDoS attacks
- Control costs (every request costs compute)

**Analogy:** A popular restaurant has a "2-hour seating limit during peak hours." Without it, one table occupying the space all night means other customers can't eat.

### Common Strategies

#### Fixed Window
Count requests per time window (e.g., 100 requests per minute). Reset the counter at the start of each minute.

**Problem:** A burst of 100 requests at 11:59:59 and another 100 at 12:00:01 means 200 requests in 2 seconds.

#### Sliding Window
Smooths out the burst problem by considering a rolling time window.

#### Token Bucket
Imagine a bucket that holds 100 tokens. Every request costs 1 token. Tokens refill at a rate of 10 per second. If the bucket is empty, requests are rejected. Allows controlled bursts.

### Implementing Rate Limiting

**Response headers to include:**
```
X-RateLimit-Limit: 100          # Max requests per window
X-RateLimit-Remaining: 67       # Requests left in this window
X-RateLimit-Reset: 1705420000   # Unix timestamp when the window resets
Retry-After: 30                 # Seconds to wait (only on 429 responses)
```

**429 Response:**
```json
{
  "error": {
    "code": "RATE_LIMITED",
    "message": "Rate limit exceeded. Try again in 30 seconds.",
    "retryAfter": 30
  }
}
```

### Rate Limiting Tiers

```
Free Tier:     60 requests/minute,   1,000/day
Basic Tier:    300 requests/minute,  10,000/day
Pro Tier:      1,000 requests/minute, 100,000/day
Enterprise:    Custom
```

### Rate Limit by What?

| By | Use When |
|----|----------|
| IP Address | Public APIs, unauthenticated endpoints |
| API Key | Third-party integrations |
| User ID | Authenticated users |
| Endpoint | Different limits for different operations (GET vs POST) |

Write-heavy endpoints (POST, PUT, DELETE) should have stricter limits than read endpoints (GET).

---

## 14. HATEOAS & Richardson Maturity Model

### Richardson Maturity Model — Levels of REST

Think of this as a "how RESTful is your API?" scorecard.

**Level 0: The Swamp of POX (Plain Old XML/JSON)**
One URL, one method (POST) for everything. Basically using HTTP as a transport tunnel.

```
POST /api
{ "action": "getUser", "userId": 42 }

POST /api
{ "action": "createOrder", "product": "laptop" }
```

This is just RPC (Remote Procedure Call) over HTTP. Not REST at all.

**Level 1: Resources**
Different URLs for different resources, but still using only POST.

```
POST /users     { "action": "get", "id": 42 }
POST /orders    { "action": "create", "product": "laptop" }
```

Better — at least resources have their own URLs.

**Level 2: HTTP Verbs**
Proper use of GET, POST, PUT, DELETE + correct status codes. This is where most "RESTful" APIs live.

```
GET    /users/42        → 200 OK
POST   /orders          → 201 Created
DELETE /users/42        → 204 No Content
```

**Level 3: HATEOAS (Hypermedia)**
Responses include links that tell the client what it can do next.

### HATEOAS — Hypermedia as the Engine of Application State

```json
{
  "id": 789,
  "status": "pending",
  "total": 150.00,
  "links": [
    { "rel": "self",    "href": "/orders/789",          "method": "GET" },
    { "rel": "cancel",  "href": "/orders/789/cancel",   "method": "POST" },
    { "rel": "pay",     "href": "/orders/789/pay",      "method": "POST" },
    { "rel": "items",   "href": "/orders/789/items",    "method": "GET" }
  ]
}
```

If the order was already paid:
```json
{
  "id": 789,
  "status": "paid",
  "total": 150.00,
  "links": [
    { "rel": "self",    "href": "/orders/789",          "method": "GET" },
    { "rel": "receipt", "href": "/orders/789/receipt",   "method": "GET" },
    { "rel": "refund",  "href": "/orders/789/refund",   "method": "POST" }
  ]
}
```

Notice: the `cancel` and `pay` links are gone because the order is already paid. The API tells the client what's possible based on the current state.

**Analogy:** It's like a GPS that shows you only the turns you can make from your current position, not every road in the city.

**In practice:** Most APIs don't implement full HATEOAS. Level 2 (proper resources + verbs + status codes) is the practical standard. HATEOAS is the theoretical ideal but adds complexity that most teams don't find worth it.

---

## 15. Idempotency — Safe Retries

### The Problem

A user clicks "Pay Now." The request takes 5 seconds. The user gets impatient and clicks again. Now they've been charged twice.

Or: your server processes the payment, but the response gets lost due to a network error. Your retry logic sends the payment request again. Double charge.

### What Is Idempotency?

An operation is **idempotent** if doing it multiple times has the same effect as doing it once.

| Method | Idempotent? | Why |
|--------|-------------|-----|
| GET | Yes | Reading data doesn't change it |
| PUT | Yes | Replacing a resource with the same data = same result |
| DELETE | Yes | Deleting something already deleted = still deleted |
| POST | No | Each POST creates a new resource |
| PATCH | Usually Yes | Depends on implementation |

### Implementing Idempotency for POST

Use an **idempotency key** — a unique identifier the client sends with the request. If the server sees the same key twice, it returns the original response without processing again.

```
POST /api/v1/payments
Idempotency-Key: pay_abc123_unique_key
Content-Type: application/json

{
  "amount": 5000,
  "currency": "INR",
  "method": "card"
}
```

**Server logic:**

```javascript
app.post('/payments', async (req, res) => {
  const idempotencyKey = req.headers['idempotency-key'];

  // Check if we've seen this key before
  const existing = await redis.get(`idempotency:${idempotencyKey}`);
  if (existing) {
    // Return the same response as before
    return res.status(200).json(JSON.parse(existing));
  }

  // Process the payment
  const payment = await processPayment(req.body);

  // Store the response keyed by the idempotency key (expire after 24h)
  await redis.set(`idempotency:${idempotencyKey}`, JSON.stringify(payment), 'EX', 86400);

  res.status(201).json(payment);
});
```

**Stripe uses this pattern.** Every Stripe API call that creates something accepts an `Idempotency-Key` header. If the network fails and you retry with the same key, you don't get charged twice.

---

## 16. Caching in APIs

### Why Cache?

Caching reduces database load, reduces response time, saves bandwidth, and reduces costs. If 1000 users request the same product page in a minute, why query the database 1000 times?

### HTTP Caching Headers

#### Cache-Control — The Main Directive

```
# Cache for 1 hour (browser + CDN can cache)
Cache-Control: public, max-age=3600

# Cache for 1 hour (only the user's browser, not CDNs)
Cache-Control: private, max-age=3600

# Don't cache at all
Cache-Control: no-store

# Can cache but must revalidate before using
Cache-Control: no-cache

# Cache for 5 min, but CDN can serve stale content for 1 hour while revalidating
Cache-Control: public, max-age=300, stale-while-revalidate=3600
```

#### ETag — Fingerprint-Based Caching

```
# Server sends an ETag (hash of the content)
GET /api/v1/products/42
→ 200 OK
   ETag: "abc123"
   { "name": "Laptop", "price": 50000 }

# Client's next request includes the ETag
GET /api/v1/products/42
If-None-Match: "abc123"

# If content hasn't changed:
→ 304 Not Modified (no body — use your cached version)

# If content changed:
→ 200 OK
   ETag: "def456"
   { "name": "Laptop", "price": 45000 }
```

**Analogy:** Calling a restaurant to ask "is the menu the same as last time?" If yes, you don't need them to read the entire menu to you again.

### What to Cache vs What Not to Cache

| Cache Aggressively | Don't Cache |
|-------------------|-------------|
| Product listings | User-specific data (unless private cache) |
| Static content | Real-time data (stock prices, live scores) |
| Search results | Authentication endpoints |
| Configuration data | Write operations (POST, PUT, DELETE) |
| Public profiles | Sensitive data |

---

## 17. File Upload & Download

### Uploading Files

#### Strategy 1: Multipart Form Data (Simple, Direct)

```
POST /api/v1/users/42/avatar
Content-Type: multipart/form-data; boundary=----WebKitFormBoundary

------WebKitFormBoundary
Content-Disposition: form-data; name="avatar"; filename="photo.jpg"
Content-Type: image/jpeg

(binary data)
------WebKitFormBoundary--
```

```javascript
// Express.js with multer
const multer = require('multer');
const upload = multer({
  limits: { fileSize: 5 * 1024 * 1024 },  // 5MB limit
  fileFilter: (req, file, cb) => {
    const allowed = ['image/jpeg', 'image/png', 'image/webp'];
    cb(null, allowed.includes(file.mimetype));
  }
});

app.post('/users/:id/avatar', upload.single('avatar'), (req, res) => {
  // req.file contains the uploaded file
  const url = await uploadToS3(req.file);
  res.status(200).json({ avatarUrl: url });
});
```

#### Strategy 2: Pre-Signed URLs (Better for Large Files)

Instead of uploading through your API server, the client uploads directly to cloud storage (S3).

```
Step 1: Client asks your API for an upload URL
POST /api/v1/upload-urls
{ "filename": "report.pdf", "contentType": "application/pdf" }

Step 2: Your API generates a pre-signed URL
{
  "uploadUrl": "https://s3.amazonaws.com/bucket/uuid-report.pdf?signature=...",
  "fileId": "file-uuid-123",
  "expiresIn": 300
}

Step 3: Client uploads directly to S3 using the pre-signed URL
PUT https://s3.amazonaws.com/bucket/uuid-report.pdf?signature=...
Content-Type: application/pdf
(binary data)

Step 4: Client tells your API the upload is complete
POST /api/v1/files/file-uuid-123/confirm
```

**Why pre-signed URLs?** Your API server never handles the file data — no memory pressure, no bandwidth cost. S3 handles the heavy lifting. This scales much better for large files.

### Downloading Files

```javascript
// Serve a file
app.get('/api/v1/invoices/:id/pdf', async (req, res) => {
  const invoice = await Invoice.findById(req.params.id);

  // Option 1: Redirect to a pre-signed download URL
  const url = await getSignedDownloadUrl(invoice.s3Key);
  res.redirect(302, url);

  // Option 2: Stream the file through your server
  res.setHeader('Content-Type', 'application/pdf');
  res.setHeader('Content-Disposition', 'attachment; filename="invoice-789.pdf"');
  const stream = await getFileStream(invoice.s3Key);
  stream.pipe(res);
});
```

---

## 18. Webhooks — APIs in Reverse

### What Is a Webhook?

Normal API: "Hey server, has anything changed?" (you keep asking — **polling**).
Webhook: "Server, call ME when something changes" (server pushes to you — **event-driven**).

**Analogy:**
- **Polling** = Checking your mailbox every 5 minutes to see if a package arrived.
- **Webhook** = Telling the delivery person to ring your doorbell when they arrive.

### How Webhooks Work

```
1. Your app registers a webhook URL with a service:
   POST /api/v1/webhooks
   {
     "url": "https://myapp.com/webhooks/stripe",
     "events": ["payment.completed", "payment.failed", "refund.created"]
   }

2. When an event occurs, the service sends a POST to your URL:
   POST https://myapp.com/webhooks/stripe
   Content-Type: application/json
   X-Webhook-Signature: sha256=abc123...

   {
     "event": "payment.completed",
     "timestamp": "2024-01-15T10:30:00Z",
     "data": {
       "paymentId": "pay_123",
       "amount": 5000,
       "currency": "INR"
     }
   }

3. Your app processes the event and returns 200 OK.
```

### Building a Webhook Receiver

```javascript
app.post('/webhooks/stripe', express.raw({ type: 'application/json' }), (req, res) => {
  const signature = req.headers['x-webhook-signature'];

  // CRITICAL: Verify the signature to ensure the request is genuine
  if (!verifySignature(req.body, signature, WEBHOOK_SECRET)) {
    return res.status(401).json({ error: "Invalid signature" });
  }

  const event = JSON.parse(req.body);

  // Process asynchronously — don't hold up the response
  eventQueue.add(event);

  // Always respond quickly with 200
  res.status(200).json({ received: true });
});
```

### Webhook Best Practices

- **Always verify signatures** — anyone could send a POST to your webhook URL
- **Respond quickly (< 5 seconds)** — process the event asynchronously via a queue
- **Handle duplicates** — webhooks can be sent more than once (use event IDs for idempotency)
- **Handle out-of-order delivery** — events might arrive in a different order than they occurred
- **Implement retry logic** (as a sender) — if the receiver returns a non-2xx status, retry with exponential backoff
- **Log everything** — webhook debugging is hard without logs

---

## 19. Real-Time APIs — WebSockets & SSE

### When REST Isn't Enough

REST is request-response: the client asks, the server answers. But what about:
- Chat messages that should appear instantly
- Live stock prices
- Notifications
- Collaborative editing
- Live sports scores

For these, you need real-time communication.

### WebSockets — Full Duplex Communication

```
HTTP:       Client → "Any new messages?" → Server → "No"
            Client → "Any new messages?" → Server → "No"
            Client → "Any new messages?" → Server → "Yes, here's one"

WebSocket:  Client ←→ Server (permanent open connection)
            Server → "New message from Ravi!"
            Client → "I'm typing..."
            Server → "Priya is also typing..."
```

**How it works:**
1. Client initiates an HTTP request with an "Upgrade" header
2. Server agrees, the connection is "upgraded" to WebSocket
3. Both sides can send messages at any time
4. Connection stays open until either side closes it

```javascript
// Server (using ws library)
const WebSocket = require('ws');
const wss = new WebSocket.Server({ port: 8080 });

wss.on('connection', (ws) => {
  console.log('Client connected');

  ws.on('message', (data) => {
    const message = JSON.parse(data);

    // Broadcast to all connected clients
    wss.clients.forEach(client => {
      if (client.readyState === WebSocket.OPEN) {
        client.send(JSON.stringify({
          type: 'chat_message',
          user: message.user,
          text: message.text,
          timestamp: new Date().toISOString()
        }));
      }
    });
  });

  ws.on('close', () => console.log('Client disconnected'));
});

// Client
const ws = new WebSocket('ws://localhost:8080');
ws.onmessage = (event) => {
  const message = JSON.parse(event.data);
  console.log(`${message.user}: ${message.text}`);
};
ws.send(JSON.stringify({ user: 'Ravi', text: 'Hello!' }));
```

### Server-Sent Events (SSE) — Server to Client Only

If you only need the server to push data to the client (not the other way around), SSE is simpler than WebSockets.

```javascript
// Server
app.get('/api/v1/notifications/stream', (req, res) => {
  res.setHeader('Content-Type', 'text/event-stream');
  res.setHeader('Cache-Control', 'no-cache');
  res.setHeader('Connection', 'keep-alive');

  // Send a notification every time one is created
  const listener = (notification) => {
    res.write(`data: ${JSON.stringify(notification)}\n\n`);
  };

  notificationEmitter.on('new', listener);

  req.on('close', () => {
    notificationEmitter.off('new', listener);
  });
});

// Client
const source = new EventSource('/api/v1/notifications/stream');
source.onmessage = (event) => {
  const notification = JSON.parse(event.data);
  showNotification(notification);
};
```

### When to Use What

| Technology | Use When | Examples |
|-----------|----------|---------|
| **REST** | Request-response, CRUD operations | User profiles, order placement |
| **WebSockets** | Bidirectional real-time communication | Chat, multiplayer games, collaborative editing |
| **SSE** | Server-to-client real-time push | Notifications, live feeds, dashboards |
| **Webhooks** | Event-driven server-to-server | Payment confirmations, CI/CD triggers |
| **Long Polling** | Fallback when SSE/WS not available | Legacy browser support |

---

## 20. GraphQL — The Complete Guide

### What Is GraphQL?

GraphQL is a query language for APIs invented by Facebook in 2012 (open-sourced in 2015). Instead of the server deciding what data to return (REST), the CLIENT decides.

**The Restaurant Analogy — Upgraded:**
- **REST** = A fixed menu. You order "Lunch Combo #3" and get everything in it — even the coleslaw you don't want.
- **GraphQL** = A buffet where you pick exactly what you want on your plate.

### The Problem GraphQL Solves

**Over-Fetching:** REST returns everything. You need just the user's name, but GET /users/42 returns name, email, address, phone, bio, avatar, preferences, settings...

**Under-Fetching:** You need the user's name AND their last 3 orders AND each order's product names. In REST, that's 3 separate API calls: GET /users/42, GET /users/42/orders?limit=3, then GET /orders/1/products for each order. The client has to orchestrate multiple requests.

**GraphQL fixes both:** One request, you ask for exactly what you need, you get exactly that.

### How GraphQL Works

Every GraphQL API has one endpoint:

```
POST /graphql
```

Yes, just one. No /users, /products, /orders. Everything goes through /graphql.

### Schema — The Contract

The schema defines what data exists and how it's structured:

```graphql
# Types — what things look like
type User {
  id: ID!                    # ! means required (non-null)
  name: String!
  email: String!
  age: Int
  posts: [Post!]!            # Array of Post objects
  followers: [User!]!
  createdAt: String!
}

type Post {
  id: ID!
  title: String!
  content: String!
  author: User!
  comments: [Comment!]!
  likes: Int!
  publishedAt: String
}

type Comment {
  id: ID!
  text: String!
  author: User!
  createdAt: String!
}

# Queries — how to read data (like GET)
type Query {
  user(id: ID!): User
  users(page: Int, limit: Int): [User!]!
  post(id: ID!): Post
  posts(authorId: ID, sortBy: String): [Post!]!
  searchPosts(query: String!): [Post!]!
}

# Mutations — how to write data (like POST/PUT/DELETE)
type Mutation {
  createUser(input: CreateUserInput!): User!
  updateUser(id: ID!, input: UpdateUserInput!): User!
  deleteUser(id: ID!): Boolean!
  createPost(input: CreatePostInput!): Post!
  likePost(postId: ID!): Post!
}

# Input types — structured input for mutations
input CreateUserInput {
  name: String!
  email: String!
  password: String!
}

input UpdateUserInput {
  name: String
  email: String
  bio: String
}

input CreatePostInput {
  title: String!
  content: String!
}

# Subscriptions — real-time updates (like WebSockets)
type Subscription {
  postCreated: Post!
  commentAdded(postId: ID!): Comment!
}
```

### Queries — Reading Data

**Basic query:**
```graphql
query {
  user(id: "42") {
    name
    email
  }
}
```

**Response:**
```json
{
  "data": {
    "user": {
      "name": "Ravi Kumar",
      "email": "ravi@example.com"
    }
  }
}
```

No over-fetching. You asked for `name` and `email`, you got `name` and `email`. Nothing more.

**Nested query (solves under-fetching):**
```graphql
query {
  user(id: "42") {
    name
    posts {
      title
      likes
      comments {
        text
        author {
          name
        }
      }
    }
  }
}
```

**Response:**
```json
{
  "data": {
    "user": {
      "name": "Ravi Kumar",
      "posts": [
        {
          "title": "GraphQL is Amazing",
          "likes": 150,
          "comments": [
            {
              "text": "Great article!",
              "author": { "name": "Priya Singh" }
            }
          ]
        }
      ]
    }
  }
}
```

One request. User name, their posts, each post's comments, each comment's author. In REST, this would be 5-10 API calls.

**Query with arguments:**
```graphql
query {
  posts(authorId: "42", sortBy: "likes") {
    title
    likes
    publishedAt
  }
}
```

**Multiple queries in one request:**
```graphql
query DashboardData {
  currentUser: user(id: "42") {
    name
    email
  }
  recentPosts: posts(sortBy: "publishedAt") {
    title
    likes
  }
  topUsers: users(limit: 5) {
    name
    followers {
      name
    }
  }
}
```

### Mutations — Writing Data

```graphql
mutation {
  createUser(input: {
    name: "Ravi Kumar"
    email: "ravi@example.com"
    password: "securePassword123"
  }) {
    id
    name
    email
    createdAt
  }
}
```

**Response:**
```json
{
  "data": {
    "createUser": {
      "id": "42",
      "name": "Ravi Kumar",
      "email": "ravi@example.com",
      "createdAt": "2024-01-15T10:30:00Z"
    }
  }
}
```

**Update mutation:**
```graphql
mutation {
  updateUser(id: "42", input: {
    name: "Ravi K."
    bio: "Backend Developer"
  }) {
    id
    name
    bio
  }
}
```

### Subscriptions — Real-Time

```graphql
subscription {
  commentAdded(postId: "100") {
    text
    author {
      name
    }
    createdAt
  }
}
```

This opens a WebSocket connection and pushes data whenever a new comment is added to post 100.

### Resolvers — Where the Logic Lives (Server Side)

```javascript
// Node.js with Apollo Server
const resolvers = {
  Query: {
    user: async (_, { id }) => {
      return await User.findById(id);
    },
    users: async (_, { page = 1, limit = 20 }) => {
      return await User.find()
        .skip((page - 1) * limit)
        .limit(limit);
    },
    post: async (_, { id }) => {
      return await Post.findById(id);
    },
  },

  Mutation: {
    createUser: async (_, { input }) => {
      const user = new User(input);
      return await user.save();
    },
    likePost: async (_, { postId }) => {
      return await Post.findByIdAndUpdate(
        postId,
        { $inc: { likes: 1 } },
        { new: true }
      );
    },
  },

  // Field-level resolvers — resolve nested fields
  User: {
    posts: async (user) => {
      return await Post.find({ authorId: user.id });
    },
    followers: async (user) => {
      return await User.find({ _id: { $in: user.followerIds } });
    },
  },

  Post: {
    author: async (post) => {
      return await User.findById(post.authorId);
    },
    comments: async (post) => {
      return await Comment.find({ postId: post.id });
    },
  },
};
```

### The N+1 Problem — GraphQL's Biggest Pitfall

```graphql
query {
  posts {           # 1 query to fetch 20 posts
    title
    author {        # 20 queries to fetch each post's author!
      name
    }
  }
}
```

This results in 21 database queries (1 for posts + 20 for authors). If there are 100 posts, that's 101 queries.

**The Fix: DataLoader (Batching)**

```javascript
const DataLoader = require('dataloader');

// Batch function: takes an array of IDs, returns an array of users
const userLoader = new DataLoader(async (userIds) => {
  const users = await User.find({ _id: { $in: userIds } });
  // Return in the same order as the input IDs
  return userIds.map(id => users.find(u => u.id === id));
});

// Resolver uses the loader
Post: {
  author: (post) => userLoader.load(post.authorId)
  // Instead of 20 individual queries, DataLoader batches them into 1:
  // SELECT * FROM users WHERE id IN (1, 2, 3, ..., 20)
}
```

### GraphQL Error Handling

GraphQL always returns 200 OK (even on errors). Errors are in the response body:

```json
{
  "data": {
    "user": null
  },
  "errors": [
    {
      "message": "User not found",
      "locations": [{ "line": 2, "column": 3 }],
      "path": ["user"],
      "extensions": {
        "code": "NOT_FOUND",
        "statusCode": 404
      }
    }
  ]
}
```

**Partial success is possible** — some fields resolve, others error:

```json
{
  "data": {
    "user": {
      "name": "Ravi Kumar",
      "posts": null
    }
  },
  "errors": [
    {
      "message": "Failed to fetch posts",
      "path": ["user", "posts"]
    }
  ]
}
```

### GraphQL Authentication & Authorization

```javascript
// Context — pass the authenticated user to every resolver
const server = new ApolloServer({
  typeDefs,
  resolvers,
  context: ({ req }) => {
    const token = req.headers.authorization?.replace('Bearer ', '');
    const user = verifyToken(token);
    return { user };
  }
});

// Resolver checks authorization
const resolvers = {
  Mutation: {
    deleteUser: async (_, { id }, context) => {
      if (!context.user) throw new AuthenticationError('Not logged in');
      if (context.user.role !== 'admin') throw new ForbiddenError('Admin only');
      return await User.findByIdAndDelete(id);
    },
  },
};
```

### GraphQL Pagination

GraphQL uses **connection-based pagination** (the Relay spec):

```graphql
query {
  posts(first: 10, after: "cursor_abc") {
    edges {
      node {
        id
        title
      }
      cursor
    }
    pageInfo {
      hasNextPage
      hasPreviousPage
      startCursor
      endCursor
    }
    totalCount
  }
}
```

---

## 21. REST vs GraphQL — The Honest Comparison

| Aspect | REST | GraphQL |
|--------|------|---------|
| **Endpoints** | Many (/users, /posts, /orders) | One (/graphql) |
| **Data fetching** | Server decides what to return | Client decides what to return |
| **Over-fetching** | Common | Eliminated |
| **Under-fetching** | Common (need multiple calls) | Eliminated (nested queries) |
| **Caching** | Easy (HTTP caching, CDNs) | Hard (single endpoint, POST-based) |
| **Learning curve** | Low | Medium-High |
| **Tooling** | Mature, everywhere | Growing, excellent in JS ecosystem |
| **File uploads** | Native (multipart) | Needs workarounds |
| **Error handling** | HTTP status codes | Always 200, errors in body |
| **Versioning** | URL versioning (/v1, /v2) | Schema evolution (deprecate fields) |
| **Real-time** | Needs WebSockets/SSE separately | Built-in subscriptions |
| **Best for** | Simple CRUD, public APIs, microservices | Complex data relationships, mobile apps, dashboards |

### When to Use REST

- Simple CRUD applications
- Public APIs (external developers expect REST)
- Microservices communicating with each other
- When HTTP caching is critical
- When your team is more comfortable with REST
- When you need file upload/download natively

### When to Use GraphQL

- Complex data with deep relationships (social networks, e-commerce)
- Mobile apps (bandwidth matters — fetch only what you need)
- Dashboards that combine data from multiple sources
- Rapid frontend development (frontend can evolve without backend changes)
- When multiple clients need different views of the same data

### Can You Use Both?

Absolutely. Many companies use REST for simple services and public APIs, and GraphQL as a gateway that aggregates data from multiple REST microservices. It's not either/or.

```
Mobile App ──→ GraphQL Gateway ──→ User Service (REST)
Web App   ──→                  ──→ Order Service (REST)
                               ──→ Product Service (REST)
```

---

## 22. API Documentation — If It's Not Documented, It Doesn't Exist

### OpenAPI / Swagger (The Standard for REST)

OpenAPI is a specification for describing REST APIs. Swagger is the tooling ecosystem around it.

```yaml
# openapi.yaml
openapi: 3.0.3
info:
  title: My E-Commerce API
  version: 1.0.0
  description: API for managing products, orders, and users

servers:
  - url: https://api.myapp.com/v1

paths:
  /products:
    get:
      summary: List all products
      tags:
        - Products
      parameters:
        - name: category
          in: query
          schema:
            type: string
          description: Filter by category
        - name: page
          in: query
          schema:
            type: integer
            default: 1
        - name: limit
          in: query
          schema:
            type: integer
            default: 20
            maximum: 100
      responses:
        '200':
          description: A list of products
          content:
            application/json:
              schema:
                type: object
                properties:
                  data:
                    type: array
                    items:
                      $ref: '#/components/schemas/Product'
                  meta:
                    $ref: '#/components/schemas/PaginationMeta'

    post:
      summary: Create a new product
      tags:
        - Products
      security:
        - bearerAuth: []
      requestBody:
        required: true
        content:
          application/json:
            schema:
              $ref: '#/components/schemas/CreateProductInput'
      responses:
        '201':
          description: Product created successfully

components:
  schemas:
    Product:
      type: object
      properties:
        id:
          type: integer
        name:
          type: string
        price:
          type: number
        category:
          type: string
        createdAt:
          type: string
          format: date-time

    CreateProductInput:
      type: object
      required:
        - name
        - price
      properties:
        name:
          type: string
          minLength: 2
          maxLength: 200
        price:
          type: number
          minimum: 0
        category:
          type: string

    PaginationMeta:
      type: object
      properties:
        currentPage:
          type: integer
        totalPages:
          type: integer
        totalItems:
          type: integer

  securitySchemes:
    bearerAuth:
      type: http
      scheme: bearer
      bearerFormat: JWT
```

### What Good Documentation Includes

1. **Authentication** — How to get and use tokens, with examples
2. **Endpoints** — Every endpoint with method, URL, parameters, request body, and response
3. **Examples** — Real request/response examples (not just schemas)
4. **Error codes** — Every possible error with its code, message, and how to fix it
5. **Rate limits** — What the limits are and how to handle 429 responses
6. **Changelog** — What changed in each version
7. **SDKs** — Code examples in multiple languages
8. **Quick start** — "Make your first API call in 5 minutes"

### Tools for Documentation

| Tool | Type | Best For |
|------|------|----------|
| **Swagger UI** | REST | Interactive API explorer from OpenAPI spec |
| **Redoc** | REST | Beautiful static documentation from OpenAPI |
| **Postman** | REST | Collections that double as documentation |
| **GraphQL Playground** | GraphQL | Interactive GraphQL IDE |
| **Apollo Studio** | GraphQL | Schema exploration and analytics |

---

## 23. API Security Checklist

### The Must-Have Security Measures

1. **Always use HTTPS** — never HTTP. No exceptions. Enforce via HSTS headers.

2. **Validate all input** — every field, every parameter, every header. Assume all input is malicious.

3. **Authenticate every request** — except explicitly public endpoints.

4. **Authorize at the resource level** — user 42 should NOT be able to access user 43's data just by changing the ID in the URL (this is called IDOR — Insecure Direct Object Reference).

```javascript
// BAD — no ownership check
app.get('/users/:id/documents', (req, res) => {
  const docs = await Document.find({ userId: req.params.id });
  res.json(docs);  // Any authenticated user can see anyone's documents!
});

// GOOD — verify ownership
app.get('/users/:id/documents', (req, res) => {
  if (req.user.id !== req.params.id && req.user.role !== 'admin') {
    return res.status(403).json({ error: "Access denied" });
  }
  const docs = await Document.find({ userId: req.params.id });
  res.json(docs);
});
```

5. **Rate limit everything** — especially authentication endpoints.

6. **Never expose internal errors** — stack traces, database queries, file paths.

7. **Use parameterised queries** — never concatenate user input into SQL/NoSQL queries.

```javascript
// BAD — SQL injection
const query = `SELECT * FROM users WHERE email = '${req.body.email}'`;

// GOOD — parameterised
const query = `SELECT * FROM users WHERE email = $1`;
db.query(query, [req.body.email]);
```

8. **Set security headers:**
```javascript
app.use(helmet());  // Sets various security headers

// Key headers:
// X-Content-Type-Options: nosniff
// X-Frame-Options: DENY
// Strict-Transport-Security: max-age=31536000
// Content-Security-Policy: default-src 'self'
```

9. **Implement CORS properly:**
```javascript
app.use(cors({
  origin: ['https://myapp.com', 'https://admin.myapp.com'],  // NOT '*' in production
  methods: ['GET', 'POST', 'PUT', 'PATCH', 'DELETE'],
  allowedHeaders: ['Content-Type', 'Authorization'],
  credentials: true,
}));
```

10. **Log everything** — requests, responses (sanitised), errors, authentication attempts.

---

## 24. API Design Patterns in the Real World

### Pattern 1: Bulk Operations

Instead of making 100 API calls to update 100 products, allow batch operations:

```
PATCH /api/v1/products/bulk
{
  "operations": [
    { "id": 1, "action": "update", "data": { "price": 999 } },
    { "id": 2, "action": "update", "data": { "price": 1499 } },
    { "id": 3, "action": "delete" }
  ]
}

Response:
{
  "results": [
    { "id": 1, "status": "success" },
    { "id": 2, "status": "success" },
    { "id": 3, "status": "error", "error": "Product has pending orders" }
  ],
  "summary": { "succeeded": 2, "failed": 1 }
}
```

### Pattern 2: Long-Running Operations (Async API)

For operations that take more than a few seconds (report generation, data export, video processing):

```
Step 1: Start the job
POST /api/v1/reports/generate
{ "type": "sales", "period": "2024-Q1" }

Response: 202 Accepted
{
  "jobId": "job-abc-123",
  "status": "processing",
  "statusUrl": "/api/v1/jobs/job-abc-123",
  "estimatedTime": 120
}

Step 2: Poll for status
GET /api/v1/jobs/job-abc-123

Response (still processing):
{ "jobId": "job-abc-123", "status": "processing", "progress": 65 }

Response (complete):
{
  "jobId": "job-abc-123",
  "status": "completed",
  "result": {
    "downloadUrl": "/api/v1/reports/job-abc-123/download",
    "expiresAt": "2024-01-16T10:30:00Z"
  }
}
```

### Pattern 3: Soft Deletes

Don't actually delete records — mark them as deleted:

```javascript
// Soft delete
app.delete('/users/:id', async (req, res) => {
  await User.findByIdAndUpdate(req.params.id, {
    deletedAt: new Date(),
    isActive: false
  });
  res.status(204).send();
});

// All GET queries filter out soft-deleted records
app.get('/users', async (req, res) => {
  const users = await User.find({ deletedAt: null });
  res.json(users);
});
```

### Pattern 4: API Gateway Pattern

A single entry point that routes, authenticates, rate limits, and aggregates:

```
Client → API Gateway → Authentication Check
                     → Rate Limiting
                     → Route to User Service
                     → Route to Order Service
                     → Route to Product Service
                     → Aggregate responses
                     → Return to client
```

### Pattern 5: Health Check Endpoint

```javascript
app.get('/health', async (req, res) => {
  const checks = {
    uptime: process.uptime(),
    timestamp: new Date().toISOString(),
    database: 'unknown',
    redis: 'unknown',
  };

  try {
    await db.ping();
    checks.database = 'healthy';
  } catch (e) {
    checks.database = 'unhealthy';
  }

  try {
    await redis.ping();
    checks.redis = 'healthy';
  } catch (e) {
    checks.redis = 'unhealthy';
  }

  const isHealthy = checks.database === 'healthy' && checks.redis === 'healthy';
  res.status(isHealthy ? 200 : 503).json(checks);
});
```

---

## 25. Interview Questions (80+ Questions)

### Beginner — Fundamentals

**Q1: What is an API?**
An API (Application Programming Interface) is a set of rules and protocols that allows different software applications to communicate with each other. Think of it as a contract: "send me this kind of request, and I'll give you this kind of response." As a backend developer, you build APIs that serve data to frontends, mobile apps, and other services.

**Q2: What is REST?**
REST (Representational State Transfer) is an architectural style for designing networked applications. It uses standard HTTP methods (GET, POST, PUT, DELETE) to perform operations on resources identified by URLs. REST is stateless — each request contains all information needed to process it. It's the most widely used approach for building web APIs.

**Q3: What is the difference between REST and SOAP?**
REST is an architectural style that uses standard HTTP and JSON. It's lightweight, flexible, and easy to use. SOAP (Simple Object Access Protocol) is a protocol with strict standards, uses XML exclusively, has built-in security (WS-Security), and supports features like transactions and ACID compliance. REST is dominant for web APIs. SOAP is still used in enterprise systems, banking, and telecommunications where strict contracts and reliability are critical.

**Q4: What are HTTP methods? Name the main ones.**
HTTP methods define the action to perform on a resource. GET reads data (safe, idempotent). POST creates new data (not idempotent). PUT replaces an entire resource (idempotent). PATCH partially updates a resource (usually idempotent). DELETE removes a resource (idempotent). There are also HEAD (like GET without body), OPTIONS (what methods are allowed), and TRACE (debugging).

**Q5: What does "stateless" mean in REST?**
Each request from client to server must contain all the information needed to understand and process the request. The server doesn't store any client context between requests. This means no server-side sessions tracking "who you are." Authentication info (like a JWT) must be sent with every request. This makes scaling easy — any server can handle any request.

**Q6: What is the difference between PUT and PATCH?**
PUT replaces the entire resource — you must send all fields, and any missing field is set to null or its default. PATCH updates only the fields you include — unmentioned fields remain unchanged. PUT is like rewriting an entire document; PATCH is like editing one paragraph. Most APIs prefer PATCH for updates because it's simpler and less error-prone.

**Q7: What are HTTP status codes? Name the most important ones.**
Status codes tell the client what happened. 200 OK means success with data returned. 201 Created means a new resource was created. 204 No Content means success with nothing to return. 400 Bad Request means invalid input. 401 Unauthorized means not authenticated. 403 Forbidden means authenticated but not allowed. 404 Not Found means the resource doesn't exist. 422 Unprocessable Entity means validation failed. 429 Too Many Requests means rate limited. 500 Internal Server Error means the server crashed.

**Q8: What is the difference between 401 and 403?**
401 means "I don't know who you are" — you haven't logged in, your token expired, or your credentials are wrong. The fix is to authenticate. 403 means "I know who you are, but you can't do this" — you're logged in but lack the necessary permissions. The fix is to get higher-level access. A bouncer asking for your ID is 401. A bouncer who checks your ID but says "VIP only" is 403.

**Q9: What is a query parameter?**
A key-value pair appended to the URL after a `?` mark, used for filtering, sorting, searching, and pagination. Example: `/products?category=electronics&sort=-price&page=2`. Multiple parameters are separated by `&`. They're for optional, non-resource data. Resource identifiers go in the URL path (/users/42), not query params (/users?id=42).

**Q10: What is JSON?**
JSON (JavaScript Object Notation) is a lightweight data format used for transmitting data in APIs. It's human-readable and language-independent. It supports strings, numbers, booleans, null, arrays, and nested objects. Almost all modern APIs use JSON for request and response bodies. It replaced XML as the dominant data format because of its simplicity.

---

### Intermediate — Design Decisions

**Q11: How do you design good RESTful URLs?**
Use nouns not verbs (/users, not /getUsers). Use plural nouns consistently. Use kebab-case for multi-word resources (/order-items). Nest resources to show relationships (/users/42/orders) but don't go deeper than 2 levels. Use query parameters for filtering, not URL paths. The HTTP method is the verb — GET reads, POST creates, PUT/PATCH updates, DELETE removes. Keep URLs predictable and self-documenting.

**Q12: What is API versioning? What strategies exist?**
Versioning lets you evolve your API without breaking existing clients. URL path versioning (/api/v1/users) is the most common and simplest. Header versioning uses Accept or custom headers. Query parameter versioning adds ?version=2. URL path versioning is the industry standard — it's obvious, easy to test, and easy to route. Version only when making breaking changes (removing/renaming fields, changing types).

**Q13: What is pagination? Compare different strategies.**
Pagination breaks large result sets into manageable chunks. Offset-based (page=2&limit=20) is simple but slow on large datasets and inconsistent when data changes. Cursor-based uses a pointer to the last item seen — it's fast and consistent but can't jump to arbitrary pages. Keyset pagination uses actual column values (created_before=timestamp). Use offset for admin panels, cursor for feeds and large datasets.

**Q14: What is HATEOAS?**
Hypermedia As The Engine Of Application State — Level 3 of the Richardson Maturity Model. API responses include links that tell the client what actions are possible from the current state. For example, an order response includes links to cancel, pay, or view items based on the order's current status. It's the theoretical ideal of REST but rarely implemented in practice because it adds complexity most teams don't find worthwhile.

**Q15: How do you handle errors in a REST API?**
Use proper HTTP status codes (don't return 200 for errors). Use a consistent error response format with a machine-readable code, human-readable message, and field-level details for validation errors. Include a request ID for debugging. Never expose internal details (stack traces, SQL queries) in production. Create an error code catalog. Implement a global error handler that catches and formats all errors consistently.

**Q16: What is content negotiation?**
The process of selecting the best representation for a resource. The client uses the Accept header to say what format it wants (application/json, application/xml, text/html), and the server returns data in that format. Content-Type tells the server what format the request body is in. If the server can't provide the requested format, it returns 406 Not Acceptable.

**Q17: What is idempotency and why does it matter?**
An operation is idempotent if performing it multiple times has the same effect as once. GET, PUT, and DELETE are naturally idempotent. POST is not — calling it twice creates two resources. Idempotency is critical for handling network retries. If a payment request fails due to a timeout, the client retries — without idempotency, the user gets charged twice. Use idempotency keys for POST operations.

**Q18: How do you handle file uploads in an API?**
Two approaches: multipart form data uploads (simple, the file goes through your server) and pre-signed URLs (the client uploads directly to cloud storage like S3). Pre-signed URLs are better for large files because your API server never handles the file data, reducing memory and bandwidth usage. Always validate file type, file size, and scan for malware before processing.

**Q19: What are webhooks? How are they different from APIs?**
A regular API is pull-based — the client asks for data. A webhook is push-based — the server notifies the client when an event occurs by sending an HTTP POST to a registered URL. Webhooks eliminate the need for polling. Key practices: verify webhook signatures for security, respond quickly (process asynchronously), handle duplicates (use event IDs for idempotency), and implement retry logic.

**Q20: What is CORS and why does it exist?**
Cross-Origin Resource Sharing is a security mechanism that controls which websites can make API requests. Browsers block requests from a different origin (different domain, port, or protocol) by default — this is the Same-Origin Policy. CORS headers (Access-Control-Allow-Origin, etc.) tell the browser which origins are allowed. It exists to prevent malicious websites from making requests to your API using a user's browser cookies. In production, never use `*` for allowed origins — whitelist specific domains.

---

### Advanced — Architecture & Production

**Q21: What is the N+1 problem in GraphQL?**
When a query requests a list of items and each item has a nested field that requires a separate database query. Fetching 20 posts with their authors results in 1 query for posts + 20 queries for authors = 21 total. The fix is DataLoader (batching) — it collects all author IDs and makes a single query. This is the most common performance problem in GraphQL and a favourite interview question.

**Q22: REST vs GraphQL — when would you choose each?**
REST: simple CRUD apps, public APIs, microservice-to-microservice communication, when HTTP caching is critical, smaller teams or teams unfamiliar with GraphQL. GraphQL: complex data relationships (social networks), mobile apps where bandwidth matters, dashboards aggregating multiple data sources, rapid frontend development where the frontend needs flexibility. They can coexist — use GraphQL as a gateway aggregating multiple REST services.

**Q23: How do you secure an API?**
Always use HTTPS. Validate all input. Authenticate every request (JWT, API keys, OAuth). Authorize at the resource level (check ownership, not just authentication). Rate limit everything, especially auth endpoints. Use parameterised queries to prevent injection. Set security headers (HSTS, CSP, X-Frame-Options). Implement CORS properly. Log everything. Never expose internal errors. Scan for OWASP Top 10 vulnerabilities.

**Q24: What is an API Gateway? Why use one?**
An API Gateway is a single entry point for all API requests. It handles cross-cutting concerns: authentication, rate limiting, logging, monitoring, request routing, response aggregation, protocol translation, and SSL termination. Instead of each microservice implementing these independently, the gateway handles them centrally. Examples: Kong, AWS API Gateway, Nginx, Traefik. Think of it as the reception desk of a building — all visitors check in through one place.

**Q25: How do you handle long-running operations in a REST API?**
Return 202 Accepted immediately with a job ID and a status URL. The client polls the status URL to check progress. When complete, the response includes the result or a download URL. Alternatively, use webhooks to notify the client when the job finishes. Never hold an HTTP connection open for minutes — timeouts, load balancers, and proxies will kill it.

**Q26: What is the difference between authentication and authorization?**
Authentication verifies identity — "who are you?" (username/password, JWT, OAuth). Authorization verifies permissions — "what can you do?" (RBAC, ABAC, ACLs). Authentication comes first; you can't check permissions if you don't know who the person is. Authentication failure returns 401. Authorization failure returns 403. Both are required for a secure API.

**Q27: Explain OAuth 2.0 and its flows.**
OAuth 2.0 is a delegation protocol — it lets users grant limited access to their data on one service to another service without sharing their password. The Authorization Code flow (most secure) has the user log in at the auth provider, get a code, exchange it for tokens server-side. The Implicit flow was for SPAs but is now deprecated. Client Credentials flow is for server-to-server communication. Resource Owner Password flow sends credentials directly (discouraged). PKCE extension secures the Authorization Code flow for mobile and SPA apps.

**Q28: What is JWT? How does it work? What are its limitations?**
JWT (JSON Web Token) is a self-contained token with three parts: header (algorithm), payload (claims/data), and signature (verification). The server creates the JWT after login and sends it to the client. The client includes it in every request. The server verifies the signature without a database lookup — this is why JWTs are stateless and scalable. Limitations: can't be revoked until expiry (unlike sessions), payload is encoded not encrypted (anyone can read it), size grows with claims, and token theft gives full access until expiry.

**Q29: How do you handle API rate limiting at scale?**
Use a distributed rate limiter backed by Redis (sliding window or token bucket algorithm). Rate limit by API key, user ID, and IP address. Apply different limits for different endpoints (writes stricter than reads). Return proper 429 responses with Retry-After headers. Communicate limits clearly in documentation. Use an API Gateway to handle rate limiting centrally. Consider tiered pricing with different rate limits.

**Q30: What is an idempotency key and how do you implement it?**
An idempotency key is a unique string the client sends with non-idempotent requests (like POST) via a header. The server stores the key with the response. If the same key is seen again, the server returns the stored response without re-processing. Store keys in Redis with a TTL (24 hours typically). This prevents duplicate charges, duplicate orders, and duplicate records when clients retry failed requests. Stripe pioneered this pattern for their payment API.

---

### Scenario-Based Questions

**Q31: Design an API for a social media platform (like Twitter).**

```
Authentication:
POST   /auth/register                  # Sign up
POST   /auth/login                     # Get tokens
POST   /auth/refresh                   # Refresh access token

Users:
GET    /users/:username                # Profile
PATCH  /users/me                       # Update own profile
POST   /users/:id/follow               # Follow
DELETE /users/:id/follow               # Unfollow
GET    /users/:id/followers             # Followers list
GET    /users/:id/following             # Following list

Posts (Tweets):
POST   /posts                           # Create
GET    /posts/:id                       # Get single
DELETE /posts/:id                       # Delete own
GET    /users/:id/posts                 # User's posts

Engagement:
POST   /posts/:id/likes                 # Like
DELETE /posts/:id/likes                 # Unlike
POST   /posts/:id/retweets              # Retweet
POST   /posts/:id/comments              # Reply
GET    /posts/:id/comments              # Get replies

Feed:
GET    /feed                            # Timeline (cursor-paginated)
GET    /explore                         # Trending

Search:
GET    /search/users?q=ravi             # Search users
GET    /search/posts?q=docker&sort=-likes  # Search posts
```

Use cursor-based pagination for feeds. Implement rate limiting per user. Cache trending/explore endpoints. Use WebSockets or SSE for real-time notifications.

**Q32: You're getting reports that your API is slow. How do you diagnose and fix it?**
First, add timing middleware to identify slow endpoints. Check database query performance (slow queries, missing indexes, N+1 queries). Add database indexing. Implement response caching (Redis) for frequently accessed, rarely changing data. Check for unnecessary data fetching (return only needed fields). Add connection pooling. Consider pagination for large result sets. Use async processing for heavy operations. Profile with APM tools. Check if the bottleneck is the database, network, or CPU.

**Q33: How would you design an API for an e-commerce checkout flow?**
The checkout flow involves: getting cart, applying discounts, calculating shipping, creating a payment intent, processing payment, and creating the order. Make it idempotent — if the client retries, they shouldn't be charged twice. Use the async pattern for payment processing. Validate stock availability at checkout time, not cart time. Use optimistic locking to prevent race conditions on inventory. Return clear error messages at each step.

**Q34: How do you handle breaking changes in your API?**
First, define what's breaking: removing fields, renaming fields, changing types, removing endpoints. Use API versioning (v1, v2). Deprecate the old version with Sunset headers and documentation. Provide a migration guide. Give clients 6-12 months to migrate. Monitor old version usage. Add new fields (non-breaking) to the existing version. Communicate changes through changelogs, emails, and developer portals.

**Q35: Design a GraphQL schema for a blog platform.**

```graphql
type User {
  id: ID!
  username: String!
  displayName: String!
  bio: String
  avatar: String
  posts(first: Int, after: String): PostConnection!
  followers: Int!
  following: Int!
}

type Post {
  id: ID!
  title: String!
  content: String!
  excerpt: String!
  slug: String!
  author: User!
  tags: [Tag!]!
  comments(first: Int, after: String): CommentConnection!
  likes: Int!
  isLikedByMe: Boolean!
  publishedAt: String
  readTime: Int!
}

type Comment {
  id: ID!
  text: String!
  author: User!
  post: Post!
  replies: [Comment!]!
  createdAt: String!
}

type Tag {
  id: ID!
  name: String!
  postCount: Int!
}

type Query {
  post(slug: String!): Post
  posts(tag: String, sortBy: PostSortBy): PostConnection!
  user(username: String!): User
  feed: PostConnection!
  trending: [Post!]!
  searchPosts(query: String!): [Post!]!
}

type Mutation {
  createPost(input: CreatePostInput!): Post!
  updatePost(id: ID!, input: UpdatePostInput!): Post!
  deletePost(id: ID!): Boolean!
  likePost(id: ID!): Post!
  unlikePost(id: ID!): Post!
  addComment(postId: ID!, text: String!): Comment!
  followUser(userId: ID!): User!
}

enum PostSortBy {
  RECENT
  POPULAR
  TRENDING
}
```

**Q36: How would you migrate from REST to GraphQL without breaking existing clients?**
Run both simultaneously — add a /graphql endpoint alongside existing REST endpoints. Use GraphQL as a layer on top of existing REST services (wrap REST endpoints in resolvers). Migrate frontend clients gradually — start with new features in GraphQL. Keep REST for external/public API consumers. Eventually deprecate REST endpoints with low usage. The key is coexistence, not a big bang switchover.

**Q37: How do you test APIs?**
Unit test individual route handlers/resolvers with mocked dependencies. Integration test the API with a real database (use Docker containers). Test happy paths and error paths. Test authentication and authorization (can user A access user B's data?). Test pagination edge cases (empty results, last page). Test validation (missing fields, wrong types, boundary values). Use tools like Supertest, Postman, or REST Client. In CI/CD, run the full test suite against a clean database.

**Q38: How do you handle concurrent requests modifying the same resource?**
Use optimistic locking — include a version number or ETag in the resource. When updating, the client sends the version they read. If the version has changed (someone else updated it), return 409 Conflict. The client reads the latest version and retries. In the database, use a version column: `UPDATE products SET price = 100 WHERE id = 42 AND version = 3`. If no rows are affected, the version changed.

**Q39: How do you design APIs for mobile clients specifically?**
Minimise payload size (return only needed fields, or use GraphQL). Support partial responses (?fields=name,price). Use cursor-based pagination for infinite scroll. Compress responses (gzip/brotli). Support offline mode with sync endpoints. Use ETags for efficient caching. Consider battery impact — batch requests, reduce polling. Support different image sizes. Handle slow/flaky networks gracefully with timeout configs and retry logic.

**Q40: How do you monitor API health in production?**
Track request rate, error rate, and latency (the "RED" metrics). Set up alerts for error rate spikes and latency increases. Monitor per-endpoint performance to find slow routes. Track 4xx vs 5xx error rates separately (client errors vs server errors). Use distributed tracing (request IDs) to debug across services. Monitor database query performance. Track API usage patterns for capacity planning. Tools: Datadog, New Relic, Prometheus + Grafana, AWS CloudWatch.

---

### GraphQL-Specific Questions

**Q41: What are the main components of a GraphQL schema?**
Types (define data shapes — User, Post, etc.), Queries (read operations), Mutations (write operations), Subscriptions (real-time events), Input Types (structured input for mutations), Enums (fixed set of values), Interfaces (shared fields), and Unions (one-of types). The schema is the contract between client and server — it defines every possible operation and the shape of every response.

**Q42: What is the difference between a Query and a Mutation in GraphQL?**
Queries are for reading data (analogous to GET in REST). Mutations are for writing data — creating, updating, or deleting (analogous to POST/PUT/DELETE). Semantically, queries should have no side effects, while mutations explicitly change server state. Queries can be executed in parallel; mutations execute sequentially.

**Q43: How do you handle authentication in GraphQL?**
Through the context. When a request arrives, extract the token from the Authorization header, verify it, and attach the user to the context object. Every resolver receives the context and can check authentication/authorization. For finer control, use directive-based authorization (@auth, @hasRole) that decorates schema fields.

**Q44: What is the N+1 problem in GraphQL and how do you solve it?**
When fetching a list of items with nested fields, each nested field triggers a separate database query. 20 posts with authors = 1 + 20 queries. The solution is Facebook's DataLoader library — it batches and caches database calls. Instead of 20 individual author queries, DataLoader collects all author IDs and makes one query: SELECT * FROM users WHERE id IN (1,2,3,...,20). It also deduplicates — if the same author wrote multiple posts, they're fetched once.

**Q45: How does GraphQL handle errors differently from REST?**
GraphQL always returns HTTP 200. Errors are in the response body alongside partial data. The response has "data" and "errors" fields — both can be present simultaneously (partial success). Each error includes a message, path (which field failed), locations (where in the query), and optional extensions (error codes, status codes). This is different from REST where HTTP status codes carry the error semantics.

**Q46: What are GraphQL fragments and why use them?**
Fragments are reusable sets of fields. Instead of repeating the same fields in multiple queries, define a fragment and spread it. They reduce query duplication and make queries maintainable. Example: `fragment UserBasic on User { id, name, avatar }` can be used as `...UserBasic` in any query.

**Q47: What is schema stitching vs federation?**
Both combine multiple GraphQL schemas into one. Schema stitching manually merges schemas — simpler but harder to maintain. Apollo Federation lets each service own its part of the schema and extend types from other services. Federation is the modern standard for microservices: the Users service defines User, the Orders service extends User with an orders field. An API Gateway merges everything.

**Q48: How do you handle pagination in GraphQL?**
The Relay connection specification is the standard. Use edges (containing node + cursor), pageInfo (hasNextPage, hasPreviousPage, startCursor, endCursor), and totalCount. Arguments are first/last (count) and before/after (cursor). This gives clients everything needed for both forward and backward pagination. It's more verbose than REST pagination but more standardised.

**Q49: What are GraphQL subscriptions and how do they work?**
Subscriptions provide real-time updates over a persistent connection (usually WebSocket). The client subscribes to an event type, and the server pushes data whenever that event occurs. Example: subscribing to new comments on a post. Under the hood, the server uses a Pub/Sub system — when a mutation creates a comment, it publishes an event, and subscribed clients receive it.

**Q50: What are the security concerns specific to GraphQL?**
Query complexity attacks (deeply nested queries consuming server resources), introspection exposure (anyone can see your entire schema), over-fetching sensitive fields, lack of per-field authorization, and batching attacks (sending hundreds of queries in one request). Mitigations: set query depth limits, disable introspection in production, implement field-level authorization, limit query complexity scores, and throttle requests.

---

### Design Thinking Questions

**Q51: How do you decide between a REST API and an RPC-style API?**
REST is resource-oriented — it models your domain as nouns (/users, /orders) with standard verbs (GET, POST). RPC is action-oriented — it models operations as functions (createUser, processPayment). REST is better for CRUD-heavy applications with clear resources. RPC (gRPC) is better for internal service-to-service communication where performance matters and the operations are more action-oriented. Many systems use REST for external APIs and gRPC for internal communication.

**Q52: Should you use UUIDs or auto-increment IDs for API resources?**
Auto-increment IDs (1, 2, 3) are simple, smaller, and database-friendly but they're sequential (users can guess other resource IDs — security issue), they expose your data scale (ID 42 means you have roughly 42 users), and they create problems when merging databases. UUIDs are random, unpredictable, globally unique, and can be generated client-side, but they're longer, worse for database indexing, and harder to communicate verbally. For public APIs, UUIDs are generally safer. For internal systems, either works.

**Q53: How do you handle API deprecation?**
Announce deprecation early (6-12 months before removal). Add Sunset and Deprecation headers to responses. Log and monitor deprecated endpoint usage. Provide a migration guide with examples. Email API consumers directly. In GraphQL, mark fields with @deprecated("Use 'displayName' instead"). Never remove without migration path. Some companies keep old versions indefinitely (legacy burden vs. breaking clients).

**Q54: How do you design APIs for backward compatibility?**
Only add, never remove or rename. New fields should be optional with defaults. Don't change field types. Don't change response structure. Use loose validation — accept extra fields in request bodies. Version aggressively when you must break compatibility. Test new versions against old client contracts. Use contract testing (Pact) to verify backward compatibility automatically.

**Q55: How do you handle partial failures in a bulk API operation?**
Return 200 (not 500) with per-item results. Each item has its own status (success/failure), error details, and the original request data for context. Include a summary (totalRequested, succeeded, failed). Let the client decide how to handle failures — retry only the failed ones. This is how Stripe, AWS, and most bulk APIs work. Never make it all-or-nothing for bulk operations unless atomicity is a business requirement.

**Q56: How do you design a search API?**
Support full-text search (?q=keyword), exact field filtering (?status=active), range filtering (?min_price=100), sorting (?sort=-relevance,price), and pagination. Consider faceted search (return available filters with counts). For autocomplete, use a dedicated lightweight endpoint. Use Elasticsearch or a similar search engine behind the scenes. Support fuzzy matching for typo tolerance. Return highlighted matches. Rate limit search endpoints aggressively.

**Q57: How do you handle time zones in APIs?**
Store everything in UTC in the database. Return all timestamps in ISO 8601 format with the Z suffix (2024-01-15T10:30:00Z) indicating UTC. Let clients convert to local time for display. Accept timestamps in any timezone (parse the offset) but normalise to UTC. Never store local times without timezone information. Include timezone in user preferences if needed for server-side formatting.

**Q58: How do you design an API for multi-tenant SaaS?**
Tenant identification: subdomain-based (tenant1.api.com), header-based (X-Tenant-Id), or path-based (/api/v1/tenants/t1/users). Data isolation: separate databases (strongest), shared database with separate schemas, or shared tables with a tenant_id column (simplest). Every query must filter by tenant_id. Middleware extracts and validates the tenant on every request. Rate limiting is per-tenant. Never expose data across tenants — this is the #1 security concern.

**Q59: How do you handle API request validation?**
Validate at multiple layers: schema validation (is it valid JSON? does it match the expected shape?), business validation (is the email unique? does the referenced order exist?), and authorization validation (can this user create resources in this account?). Use validation libraries (Joi, Zod, class-validator). Return all validation errors at once (not one at a time). Include the field name, the rejected value, and a human-readable message for each error.

**Q60: How would you design a notification API that supports email, SMS, and push?**
Abstract the channel from the request. The client sends what to notify, who, and optionally which channel. The server decides the channel based on user preferences. Use a queue (RabbitMQ, SQS) to process notifications asynchronously. Support templates with variable substitution. Track delivery status. Allow users to manage notification preferences via API. Implement retry with exponential backoff. Rate limit to prevent spam.

---

### Quick-Fire Round

**Q61: What does CRUD stand for?**
Create, Read, Update, Delete — the four basic operations mapped to POST, GET, PUT/PATCH, DELETE.

**Q62: What is the Accept header used for?**
Tells the server what content type the client wants back (e.g., application/json, text/html).

**Q63: What is the Content-Type header used for?**
Tells the server what format the request body is in (e.g., application/json, multipart/form-data).

**Q64: Can you use GET with a request body?**
Technically the HTTP spec doesn't forbid it, but it's strongly discouraged. Many clients, proxies, and servers ignore or strip the body on GET requests. Use POST if you need to send a body. Elasticsearch historically used GET with a body for its search API but now also supports POST.

**Q65: What is the Richardson Maturity Model?**
A model that grades REST APIs from Level 0 to Level 3. Level 0: one URL, one method (RPC over HTTP). Level 1: multiple URLs for different resources. Level 2: proper HTTP verbs and status codes. Level 3: HATEOAS with hypermedia links. Most production APIs are at Level 2.

**Q66: What is the difference between a resource and a representation?**
A resource is the concept (the user entity). A representation is how that concept is expressed in a specific format (JSON, XML, HTML). The same resource can have multiple representations — the user as JSON for the API, as HTML for a web page, as CSV for an export.

**Q67: What is a safe HTTP method?**
A method that doesn't modify server state. GET, HEAD, and OPTIONS are safe. They're read-only. You can call them any number of times without consequences. POST, PUT, PATCH, and DELETE are unsafe — they modify data.

**Q68: What is ETag?**
An "Entity Tag" — a hash or version identifier for a specific version of a resource. Used for caching and conditional requests. If the ETag hasn't changed, the server returns 304 Not Modified instead of resending the data.

**Q69: What is JSONP? Is it still used?**
JSON with Padding — a historical workaround for cross-origin requests before CORS existed. It wraps JSON in a callback function and loads it via a script tag. It's outdated and insecure. Use CORS instead.

**Q70: What is a HAL link?**
Hypertext Application Language — a standard for adding hypermedia links to JSON responses. It uses `_links` for related URLs and `_embedded` for nested resources. It's one implementation of HATEOAS.

**Q71: What is the OpenAPI specification?**
A standard format (YAML/JSON) for describing REST APIs — endpoints, methods, parameters, request/response schemas, authentication, and examples. Tools like Swagger UI generate interactive documentation from it. It's the industry standard for API documentation.

**Q72: What is gRPC and how does it differ from REST?**
gRPC is a high-performance RPC framework from Google that uses Protocol Buffers (binary format) instead of JSON and HTTP/2 instead of HTTP/1.1. It's faster, supports streaming, and has strong typing via .proto files. Used for internal service-to-service communication. REST is better for public/external APIs because of broader tooling, browser support, and simplicity.

**Q73: What is API throttling vs rate limiting?**
Rate limiting sets a hard cap (max 100 requests/minute). Throttling slows down requests progressively — as you approach the limit, responses get delayed. Rate limiting rejects excess requests with 429. Throttling queues them. Many systems combine both.

**Q74: What is CSRF and how do APIs prevent it?**
Cross-Site Request Forgery tricks a user's browser into making unwanted requests to your API using their cookies. Prevention: use anti-CSRF tokens in forms, validate the Origin/Referer headers, use SameSite cookies, or use token-based auth (JWT in headers) which isn't vulnerable to CSRF since browsers don't auto-send custom headers.

**Q75: What is the difference between cookies and tokens for API auth?**
Cookies are automatically sent by the browser with every request — convenient but vulnerable to CSRF. Tokens (JWT) are manually attached to requests via headers — more work but immune to CSRF and work across domains and non-browser clients. Cookies are stateful (usually tied to a session). Tokens are stateless. For APIs consumed by mobile apps or third parties, tokens are standard.

**Q76: What is API-first design?**
Designing the API before writing any implementation code. You define the contract (OpenAPI spec, GraphQL schema) first, get feedback from consumers, iterate on the design, then implement. It leads to better APIs because design decisions aren't driven by implementation constraints. Teams can work in parallel — frontend uses the spec to build against mocks while backend implements.

**Q77: What is a BFF (Backend for Frontend)?**
A pattern where you build a separate API layer for each frontend type (web BFF, mobile BFF, admin BFF). Each BFF aggregates and transforms data from microservices into exactly what its frontend needs. This prevents generic APIs that over-serve some clients and under-serve others. GraphQL can sometimes replace the need for BFFs.

**Q78: What is API composition?**
Combining data from multiple microservices into a single response. Example: an order detail page needs data from the User service, Order service, Product service, and Shipping service. The API Gateway or BFF makes parallel calls, merges the results, and returns one response. Challenges: handling partial failures, maintaining performance, and managing data consistency.

**Q79: What is contract testing?**
Testing that the API contract between a consumer (frontend) and provider (backend) is maintained. The consumer defines its expectations (what it calls, what it expects back), the provider verifies it meets those expectations. Tools like Pact automate this. It catches breaking changes before they reach production, without running full integration tests.

**Q80: What would you change about an API you've worked on?**
This is an open-ended question interviewers love. Talk about a real API you built. Mention specific improvements: better error handling, adding pagination you forgot initially, switching from PUT to PATCH, adding rate limiting, improving documentation, better naming conventions, adding versioning, or replacing polling with webhooks. Show that you've learned from real experience and can critically evaluate your own work.

---

> **Final Advice for Interviews:**
>
> API design interviews test your ability to think systematically about trade-offs, not memorise specifications. When asked to design an API, always start with: "What are the main resources?", "Who are the consumers?", "What are the most common operations?", and "What are the performance constraints?" Then build from there, discussing the trade-offs of each decision. Show that you understand *why* things are done a certain way, not just *how*.
