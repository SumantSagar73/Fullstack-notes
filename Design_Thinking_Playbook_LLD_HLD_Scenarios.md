# The Design Thinking Playbook — Scenario-Based LLD & HLD Questions with Full Solutions

> **What is this?** You've read the notes. You know what Strategy pattern is. You know what sharding is. But when someone says "Design a food delivery backend" — your mind goes blank. This document fixes that. It teaches you HOW to think, gives you frameworks, and walks through 30+ full scenario designs with detailed step-by-step reasoning.

---

## Table of Contents

**Part 1 — How to Think**
1. [The LLD Thinking Framework — From Problem to Pattern](#1-the-lld-thinking-framework)
2. [The HLD Thinking Framework — From Requirement to Architecture](#2-the-hld-thinking-framework)

**Part 2 — LLD Scenario Designs (15 Full Solutions)**
3. [Design a Payment Processing System](#3-design-a-payment-processing-system)
4. [Design a Food Delivery Order System](#4-design-a-food-delivery-order-system)
5. [Design a Notification Engine](#5-design-a-notification-engine)
6. [Design a File Export Service](#6-design-a-file-export-service)
7. [Design a Coupon/Discount System](#7-design-a-coupondiscount-system)
8. [Design a Permission/Authorization System](#8-design-a-permissionauthorization-system)
9. [Design a Logging Framework](#9-design-a-logging-framework)
10. [Design a Task Scheduler / Job Queue](#10-design-a-task-scheduler--job-queue)
11. [Design a Shopping Cart](#11-design-a-shopping-cart)
12. [Design an Email Template Engine](#12-design-an-email-template-engine)
13. [Design a Rate Limiter (Code Level)](#13-design-a-rate-limiter-code-level)
14. [Design a Plugin System](#14-design-a-plugin-system)
15. [Design a Webhook Delivery System](#15-design-a-webhook-delivery-system)
16. [Design an Audit Trail System](#16-design-an-audit-trail-system)
17. [Design a Multi-Tenant Configuration System](#17-design-a-multi-tenant-configuration-system)

**Part 3 — HLD Scenario Designs (15 Full Solutions)**
18. [Design a URL Shortener at Scale](#18-design-a-url-shortener-at-scale)
19. [Design WhatsApp / Chat System](#19-design-whatsapp--chat-system)
20. [Design Instagram News Feed](#20-design-instagram-news-feed)
21. [Design Uber / Ride Sharing](#21-design-uber--ride-sharing)
22. [Design Netflix / Video Streaming](#22-design-netflix--video-streaming)
23. [Design Google Docs / Collaborative Editing](#23-design-google-docs--collaborative-editing)
24. [Design Twitter Search](#24-design-twitter-search)
25. [Design a Ticket Booking System (BookMyShow)](#25-design-a-ticket-booking-system-bookmyshow)
26. [Design Dropbox / File Sync](#26-design-dropbox--file-sync)
27. [Design a Distributed Cache (like Redis)](#27-design-a-distributed-cache-like-redis)
28. [Design a Food Delivery System (Swiggy/Zomato)](#28-design-a-food-delivery-system-swiggyzomato)
29. [Design a Payment Gateway (like Razorpay)](#29-design-a-payment-gateway-like-razorpay)
30. [Design an Analytics / Event Tracking System](#30-design-an-analytics--event-tracking-system)
31. [Design a Notification System at Scale](#31-design-a-notification-system-at-scale)
32. [Design an E-Commerce Flash Sale System](#32-design-an-e-commerce-flash-sale-system)

---

# PART 1 — HOW TO THINK

---

## 1. The LLD Thinking Framework

When someone asks you to design a system at the code level, follow this process. Don't jump to patterns — start from the problem.

### Step 1: List the Core Actions (Verbs)

What does the system DO? Write down every action.

```
Example: "Design a notification system"

Actions:
- Send a notification
- Choose which channel (email, SMS, push)
- Retry if sending fails
- Check user preferences (opted in?)
- Log every notification sent
- Rate limit notifications (don't spam)
```

### Step 2: List the Core Entities (Nouns)

What are the things involved?

```
Entities:
- Notification
- User
- Channel (email, SMS, push, WhatsApp)
- Template
- DeliveryResult
- UserPreference
```

### Step 3: Identify What Changes vs What Stays Stable

This is the most important step — it tells you where patterns are needed.

```
What changes (likely to grow/modify):
- New channels will be added (Telegram, Slack) ← STRATEGY or FACTORY
- New notification types will be added ← TEMPLATE METHOD
- Retry logic might change ← STRATEGY

What stays stable:
- The overall flow: validate → check preferences → choose channel → send → log
- User preferences structure
```

### Step 4: Spot the Code Smells in Your Mental Design

Ask yourself:

| Question | If YES → | Pattern |
|----------|----------|---------|
| Am I writing if-else to decide WHICH type to create? | Growing conditional | Factory |
| Am I writing if-else to decide HOW to do something? | Multiple algorithms | Strategy |
| Am I writing if-else based on the object's STATUS? | State transitions | State |
| Does one event trigger MANY independent reactions? | Multiple listeners | Observer |
| Do I need to ADD behavior without changing existing code? | Layered behavior | Decorator |
| Is the constructor getting 10+ parameters? | Complex construction | Builder |
| Are multiple subsystems needed for one operation? | Complex orchestration | Facade |
| Is the third-party API different from my interface? | Incompatible interface | Adapter |

### Step 5: Design the Classes and Their Relationships

Draw it out: which classes exist, who depends on whom, what interfaces connect them.

### Step 6: Validate Against SOLID

Run through each principle mentally:
- **S:** Does each class have exactly one reason to change?
- **O:** Can I add a new channel/type without modifying existing classes?
- **L:** Can any child class be used wherever the parent is expected?
- **I:** Is any class forced to implement methods it doesn't use?
- **D:** Do high-level classes depend on abstractions, not concrete implementations?

---

## 2. The HLD Thinking Framework

### The RESHADE Framework

Use this for every system design problem:

| Letter | Step | What to Do |
|--------|------|-----------|
| **R** | Requirements | Functional (what it does) + Non-functional (scale, latency, availability) |
| **E** | Estimation | Users, QPS, storage, bandwidth — back of envelope math |
| **S** | Storage | Database choice, schema, read/write patterns |
| **H** | High-level design | Draw boxes: clients, servers, databases, caches, queues |
| **A** | API design | Key endpoints, request/response formats |
| **D** | Deep dive | The hardest component — how does it actually work? |
| **E** | Edge cases & Bottlenecks | What breaks at scale? Single points of failure? |

### The Decision Tree for HLD Components

```
"Is the data read-heavy?"
  YES → Add cache (Redis) + Read replicas
  NO  → Focus on write optimization

"Does the user need real-time updates?"
  YES → WebSocket or SSE
  NO  → REST/polling is fine

"Is there work that doesn't need immediate response?"
  YES → Message queue (async processing)
  NO  → Synchronous is fine

"Does the system need to handle millions of users?"
  YES → Horizontal scaling + load balancer + CDN
  NO  → Vertical scaling is fine for now

"Is one database not enough?"
  YES → Sharding or multiple databases per service
  NO  → Single database with read replicas

"Do different components scale differently?"
  YES → Consider microservices
  NO  → Monolith or modular monolith

"Does the user upload/download large files?"
  YES → Object storage (S3) + CDN + pre-signed URLs
  NO  → Standard API responses
```

---

# PART 2 — LLD SCENARIO DESIGNS

---

## 3. Design a Payment Processing System

### The Thinking Process

**Step 1 — Actions:** charge, refund, check status, retry failed payments, support multiple methods (card, UPI, wallet, net banking).

**Step 2 — What changes?** Payment methods WILL grow. Each method has a different API (Stripe, Razorpay, Paytm). Retry logic might differ per method.

**Step 3 — Smell detection:** "if method === 'card' ... else if method === 'upi'" → **Strategy + Factory**. "After payment: email + invoice + inventory" → **Observer**. Different payment APIs → **Adapter**.

### The Design

```
                    ┌─────────────────────┐
                    │   PaymentService     │ ← Facade (orchestrates everything)
                    │   (facade)           │
                    └─────────┬───────────┘
                              │
              ┌───────────────┼────────────────┐
              ↓               ↓                ↓
     PaymentGateway    PaymentValidator   EventPublisher
     (interface)       (validates input)  (Observer pattern)
              │                                │
    ┌─────────┼──────────┐           ┌─────────┼──────────┐
    ↓         ↓          ↓           ↓         ↓          ↓
StripeAdapter RazorpayAdapter PaytmAdapter  EmailListener InventoryListener
(Adapter)     (Adapter)       (Adapter)    AnalyticsListener InvoiceListener
```

```javascript
// === INTERFACES ===
class PaymentGateway {
  async charge(amount, currency, paymentDetails) { throw new Error('Not implemented'); }
  async refund(transactionId, amount) { throw new Error('Not implemented'); }
  async getStatus(transactionId) { throw new Error('Not implemented'); }
}

// === ADAPTERS (one per provider) ===
class StripeAdapter extends PaymentGateway {
  async charge(amount, currency, paymentDetails) {
    // Translate OUR interface → STRIPE's interface
    const result = await this.stripeClient.charges.create({
      amount: Math.round(amount * 100), // Stripe uses cents
      currency: currency.toLowerCase(),
      source: paymentDetails.token,
    });
    // Translate STRIPE's response → OUR format
    return {
      transactionId: result.id,
      status: result.status === 'succeeded' ? 'success' : 'failed',
      providerRef: result.id,
    };
  }

  async refund(transactionId, amount) {
    const result = await this.stripeClient.refunds.create({
      charge: transactionId,
      amount: Math.round(amount * 100),
    });
    return { refundId: result.id, status: result.status };
  }
}

class RazorpayAdapter extends PaymentGateway {
  async charge(amount, currency, paymentDetails) {
    // Razorpay has a completely different API — the adapter handles it
    const order = await this.razorpay.orders.create({
      amount: amount * 100,
      currency,
      payment_capture: 1,
    });
    return {
      transactionId: order.id,
      status: order.status === 'paid' ? 'success' : 'pending',
      providerRef: order.id,
    };
  }
}

// === FACTORY (creates the right gateway) ===
class PaymentGatewayFactory {
  constructor(gateways) {
    this.gateways = gateways; // Map<string, PaymentGateway>
  }

  getGateway(method) {
    const gateway = this.gateways.get(method);
    if (!gateway) throw new Error(`Unsupported payment method: ${method}`);
    return gateway;
  }
}

// === OBSERVER (reactions to payment events) ===
class PaymentEventBus {
  constructor() { this.listeners = new Map(); }

  on(event, listener) {
    if (!this.listeners.has(event)) this.listeners.set(event, []);
    this.listeners.get(event).push(listener);
  }

  async emit(event, data) {
    const handlers = this.listeners.get(event) || [];
    await Promise.allSettled(handlers.map(h => h(data)));
  }
}

// === FACADE (the clean entry point) ===
class PaymentService {
  constructor(factory, validator, eventBus, repository) {
    this.factory = factory;
    this.validator = validator;
    this.eventBus = eventBus;
    this.repository = repository;
  }

  async processPayment(paymentRequest) {
    // 1. Validate
    this.validator.validate(paymentRequest);

    // 2. Check idempotency
    const existing = await this.repository.findByIdempotencyKey(paymentRequest.idempotencyKey);
    if (existing) return existing;

    // 3. Get the right gateway
    const gateway = this.factory.getGateway(paymentRequest.method);

    // 4. Charge
    const result = await gateway.charge(
      paymentRequest.amount,
      paymentRequest.currency,
      paymentRequest.details
    );

    // 5. Save to database
    const payment = await this.repository.save({
      ...paymentRequest,
      ...result,
      createdAt: new Date(),
    });

    // 6. Publish event (email, invoice, inventory — all react independently)
    await this.eventBus.emit('payment.completed', payment);

    return payment;
  }
}

// === WIRING ===
const eventBus = new PaymentEventBus();
eventBus.on('payment.completed', async (payment) => emailService.sendReceipt(payment));
eventBus.on('payment.completed', async (payment) => invoiceService.generate(payment));
eventBus.on('payment.completed', async (payment) => analyticsService.track('payment', payment));

const factory = new PaymentGatewayFactory(new Map([
  ['card', new StripeAdapter()],
  ['upi', new RazorpayAdapter()],
  ['wallet', new PaytmAdapter()],
]));

const paymentService = new PaymentService(factory, new PaymentValidator(), eventBus, paymentRepo);
```

### Why Each Pattern?

| Pattern | Why Here | What It Gives You |
|---------|---------|-------------------|
| **Adapter** | Stripe and Razorpay have different APIs | Switch providers without changing business logic |
| **Factory** | Need to pick the right gateway at runtime | Centralised creation logic, easy to add new methods |
| **Observer** | Payment triggers email, invoice, analytics | Add new reactions without touching PaymentService |
| **Facade** | Multiple steps (validate, charge, save, notify) | Clean API for the controller |

### Adding UPI via PhonePe Tomorrow?

1. Create `PhonePeAdapter extends PaymentGateway` — **one new file**
2. Add `['phonePe', new PhonePeAdapter()]` to the factory map — **one line**
3. **Zero changes** to PaymentService, Validator, EventBus, or any listener

That's the Open/Closed Principle in action.

---

## 4. Design a Food Delivery Order System

### Thinking Process

**What changes?** Order states (placed → confirmed → preparing → picked_up → delivered). Many states, each with different allowed actions.

**Smell:** "if status === 'placed' ... else if status === 'preparing'" → **State Pattern**.

### The Design

```javascript
// === STATE INTERFACE ===
class OrderState {
  get name() { throw new Error('Not implemented'); }
  confirm(order) { throw new Error(`Cannot confirm in ${this.name} state`); }
  startPreparing(order) { throw new Error(`Cannot start preparing in ${this.name} state`); }
  pickUp(order) { throw new Error(`Cannot pick up in ${this.name} state`); }
  deliver(order) { throw new Error(`Cannot deliver in ${this.name} state`); }
  cancel(order) { throw new Error(`Cannot cancel in ${this.name} state`); }
}

// === CONCRETE STATES ===
class PlacedState extends OrderState {
  get name() { return 'placed'; }

  confirm(order) {
    order.confirmedAt = new Date();
    order.setState(new ConfirmedState());
    order.emit('order.confirmed', order);
  }

  cancel(order) {
    order.cancelledAt = new Date();
    order.cancelReason = 'cancelled_by_user';
    order.setState(new CancelledState());
    order.emit('order.cancelled', order);
  }
}

class ConfirmedState extends OrderState {
  get name() { return 'confirmed'; }

  startPreparing(order) {
    order.preparingAt = new Date();
    order.setState(new PreparingState());
    order.emit('order.preparing', order);
  }

  cancel(order) {
    // Can cancel in confirmed state but with a penalty
    order.cancelledAt = new Date();
    order.cancelReason = 'cancelled_after_confirmation';
    order.cancellationFee = order.total * 0.1; // 10% fee
    order.setState(new CancelledState());
    order.emit('order.cancelled', order);
  }
}

class PreparingState extends OrderState {
  get name() { return 'preparing'; }

  pickUp(order, driverId) {
    order.driverId = driverId;
    order.pickedUpAt = new Date();
    order.setState(new InTransitState());
    order.emit('order.picked_up', order);
  }

  // Cannot cancel once preparation starts — restaurant already cooking
  cancel(order) {
    throw new Error('Cannot cancel: food is being prepared. Contact support.');
  }
}

class InTransitState extends OrderState {
  get name() { return 'in_transit'; }

  deliver(order) {
    order.deliveredAt = new Date();
    order.setState(new DeliveredState());
    order.emit('order.delivered', order);
  }
}

class DeliveredState extends OrderState {
  get name() { return 'delivered'; }
  // No actions allowed — terminal state
}

class CancelledState extends OrderState {
  get name() { return 'cancelled'; }
  // No actions allowed — terminal state
}

// === THE ORDER ===
class Order extends EventEmitter {
  constructor(data) {
    super();
    this.id = data.id;
    this.userId = data.userId;
    this.restaurantId = data.restaurantId;
    this.items = data.items;
    this.total = data.total;
    this.state = new PlacedState();
  }

  setState(state) { this.state = state; }
  getStatus() { return this.state.name; }

  // Delegate all actions to current state
  confirm() { this.state.confirm(this); }
  startPreparing() { this.state.startPreparing(this); }
  pickUp(driverId) { this.state.pickUp(this, driverId); }
  deliver() { this.state.deliver(this); }
  cancel() { this.state.cancel(this); }
}
```

**Why State Pattern here?** Because the order's behavior fundamentally changes based on its status. A placed order can be cancelled freely. A confirmed order can be cancelled with a fee. A preparing order cannot be cancelled at all. Each state encapsulates its own rules — no giant if-else chains.

---

## 5. Design a Notification Engine

### Thinking: What patterns are needed?

- Multiple channels (email, SMS, push, WhatsApp) → **Strategy**
- Creating the right channel based on type → **Factory**
- Multiple notifications after one event → **Observer**
- Adding retry/logging without modifying senders → **Decorator**
- Same flow (validate → render template → send → log) for all channels → **Template Method**

```javascript
// === TEMPLATE METHOD: The fixed pipeline ===
class NotificationSender {
  async send(notification) {
    this.validate(notification);                              // Step 1
    const rendered = await this.renderTemplate(notification);  // Step 2
    const result = await this.deliver(rendered);               // Step 3
    await this.log(notification, result);                      // Step 4
    return result;
  }

  validate(notification) {
    if (!notification.recipient) throw new Error('Recipient required');
    if (!notification.message) throw new Error('Message required');
  }

  async renderTemplate(notification) {
    if (notification.templateId) {
      return templateEngine.render(notification.templateId, notification.data);
    }
    return notification.message;
  }

  // Subclasses MUST implement this — the part that differs
  async deliver(rendered) { throw new Error('deliver() must be implemented'); }

  async log(notification, result) {
    await notificationLog.save({ ...notification, result, sentAt: new Date() });
  }
}

// === STRATEGY: Each channel implements deliver() differently ===
class EmailSender extends NotificationSender {
  async deliver(rendered) {
    return await sendGridClient.send({
      to: this.currentNotification.recipient,
      subject: rendered.subject,
      html: rendered.body,
    });
  }
}

class SmsSender extends NotificationSender {
  async deliver(rendered) {
    return await twilioClient.messages.create({
      to: this.currentNotification.recipient,
      body: rendered.body,
    });
  }
}

// === DECORATOR: Add retry logic without modifying senders ===
class RetryableNotificationSender {
  constructor(sender, maxRetries = 3) {
    this.sender = sender;
    this.maxRetries = maxRetries;
  }

  async send(notification) {
    for (let attempt = 1; attempt <= this.maxRetries; attempt++) {
      try {
        return await this.sender.send(notification);
      } catch (error) {
        if (attempt === this.maxRetries) throw error;
        await sleep(Math.pow(2, attempt) * 1000); // Exponential backoff
      }
    }
  }
}

// === USAGE ===
const emailSender = new RetryableNotificationSender(new EmailSender(), 3);
const smsSender = new RetryableNotificationSender(new SmsSender(), 2);
```

---

## 6. Design a File Export Service

**Thinking:** Multiple formats (PDF, CSV, Excel, JSON) → **Strategy + Factory**. Complex export object (filters, columns, date range, format) → **Builder**.

```javascript
class ExportRequestBuilder {
  constructor(dataSource) {
    this.request = { dataSource, columns: [], filters: {} };
  }

  columns(...cols) { this.request.columns = cols; return this; }
  dateRange(from, to) { this.request.filters.dateRange = { from, to }; return this; }
  format(fmt) { this.request.format = fmt; return this; }
  fileName(name) { this.request.fileName = name; return this; }

  build() {
    if (!this.request.format) throw new Error('Format is required');
    return this.request;
  }
}

// Usage
const exportReq = new ExportRequestBuilder('orders')
  .columns('orderId', 'customerName', 'total', 'status')
  .dateRange('2024-01-01', '2024-06-30')
  .format('csv')
  .fileName('orders-q1-q2')
  .build();

const exporter = ExporterFactory.create(exportReq.format);
const file = await exporter.export(exportReq);
```

---

## 7. Design a Coupon/Discount System

**Thinking:** Multiple discount types (percentage, flat, buy-one-get-one, free shipping) → **Strategy**. Multiple rules applied in sequence (minimum order, first-time user, category restriction) → **Chain of Responsibility**. Discounts can stack → **Composite**.

```javascript
// === STRATEGY: Different discount calculations ===
class DiscountStrategy {
  calculate(order) { throw new Error('Not implemented'); }
}

class PercentageDiscount extends DiscountStrategy {
  constructor(percent) { super(); this.percent = percent; }
  calculate(order) { return order.subtotal * (this.percent / 100); }
}

class FlatDiscount extends DiscountStrategy {
  constructor(amount) { super(); this.amount = amount; }
  calculate(order) { return Math.min(this.amount, order.subtotal); }
}

class BuyOneGetOneFree extends DiscountStrategy {
  calculate(order) {
    const cheapestItem = Math.min(...order.items.map(i => i.price));
    return cheapestItem;
  }
}

// === CHAIN OF RESPONSIBILITY: Validation rules ===
class CouponRule {
  constructor(next = null) { this.next = next; }

  validate(coupon, order) {
    const result = this.check(coupon, order);
    if (!result.valid) return result;
    if (this.next) return this.next.validate(coupon, order);
    return { valid: true };
  }

  check(coupon, order) { throw new Error('Not implemented'); }
}

class MinimumOrderRule extends CouponRule {
  check(coupon, order) {
    if (coupon.minOrder && order.subtotal < coupon.minOrder) {
      return { valid: false, reason: `Minimum order ₹${coupon.minOrder} required` };
    }
    return { valid: true };
  }
}

class ExpiryRule extends CouponRule {
  check(coupon, order) {
    if (coupon.expiresAt && new Date() > coupon.expiresAt) {
      return { valid: false, reason: 'Coupon has expired' };
    }
    return { valid: true };
  }
}

class UsageLimitRule extends CouponRule {
  check(coupon, order) {
    if (coupon.usageCount >= coupon.maxUsage) {
      return { valid: false, reason: 'Coupon usage limit reached' };
    }
    return { valid: true };
  }
}

// === WIRING: Build the validation chain ===
const validationChain = new MinimumOrderRule(
  new ExpiryRule(
    new UsageLimitRule()
  )
);

// Apply coupon
function applyCoupon(coupon, order) {
  const validation = validationChain.validate(coupon, order);
  if (!validation.valid) throw new Error(validation.reason);

  const strategy = DiscountStrategyFactory.create(coupon.type, coupon.value);
  const discount = strategy.calculate(order);

  return { discount, finalTotal: order.subtotal - discount };
}
```

---

## 8. Design a Permission/Authorization System

**Thinking:** Users have roles, roles have permissions. Need to check permissions at various levels. **Composite** (groups contain users and other groups). **Strategy** (different permission check strategies: RBAC, ABAC).

```javascript
// Role-Based Access Control (RBAC)
class PermissionService {
  constructor(roleRepository) {
    this.roleRepo = roleRepository;
  }

  async hasPermission(userId, resource, action) {
    const userRoles = await this.roleRepo.getRolesForUser(userId);
    const permissions = await this.roleRepo.getPermissionsForRoles(userRoles);

    return permissions.some(p =>
      p.resource === resource && (p.action === action || p.action === '*')
    );
  }
}

// Middleware
function authorize(resource, action) {
  return async (req, res, next) => {
    const allowed = await permissionService.hasPermission(req.user.id, resource, action);
    if (!allowed) return res.status(403).json({ error: 'Insufficient permissions' });
    next();
  };
}

// Usage
app.delete('/api/users/:id', authenticate, authorize('users', 'delete'), deleteUser);
app.get('/api/reports', authenticate, authorize('reports', 'read'), getReports);
```

---

## 9-17. Quick-Fire LLD Designs

### 9. Logging Framework
**Patterns:** Strategy (output to console/file/cloud), Decorator (add timestamps, add context), Singleton (one logger instance). Chain of Responsibility (log level filtering: debug → info → warn → error).

### 10. Task Scheduler / Job Queue
**Patterns:** Command (each job is a command object with execute/retry/undo), Strategy (different scheduling strategies: immediate, delayed, cron), Observer (notify on completion/failure), State (job states: queued → running → completed/failed).

### 11. Shopping Cart
**Patterns:** Observer (cart changes → update price display, update inventory holds, update recommendations), Strategy (pricing strategies: regular, member, wholesale), Composite (cart contains items, bundles, and sub-carts/wishlists).

### 12. Email Template Engine
**Patterns:** Template Method (load template → replace variables → add header/footer → send), Strategy (different renderers: HTML, plain text, AMP), Builder (build email step by step: to, subject, template, variables, attachments).

### 13. Rate Limiter (Code Level)
**Patterns:** Strategy (different algorithms: token bucket, sliding window, fixed window), Decorator (wrap any service with rate limiting), Singleton (shared counter).

### 14. Plugin System
**Patterns:** Observer (plugins subscribe to lifecycle events), Strategy (plugins provide different implementations), Factory (plugin loader creates instances from config).

### 15. Webhook Delivery System
**Patterns:** Command (each webhook delivery is a command), Strategy (retry strategies: linear, exponential), Observer (delivery events: sent, failed, exhausted).

### 16. Audit Trail System
**Patterns:** Observer (listen to all entity changes), Command (each change is a logged command), Decorator (wrap repositories with audit logging).

### 17. Multi-Tenant Configuration System
**Patterns:** Strategy (different config sources: database, file, environment), Proxy (tenant-aware config proxy that adds tenant filtering), Prototype (clone base config and override per tenant).

---

# PART 3 — HLD SCENARIO DESIGNS

---

## 18. Design a URL Shortener at Scale

### Step 1: Requirements
- **Functional:** Create short URL, redirect to original, optional analytics, optional custom aliases, optional expiry.
- **Non-functional:** 100M URLs/month, 10:1 read/write ratio, < 100ms redirect latency, 99.99% availability.

### Step 2: Estimation
```
Writes: 100M / month ≈ 40 writes/sec
Reads:  1B / month ≈ 400 reads/sec
Storage: 100M × 500 bytes × 5 years = ~300 GB
Short code: 7 chars of Base62 = 62^7 = 3.5 trillion combinations (more than enough)
```

### Step 3: High-Level Architecture
```
Client → DNS → Load Balancer → App Servers → Redis Cache (hot URLs)
                                            → PostgreSQL (all URLs)

Write Flow:
1. Generate unique short code (Base62 of auto-increment from Redis counter)
2. Store {shortCode → longURL} in PostgreSQL
3. Cache in Redis
4. Return short URL

Read Flow (redirect):
1. Check Redis cache → HIT → 301 redirect
2. MISS → query PostgreSQL → cache in Redis → 301 redirect
3. Async: increment click counter in Redis, flush to DB periodically
```

### Step 4: Key Design Decisions

**How to generate short codes?**
Use a Redis counter (INCR) → convert to Base62. Fast, unique, no collisions. For multiple app servers, pre-allocate ranges (Server 1 gets IDs 1-10000, Server 2 gets 10001-20000).

**301 vs 302 redirect?**
301 (permanent) — browser caches it, fewer requests to your server but you lose analytics. 302 (temporary) — browser always comes back, more load but you can track every click. Use 302 if analytics matter.

**Where to count clicks?**
Don't write to the database on every click. Increment a counter in Redis, then batch-write to PostgreSQL every minute. This handles 400 reads/sec without killing the database.

### Step 5: Scaling Bottlenecks
- **Database writes:** Batch write click counts, use write-behind cache.
- **Read latency:** Redis cache handles 90%+ of redirects.
- **Storage growth:** Periodically archive expired URLs to cold storage.
- **Single point of failure:** Redis Cluster for cache, PostgreSQL replicas for reads.

---

## 19. Design WhatsApp / Chat System

### Requirements
- 1-on-1 messaging, group chats (up to 256)
- Delivered/read receipts, online status, media sharing
- 500M DAU, each sends 40 messages/day = 20B messages/day

### Architecture
```
┌──────────┐    ┌───────────────┐    ┌──────────────┐
│  Mobile   │←──│   WebSocket    │←──│  Connection   │
│  Client   │──→│   Gateway      │──→│  Registry     │
└──────────┘    │   (Cluster)    │    │  (Redis)      │
                └───────┬───────┘    └──────────────┘
                        │
                ┌───────┴───────┐
                │  Message       │
                │  Router        │
                └───────┬───────┘
                        │
          ┌─────────────┼─────────────┐
          ↓             ↓             ↓
   ┌──────────┐  ┌──────────┐  ┌──────────┐
   │ Message   │  │  Media    │  │  Push     │
   │ Store     │  │  Service  │  │  Service  │
   │(Cassandra)│  │  (S3)     │  │(FCM/APNS)│
   └──────────┘  └──────────┘  └──────────┘
```

### Key Design Decisions

**How are messages delivered in real-time?**
Each client maintains a WebSocket connection to the gateway. The Connection Registry (Redis) maps `userId → gatewayServerId`. When User A sends a message to User B, the router checks which gateway server User B is connected to and routes the message there.

**What if the recipient is offline?**
Store the message in Cassandra with status "sent." When the user comes online, the gateway fetches all undelivered messages. Also trigger a push notification via FCM/APNS.

**How is the message stored?**
Cassandra with partition key = `chatId`, clustering key = `timestamp DESC`. This gives O(1) access to any chat's messages, sorted by time. Each chat's data is on one partition — efficient reads.

**Group messages?**
When User A sends to a group of 100, the Message Router fans out: sends to each online member's WebSocket, queues push notifications for offline members. Store one copy of the message per group (not per member).

**End-to-end encryption?**
The server never sees the plaintext. Keys are exchanged client-to-client using the Signal Protocol. The server stores and routes encrypted blobs. This is a client-side concern but important to mention in interviews.

---

## 20. Design Instagram News Feed

### The Core Challenge

When User A opens their feed, show recent posts from all 500 people they follow, sorted by relevance. At 500M DAU, this must be fast.

### The Hybrid Approach (What Instagram Actually Does)

```
For regular users (< 10K followers):
Post Created → Fan-out Worker → Write to each follower's feed cache (Redis sorted set)
                                 │
                                 ├── Follower 1's feed cache
                                 ├── Follower 2's feed cache
                                 └── Follower N's feed cache

For celebrities (> 10K followers):
Post Created → Just save to Post DB (no fan-out)

When user opens feed:
1. Read pre-computed feed from Redis (posts from regular users they follow)
2. Fetch recent posts from celebrities they follow (on-the-fly)
3. Merge, rank by relevance (ML ranking model)
4. Return top 50 posts
```

### Why Hybrid?

**Pure fan-out on write:** A celebrity with 10M followers posts → 10M cache writes. At 100 celebrity posts/minute, that's 1B writes/minute. Unsustainable.

**Pure fan-out on read:** User opens feed → query posts from all 500 followed users → merge → sort. 500 database queries per feed load. With 500M DAU = 250B queries/day. Too slow.

**Hybrid:** Fan-out on write for the 95% of users with few followers (cheap). Fan-out on read only for celebrities (small number of special cases). Best of both worlds.

---

## 21. Design Uber / Ride Sharing

### Architecture
```
┌───────────┐     ┌───────────────┐     ┌─────────────────┐
│  Rider    │────→│  API Gateway   │────→│  Trip Service    │
│  App      │     │               │     │  (state machine) │
└───────────┘     └───────┬───────┘     └────────┬────────┘
                          │                      │
┌───────────┐     ┌───────┴───────┐     ┌────────┴────────┐
│  Driver   │────→│  Location      │────→│  Matching        │
│  App      │     │  Service       │     │  Service         │
└───────────┘     │  (QuadTree +   │     │  (find nearby    │
                  │   Redis Geo)   │     │   drivers)       │
                  └───────────────┘     └─────────────────┘
                                                │
                                        ┌───────┴───────┐
                                        │  Pricing       │
                                        │  Service       │
                                        │  (surge calc)  │
                                        └───────────────┘
```

### Key: How to Find Nearby Drivers?

Drivers send GPS coordinates every 3-4 seconds. Store in Redis using geospatial commands:

```
GEOADD drivers 77.209 28.613 "driver:101"   -- Delhi coordinates
GEOADD drivers 77.210 28.615 "driver:102"

-- Find all drivers within 3 km radius
GEORADIUS drivers 77.212 28.614 3 km WITHDIST COUNT 10 ASC
-- Returns: driver:102 (0.3 km), driver:101 (0.4 km)
```

### Trip State Machine (LLD Inside HLD)

```
REQUESTED → MATCHING → DRIVER_ASSIGNED → DRIVER_EN_ROUTE → 
ARRIVED → TRIP_STARTED → TRIP_IN_PROGRESS → TRIP_COMPLETED
                    ↓
               CANCELLED
```

Each state has rules: REQUESTED can be cancelled by rider. DRIVER_ASSIGNED can be cancelled with a fee. TRIP_IN_PROGRESS cannot be cancelled.

---

## 22-32. Remaining HLD Designs (Key Decisions Only)

### 22. Netflix / Video Streaming
**Key decisions:** CDN is everything — pre-position popular content at edge locations. Adaptive bitrate streaming (start low, increase quality as bandwidth allows). Transcode each video into 100+ formats on upload (different resolutions, codecs, devices). Recommendation engine uses collaborative filtering. Pre-compute recommendations, don't calculate in real-time.

### 23. Google Docs / Collaborative Editing
**Key decisions:** Operational Transformation (OT) or CRDT for conflict resolution when two users edit the same section simultaneously. WebSocket connections for real-time sync. Every keystroke is an "operation" (insert char at position 5, delete char at position 10). Operations are transformed against concurrent operations to maintain consistency.

### 24. Twitter Search
**Key decisions:** Inverted index (word → list of tweet IDs that contain it). When a tweet is posted, extract keywords and update the index. Search queries find matching tweet IDs from the index, then fetch and rank tweets. Use Elasticsearch for full-text search with relevance scoring. Real-time indexing via Kafka.

### 25. Ticket Booking (BookMyShow)
**Key decisions:** The critical problem is preventing double-booking. Use optimistic locking: when user selects seat, mark it "held" for 10 minutes (Redis with TTL). When they pay, verify the hold is still valid, then confirm. If hold expires, release the seat. For flash sales (thousands booking simultaneously), use a queue — first come, first served.

### 26. Dropbox / File Sync
**Key decisions:** Chunking — split files into 4MB chunks, hash each chunk. Only upload chunks that changed (delta sync). If a chunk already exists (same hash), skip it (deduplication). Use a sync queue for conflict resolution — last writer wins with conflict copies. Metadata service tracks file versions, chunks, and which devices have which versions.

### 27. Distributed Cache (like Redis)
**Key decisions:** Consistent hashing for key distribution across nodes. Each key is stored on 2-3 nodes (replication) for fault tolerance. LRU eviction when memory is full. Client-side library handles routing (hash the key → find the right node). Gossip protocol for cluster membership — nodes discover each other.

### 28. Food Delivery (Swiggy/Zomato)
**Key decisions:** Three-sided marketplace (user, restaurant, delivery partner). Restaurant discovery via geospatial search. ETA calculation combines: restaurant prep time (historical average) + driver travel time (routing API) + current demand. Order assignment: find nearest available delivery partner using geospatial query. Surge pricing based on demand/supply ratio in each zone.

### 29. Payment Gateway (Razorpay)
**Key decisions:** Idempotency keys for every transaction (prevent double charging). Write-ahead log for every state change. Saga pattern for multi-step flows (authorize → capture → settle). PCI-DSS compliance — tokenize card numbers, never store raw. Webhook delivery for merchant notifications with retry and signing. Reconciliation system that matches bank records with internal records daily.

### 30. Analytics / Event Tracking
**Key decisions:** Events arrive at thousands per second — use Kafka for ingestion (handles spikes). Stream processing (Flink or Kafka Streams) for real-time dashboards. Batch processing (Spark) for historical reports. Store raw events in S3 (cheap, durable). Store aggregated metrics in a time-series database (InfluxDB). Pre-compute common queries (daily active users, funnel metrics) — don't calculate on the fly.

### 31. Notification System at Scale
**Key decisions:** Priority queues (password reset > marketing email). Per-user rate limiting (max 5 notifications/hour). Channel preference check before sending. Deduplication (don't send the same notification twice). Template rendering at send time (not at queue time — data might change). Dead letter queue for permanent failures. Delivery tracking (sent, delivered, read, failed).

### 32. E-Commerce Flash Sale
**Key decisions:** The problem is 100K users trying to buy 1000 items at exactly 12:00 PM. Pre-warm cache with product data. Use Redis for inventory counter (DECR is atomic — no overselling). Queue requests — first 1000 get a "purchase token," rest get "sold out." Separate the "claim" from the "checkout" — users have 10 minutes to complete payment after claiming.

```
12:00:00 — Sale starts
  → 100K requests hit load balancer
  → App servers check Redis: DECR stock counter
  → Counter > 0? Issue purchase token, redirect to checkout
  → Counter = 0? Return "Sold Out"

  → Checkout: User has 10 minutes to pay
  → If they don't pay: release token, INCR counter, offer to next in queue
```

---

# HOW TO ANSWER IN AN INTERVIEW

### The Template

**"Let me start with requirements..."** (shows you don't jump to solutions)
- Clarify scope: which features to focus on?
- Clarify scale: users? requests per second?
- Clarify priorities: latency? consistency? cost?

**"Let me do some quick math..."** (shows you think quantitatively)
- Users × actions × data size = storage
- Requests per second = server capacity needed

**"Here's the high-level architecture..."** (draw boxes and arrows)
- Start with the simplest version that works
- Add components as you explain WHY they're needed

**"Let me dive into [hardest part]..."** (shows depth)
- Pick the most interesting/challenging component
- Explain the trade-offs of your choices

**"At scale, the bottleneck would be..."** (shows experience)
- Identify what breaks first
- Explain how to fix it
- Mention monitoring and failure scenarios

### What Interviewers Actually Score

| They're Looking For | How to Show It |
|--------------------|---------------|
| Requirement gathering | Ask 3-5 clarifying questions before designing |
| Back-of-envelope math | Estimate QPS, storage, bandwidth |
| Trade-off awareness | "I chose X over Y because..." |
| Component knowledge | Know when to use cache, queue, CDN, etc. |
| Database choice | Justify SQL vs NoSQL with specific reasons |
| Scaling awareness | Identify bottlenecks and solutions |
| Failure handling | "What if this component fails?" |
| Real-world awareness | Reference how real systems (Twitter, Uber) do it |

---

> **The One Thing That Separates Good from Great Answers:**
>
> A good answer describes an architecture. A great answer explains WHY each decision was made and what happens if you chose differently. "I'm using Redis here because this is read-heavy with 400 reads/sec, and a cache reduces database load by 90%. If we didn't cache, the database would need 5 replicas just for reads." That one sentence shows more engineering maturity than a perfect diagram.
