# Ultimate Low-Level Design (LLD) Notes — From Code That Works to Code That's Beautiful

> **Who is this for?** Backend developers who can write code that works but struggle with: "How do I structure this?", "Which pattern fits here?", "Why does this codebase feel messy?", and "How do I answer LLD interview questions?" After reading this, you'll look at any codebase and immediately see what's wrong and how to fix it.

---

## Table of Contents

**Part 1 — The Foundation**

1. [What Is LLD and Why Should You Care?](#1-what-is-lld-and-why-should-you-care)
2. [Code Smells — How to Spot Problems Before They Rot](#2-code-smells--how-to-spot-problems-before-they-rot)
3. [SOLID Principles — The Five Laws of Good Code](#3-solid-principles--the-five-laws-of-good-code)

**Part 2 — Design Patterns (Creational)**

4. [Singleton — One and Only One](#4-singleton--one-and-only-one)
5. [Factory Method — Let Someone Else Decide](#5-factory-method--let-someone-else-decide)
6. [Abstract Factory — Factory of Factories](#6-abstract-factory--factory-of-factories)
7. [Builder — Step by Step Construction](#7-builder--step-by-step-construction)
8. [Prototype — Clone Instead of Create](#8-prototype--clone-instead-of-create)

**Part 3 — Design Patterns (Structural)**

9. [Adapter — Making Incompatible Things Work Together](#9-adapter--making-incompatible-things-work-together)
10. [Decorator — Adding Features Without Touching Original Code](#10-decorator--adding-features-without-touching-original-code)
11. [Facade — One Simple Door to a Complex Building](#11-facade--one-simple-door-to-a-complex-building)
12. [Proxy — A Gatekeeper That Controls Access](#12-proxy--a-gatekeeper-that-controls-access)
13. [Composite — Treating One and Many the Same Way](#13-composite--treating-one-and-many-the-same-way)

**Part 4 — Design Patterns (Behavioral)**

14. [Strategy — Swappable Algorithms](#14-strategy--swappable-algorithms)
15. [Observer — When Something Happens, Tell Everyone Who Cares](#15-observer--when-something-happens-tell-everyone-who-cares)
16. [Command — Turn Actions into Objects](#16-command--turn-actions-into-objects)
17. [Template Method — Same Steps, Different Details](#17-template-method--same-steps-different-details)
18. [State — Behavior Changes When State Changes](#18-state--behavior-changes-when-state-changes)
19. [Chain of Responsibility — Pass It Along Until Someone Handles It](#19-chain-of-responsibility--pass-it-along-until-someone-handles-it)
20. [Iterator — Walk Through a Collection Without Knowing Its Guts](#20-iterator--walk-through-a-collection-without-knowing-its-guts)
21. [Mediator — Stop Everyone from Talking to Everyone](#21-mediator--stop-everyone-from-talking-to-everyone)

**Part 5 — Putting It All Together**

22. [Clean Architecture & Layered Architecture](#22-clean-architecture--layered-architecture)
23. [Dependency Injection — Don't Create, Receive](#23-dependency-injection--dont-create-receive)
24. [Refactoring — How to Fix Bad Code Without Breaking It](#24-refactoring--how-to-fix-bad-code-without-breaking-it)
25. [How to Spot Which Pattern You Need — The Decision Guide](#25-how-to-spot-which-pattern-you-need--the-decision-guide)
26. [LLD Interview Questions (70+ Questions)](#26-lld-interview-questions-70-questions)

---

# PART 1 — THE FOUNDATION

---

## 1. What Is LLD and Why Should You Care?

### The Difference Between Code That Works and Code That Lasts

You've written code that works. It handles requests, talks to the database, returns responses. But six months later, your manager says "add payment via UPI alongside credit cards" and you realise that payment logic is scattered across 12 files, tangled with email notification code, and every change breaks something else.

**That's the difference between code that works and code that's designed.**

LLD (Low-Level Design) is about how you **structure code inside a service** — classes, functions, modules, their relationships, and their responsibilities. HLD (High-Level Design) is about how services talk to each other (databases, queues, load balancers). LLD is the inside of the building; HLD is the city plan.

### Why It Matters for Backend Developers

| Without LLD | With LLD |
|------------|---------|
| Adding a new payment method requires changing 15 files | Adding a new payment method means creating 1 new class |
| A bug in notifications breaks the order flow | Notifications are independent — a bug there doesn't touch orders |
| New developers take weeks to understand the code | New developers understand the structure in a day |
| Testing requires spinning up the entire app | Each piece can be tested in isolation |
| "Don't touch that file, it'll break everything" | Every file has one clear job |

### The Three Pillars of LLD

1. **SOLID Principles** — The rules of writing good code
2. **Design Patterns** — Proven solutions to common problems
3. **Clean Architecture** — How to organise the big picture inside a service

Think of SOLID as grammar rules, Design Patterns as writing techniques (metaphor, foreshadowing, flashback), and Clean Architecture as the overall structure of a novel (chapters, acts, arc).

---

## 2. Code Smells — How to Spot Problems Before They Rot

A **code smell** isn't a bug. Your code works. But something about it stinks — it's a sign that the design underneath is weak and will cause pain later.

**Analogy:** A code smell is like a weird noise in your car's engine. The car still drives, but that noise means something is wearing out, and if you ignore it, you'll be stranded on the highway in three months.

### Smell 1: God Class / God Function

**What it looks like:** One class or function that does everything — handles HTTP requests, validates data, talks to the database, sends emails, generates PDFs, calculates prices.

```javascript
// THE SMELL — OrderService that does everything
class OrderService {
  async createOrder(req, res) {
    // Validate input (50 lines)
    // Check inventory (30 lines)
    // Calculate price with discounts (40 lines)
    // Process payment via Stripe (60 lines)
    // Save to database (20 lines)
    // Send confirmation email (30 lines)
    // Send SMS notification (20 lines)
    // Update analytics (15 lines)
    // Generate invoice PDF (40 lines)
    // Return response
  }
  // This function is 300+ lines long
}
```

**Why it's a problem:** If you need to change how emails are sent, you're editing a 300-line function that also handles payments. One wrong edit and orders break. You can't test email logic without setting up payment logic, database, and everything else.

**The fix:** Split responsibilities. Each class does one thing (Single Responsibility Principle).

### Smell 2: Shotgun Surgery

**What it looks like:** Adding one feature requires changing many files in many places.

"To add UPI payments, I need to change OrderService, PaymentController, PaymentValidator, InvoiceGenerator, NotificationService, AnalyticsTracker, and the admin dashboard API."

**Why it's a problem:** High chance of missing one place. Changes ripple everywhere. Hard to review in code review because the change is scattered.

**The fix:** Group related logic together. If payment logic is in one place, adding a new payment method means changing one place.

### Smell 3: Feature Envy

**What it looks like:** A method in one class spends most of its time accessing data from another class.

```javascript
// THE SMELL — OrderService reaching deep into Customer
class OrderService {
  calculateDiscount(order) {
    if (customer.getMembership().getTier() === 'gold' &&
        customer.getOrderHistory().length > 10 &&
        customer.getOrderHistory().reduce((sum, o) => sum + o.total, 0) > 50000) {
      return 0.15;
    }
    return 0;
  }
}
```

**The fix:** Move `calculateDiscount` to the Customer class — it's about customer data, not order logic.

```javascript
// FIXED — Customer owns its own logic
class Customer {
  isEligibleForDiscount() {
    return this.membership.tier === 'gold' &&
           this.orderHistory.length > 10 &&
           this.totalSpent() > 50000;
  }

  getDiscountRate() {
    return this.isEligibleForDiscount() ? 0.15 : 0;
  }
}
```

### Smell 4: Primitive Obsession

**What it looks like:** Using raw strings, numbers, and booleans for everything instead of creating meaningful types.

```javascript
// THE SMELL
function createUser(name, email, phone, address, city, state, pincode, role) { ... }

// What's the difference between phone and pincode? Both are strings.
// What if someone passes email in the phone parameter?
createUser("Ravi", "9876543210", "ravi@email.com", ...); // OOPS — swapped email and phone
```

**The fix:** Create value objects / domain classes.

```javascript
// FIXED
class Email {
  constructor(value) {
    if (!value.includes('@')) throw new Error('Invalid email');
    this.value = value;
  }
}

class PhoneNumber {
  constructor(value) {
    if (!/^\d{10}$/.test(value)) throw new Error('Invalid phone');
    this.value = value;
  }
}

class Address {
  constructor(street, city, state, pincode) { ... }
}

function createUser(name, email, phone, address, role) { ... }
// Now you CAN'T accidentally swap email and phone — they're different types
```

### Smell 5: Long Parameter List

**What it looks like:** Functions with 5+ parameters.

```javascript
// THE SMELL
function sendNotification(userId, message, channel, priority, retryCount, delay, template, attachments) { ... }
```

**The fix:** Group related parameters into objects.

```javascript
// FIXED
function sendNotification({ userId, message, options }) {
  const { channel, priority, retryCount, delay, template, attachments } = options;
}

// Or create a Notification class
class Notification {
  constructor(userId, message) { ... }
  viaEmail() { ... }
  withPriority(p) { ... }
  send() { ... }
}
```

### Smell 6: Duplicated Code

**What it looks like:** The same logic copied across multiple places.

**Why it's a problem:** Fix a bug in one copy, forget the other four copies. Now you have inconsistent behavior.

**The fix:** Extract into a shared function, class, or module.

### Smell 7: Switch/If-Else Chains That Keep Growing

**What it looks like:**

```javascript
// THE SMELL — and every new payment method adds another case
function processPayment(type, amount) {
  if (type === 'credit_card') {
    // 30 lines of credit card logic
  } else if (type === 'debit_card') {
    // 25 lines of debit card logic
  } else if (type === 'upi') {
    // 20 lines of UPI logic
  } else if (type === 'net_banking') {
    // 35 lines of net banking logic
  } else if (type === 'wallet') {
    // 15 lines of wallet logic
  }
  // This function grows every time a new method is added
}
```

**Why it's a problem:** The function grows forever. Every new payment method modifies existing, working code. Testing requires covering every branch. This is the #1 smell that design patterns fix (Strategy, Factory).

**The fix:** Use the Strategy pattern (covered in detail later).

### Quick Reference: Smell → Fix

| Smell | Signal | Fix |
|-------|--------|-----|
| God Class | One class does everything | Split into focused classes (SRP) |
| Shotgun Surgery | One change touches many files | Group related logic (high cohesion) |
| Feature Envy | Method uses another class's data more than its own | Move method to that class |
| Primitive Obsession | Raw strings/numbers everywhere | Create value objects |
| Long Parameter List | 5+ function parameters | Parameter objects or Builder |
| Duplicated Code | Same logic in multiple places | Extract shared function/class |
| Switch/If-Else Chains | Growing conditional blocks | Strategy pattern, polymorphism |
| Long Method | Function > 20-30 lines | Extract into smaller methods |
| Dead Code | Unused functions/variables | Delete it (git remembers) |
| Comments Explaining "What" | `// increment counter by 1` | Make code self-documenting |

---

## 3. SOLID Principles — The Five Laws of Good Code

SOLID is an acronym for five principles that guide you toward code that's easy to maintain, extend, and test. They were popularised by Robert C. Martin (Uncle Bob).

**Analogy:** SOLID principles are like traffic rules. You *can* drive without them — but as soon as there's more than one car on the road, chaos begins. These rules keep everything flowing smoothly.

### S — Single Responsibility Principle (SRP)

> **A class should have only one reason to change.**

**Simple English:** Every class (or module or function) should do one thing and do it well. If a class has two responsibilities, changes to one responsibility can break the other.

**Analogy:** A chef cooks food. A waiter serves food. A cashier handles payments. If one person does all three, when the restaurant gets busy, that person becomes the bottleneck. And if they get sick, everything stops.

**The Problem:**

```javascript
// BAD — UserService does three unrelated things
class UserService {
  createUser(data) {
    // Validate data
    if (!data.email.includes('@')) throw new Error('Invalid email');
    if (data.password.length < 8) throw new Error('Password too short');

    // Save to database
    const user = db.query('INSERT INTO users ...');

    // Send welcome email
    const html = `<h1>Welcome ${data.name}!</h1><p>Thanks for joining...</p>`;
    emailClient.send({ to: data.email, subject: 'Welcome!', html });

    return user;
  }
}
```

**Three reasons this class could change:**
1. Validation rules change (e.g., add phone number validation)
2. Database schema changes (e.g., add new columns)
3. Email template changes (e.g., new design, add unsubscribe link)

**The Fix:**

```javascript
// GOOD — each class has one responsibility

class UserValidator {
  validate(data) {
    const errors = [];
    if (!data.email.includes('@')) errors.push('Invalid email');
    if (data.password.length < 8) errors.push('Password too short');
    if (errors.length > 0) throw new ValidationError(errors);
  }
}

class UserRepository {
  async save(userData) {
    return db.query('INSERT INTO users ...', userData);
  }

  async findByEmail(email) {
    return db.query('SELECT * FROM users WHERE email = ?', email);
  }
}

class WelcomeEmailService {
  async send(user) {
    const html = templates.render('welcome', { name: user.name });
    await emailClient.send({ to: user.email, subject: 'Welcome!', html });
  }
}

class UserService {
  constructor(validator, repository, emailService) {
    this.validator = validator;
    this.repository = repository;
    this.emailService = emailService;
  }

  async createUser(data) {
    this.validator.validate(data);
    const user = await this.repository.save(data);
    await this.emailService.send(user);
    return user;
  }
}
```

**Java (Spring Boot):**

```java
@Service
public class UserService {
    private final UserValidator validator;
    private final UserRepository repository;
    private final WelcomeEmailService emailService;

    public UserService(UserValidator validator, UserRepository repository,
                       WelcomeEmailService emailService) {
        this.validator = validator;
        this.repository = repository;
        this.emailService = emailService;
    }

    public User createUser(CreateUserRequest request) {
        validator.validate(request);
        User user = repository.save(request.toEntity());
        emailService.send(user);
        return user;
    }
}

@Component
public class UserValidator {
    public void validate(CreateUserRequest request) {
        if (!request.getEmail().contains("@"))
            throw new ValidationException("Invalid email");
        if (request.getPassword().length() < 8)
            throw new ValidationException("Password too short");
    }
}

@Repository
public interface UserRepository extends JpaRepository<User, Long> {
    Optional<User> findByEmail(String email);
}

@Service
public class WelcomeEmailService {
    private final EmailClient emailClient;

    public void send(User user) {
        emailClient.send(user.getEmail(), "Welcome!", renderTemplate(user));
    }
}
```

**Now each class has exactly one reason to change.** Email template changes? Only `WelcomeEmailService` is touched. Database schema changes? Only `UserRepository`. Validation rules change? Only `UserValidator`. Nothing ripples.

---

### O — Open/Closed Principle (OCP)

> **Software entities should be open for extension but closed for modification.**

**Simple English:** You should be able to add new features WITHOUT changing existing code. Existing, tested, working code should stay untouched.

**Analogy:** A power strip has sockets. To use a new device, you plug it in — you don't open the power strip and rewire it. The power strip is **closed** for modification but **open** for extension (new devices).

**The Problem:**

```javascript
// BAD — Every new notification channel requires modifying this function
class NotificationService {
  send(user, message, channel) {
    if (channel === 'email') {
      emailClient.send(user.email, message);
    } else if (channel === 'sms') {
      smsClient.send(user.phone, message);
    } else if (channel === 'push') {
      pushService.send(user.deviceToken, message);
    } else if (channel === 'whatsapp') {     // ← New channel = modify existing code
      whatsappClient.send(user.phone, message);
    } else if (channel === 'slack') {         // ← Another change to existing code
      slackClient.send(user.slackId, message);
    }
  }
}
```

Every new channel modifies a working, tested function. What if you accidentally break the SMS logic while adding Slack?

**The Fix:**

```javascript
// GOOD — define a contract (interface), then extend by adding new classes

// The contract — every notification channel must implement this
class NotificationChannel {
  send(user, message) {
    throw new Error('send() must be implemented');
  }
}

// Each channel is its own class
class EmailChannel extends NotificationChannel {
  send(user, message) {
    emailClient.send(user.email, message);
  }
}

class SmsChannel extends NotificationChannel {
  send(user, message) {
    smsClient.send(user.phone, message);
  }
}

class WhatsAppChannel extends NotificationChannel {
  send(user, message) {
    whatsappClient.send(user.phone, message);
  }
}

// NotificationService doesn't know or care which channels exist
class NotificationService {
  constructor(channels) {
    this.channels = channels; // Map of channel name → channel object
  }

  send(user, message, channelName) {
    const channel = this.channels[channelName];
    if (!channel) throw new Error(`Unknown channel: ${channelName}`);
    channel.send(user, message);
  }
}

// Adding Slack? Create a new class. Don't touch anything else.
class SlackChannel extends NotificationChannel {
  send(user, message) {
    slackClient.send(user.slackId, message);
  }
}
```

**Java (Spring Boot):**

```java
// The contract
public interface NotificationChannel {
    void send(User user, String message);
    String getChannelName();
}

// Each channel is a Spring bean
@Component
public class EmailChannel implements NotificationChannel {
    public void send(User user, String message) {
        emailClient.send(user.getEmail(), message);
    }
    public String getChannelName() { return "email"; }
}

@Component
public class SmsChannel implements NotificationChannel {
    public void send(User user, String message) {
        smsClient.send(user.getPhone(), message);
    }
    public String getChannelName() { return "sms"; }
}

// Service auto-discovers all channels via Spring DI
@Service
public class NotificationService {
    private final Map<String, NotificationChannel> channels;

    // Spring injects ALL beans that implement NotificationChannel
    public NotificationService(List<NotificationChannel> channelList) {
        this.channels = channelList.stream()
            .collect(Collectors.toMap(NotificationChannel::getChannelName, c -> c));
    }

    public void send(User user, String message, String channelName) {
        NotificationChannel channel = channels.get(channelName);
        if (channel == null) throw new IllegalArgumentException("Unknown channel: " + channelName);
        channel.send(user, message);
    }
}

// Adding WhatsApp? Create ONE new file. Touch NOTHING else.
@Component
public class WhatsAppChannel implements NotificationChannel {
    public void send(User user, String message) {
        whatsappClient.send(user.getPhone(), message);
    }
    public String getChannelName() { return "whatsapp"; }
}
```

**The beauty:** In Spring Boot, just creating the `WhatsAppChannel` class with `@Component` is enough. Spring automatically discovers it and injects it into `NotificationService`. Zero existing code modified.

---

### L — Liskov Substitution Principle (LSP)

> **Objects of a parent class should be replaceable with objects of a child class without breaking the application.**

**Simple English:** If your code works with a parent type, it should work with any child type too. A child class should behave like a more specific version of its parent, not a fundamentally different thing.

**Analogy:** If you hire a "vehicle" to transport goods, any vehicle should work — truck, van, car. But if someone sends you a boat and your route is on roads, substituting a boat for a vehicle breaks everything. The boat is technically a vehicle, but it can't do what was expected.

**The Classic Violation — Rectangle and Square:**

```javascript
// BAD — Square violates LSP
class Rectangle {
  constructor(width, height) {
    this.width = width;
    this.height = height;
  }

  setWidth(w) { this.width = w; }
  setHeight(h) { this.height = h; }
  area() { return this.width * this.height; }
}

class Square extends Rectangle {
  setWidth(w) {
    this.width = w;
    this.height = w;  // ← Forces height to match width
  }

  setHeight(h) {
    this.width = h;   // ← Forces width to match height
    this.height = h;
  }
}

// This code works with Rectangle but BREAKS with Square
function resizeAndCalculate(rectangle) {
  rectangle.setWidth(5);
  rectangle.setHeight(10);
  console.log(rectangle.area());  // Expected: 50
}

resizeAndCalculate(new Rectangle(2, 3));  // 50 ✓
resizeAndCalculate(new Square(2));        // 100 ✗ — LSP violated!
```

**A Real Backend Example:**

```javascript
// BAD — ReadOnlyFileStorage violates LSP
class FileStorage {
  read(path) { /* ... */ }
  write(path, data) { /* ... */ }
  delete(path) { /* ... */ }
}

class ReadOnlyFileStorage extends FileStorage {
  write(path, data) {
    throw new Error("Cannot write to read-only storage!"); // ← VIOLATION
  }
  delete(path) {
    throw new Error("Cannot delete from read-only storage!"); // ← VIOLATION
  }
}

// Code that expects FileStorage will crash with ReadOnlyFileStorage
function backupFiles(storage) {
  const data = storage.read('/data.json');
  storage.write('/backup/data.json', data);  // BOOM with ReadOnlyFileStorage
}
```

**The Fix:** Separate interfaces.

```javascript
// GOOD — separate capabilities into different interfaces
class Readable {
  read(path) { throw new Error('Not implemented'); }
}

class Writable {
  write(path, data) { throw new Error('Not implemented'); }
}

class ReadWriteStorage extends Readable {
  read(path) { /* ... */ }
  write(path, data) { /* ... */ }
}

class ReadOnlyStorage extends Readable {
  read(path) { /* ... */ }
  // No write method — can't be called by mistake
}
```

**Java:**

```java
public interface Readable {
    byte[] read(String path);
}

public interface Writable {
    void write(String path, byte[] data);
}

public class LocalStorage implements Readable, Writable {
    public byte[] read(String path) { /* ... */ }
    public void write(String path, byte[] data) { /* ... */ }
}

public class CdnStorage implements Readable {
    public byte[] read(String path) { /* ... */ }
    // No write — CDN is read-only. No LSP violation.
}
```

---

### I — Interface Segregation Principle (ISP)

> **A client should not be forced to depend on methods it doesn't use.**

**Simple English:** Don't create one giant interface that forces implementors to deal with methods they don't need. Split into smaller, focused interfaces.

**Analogy:** A restaurant menu with one section is bad for everyone. Vegetarians have to scroll past 50 meat dishes. Kids have to ignore the cocktails. A menu with sections (Starters, Mains, Desserts, Drinks, Kids Menu) lets each customer see only what's relevant.

**The Problem:**

```javascript
// BAD — one fat interface for everything
class Worker {
  work() { throw new Error('Not implemented'); }
  eat() { throw new Error('Not implemented'); }
  sleep() { throw new Error('Not implemented'); }
  attendMeeting() { throw new Error('Not implemented'); }
}

class HumanWorker extends Worker {
  work() { /* writes code */ }
  eat() { /* has lunch */ }
  sleep() { /* goes home */ }
  attendMeeting() { /* joins Zoom call */ }
}

class RobotWorker extends Worker {
  work() { /* assembles parts */ }
  eat() { /* WHAT? Robots don't eat! */ }
  sleep() { /* WHAT? Robots don't sleep! */ }
  attendMeeting() { /* WHAT? Robots don't attend meetings! */ }
}
```

**The Fix:**

```java
// GOOD — split into focused interfaces
public interface Workable {
    void work();
}

public interface Feedable {
    void eat();
}

public interface Restable {
    void sleep();
}

public class HumanWorker implements Workable, Feedable, Restable {
    public void work() { /* writes code */ }
    public void eat() { /* has lunch */ }
    public void sleep() { /* goes home */ }
}

public class RobotWorker implements Workable {
    public void work() { /* assembles parts */ }
    // Only implements what it actually does
}
```

**Real Backend Example:**

```java
// BAD — fat repository interface
public interface UserRepository {
    User save(User user);
    User findById(Long id);
    List<User> findAll();
    void delete(Long id);
    List<User> findByRole(String role);
    List<User> searchByName(String name);
    Map<String, Long> getRegistrationStats();
    void bulkImport(List<User> users);
    byte[] exportToCsv();
}

// A read-only reporting service is FORCED to depend on save, delete, bulkImport...

// GOOD — split by use case
public interface UserReader {
    User findById(Long id);
    List<User> findAll();
    List<User> findByRole(String role);
}

public interface UserWriter {
    User save(User user);
    void delete(Long id);
}

public interface UserSearcher {
    List<User> searchByName(String name);
}

public interface UserAnalytics {
    Map<String, Long> getRegistrationStats();
}

// Now the reporting service only depends on UserReader + UserAnalytics
// The import service only depends on UserWriter
```

---

### D — Dependency Inversion Principle (DIP)

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**

**Simple English:** Your business logic should not directly depend on specific databases, email providers, or APIs. Instead, it should depend on interfaces (contracts), and the specific implementations are plugged in from outside.

**Analogy:** A wall socket is an abstraction. You don't hardwire your laptop's charger into the building's electrical system. Instead, you plug it into a socket (interface). Tomorrow, you can plug in a phone charger, a lamp, or a blender — the socket doesn't care. If you hardwired your laptop, switching to a different charger would require rewiring the entire building.

**The Problem:**

```javascript
// BAD — OrderService is hardwired to Stripe and SendGrid
class OrderService {
  async placeOrder(orderData) {
    // Directly coupled to Stripe
    const stripe = require('stripe')('sk_live_...');
    const payment = await stripe.charges.create({
      amount: orderData.total * 100,
      currency: 'inr',
      source: orderData.cardToken
    });

    // Directly coupled to SendGrid
    const sgMail = require('@sendgrid/mail');
    sgMail.setApiKey('SG.xxx');
    await sgMail.send({
      to: orderData.email,
      subject: 'Order Confirmed',
      text: `Your order ${payment.id} is confirmed.`
    });
  }
}
```

**Problems:**
- Want to switch from Stripe to Razorpay? Rewrite OrderService.
- Want to switch from SendGrid to AWS SES? Rewrite OrderService.
- Want to test OrderService? You need real Stripe and SendGrid credentials.
- OrderService knows too much about payment and email internals.

**The Fix:**

```javascript
// GOOD — depend on abstractions, not concrete implementations

// Define contracts (abstractions)
class PaymentGateway {
  async charge(amount, currency, token) {
    throw new Error('Not implemented');
  }
}

class EmailService {
  async send(to, subject, body) {
    throw new Error('Not implemented');
  }
}

// Concrete implementations
class StripeGateway extends PaymentGateway {
  async charge(amount, currency, token) {
    const stripe = require('stripe')('sk_live_...');
    return stripe.charges.create({ amount: amount * 100, currency, source: token });
  }
}

class RazorpayGateway extends PaymentGateway {
  async charge(amount, currency, token) {
    // Razorpay-specific logic
  }
}

class SendGridEmailService extends EmailService {
  async send(to, subject, body) {
    // SendGrid-specific logic
  }
}

// OrderService depends on ABSTRACTIONS, not Stripe or SendGrid
class OrderService {
  constructor(paymentGateway, emailService) {
    this.paymentGateway = paymentGateway;  // Could be Stripe, Razorpay, anything
    this.emailService = emailService;       // Could be SendGrid, SES, anything
  }

  async placeOrder(orderData) {
    const payment = await this.paymentGateway.charge(
      orderData.total, 'inr', orderData.cardToken
    );

    await this.emailService.send(
      orderData.email,
      'Order Confirmed',
      `Your order ${payment.id} is confirmed.`
    );
  }
}

// Wire it up
const orderService = new OrderService(
  new StripeGateway(),       // ← Easy to swap to RazorpayGateway()
  new SendGridEmailService() // ← Easy to swap to AwsSesEmailService()
);
```

**Java (Spring Boot) — DIP is built into the framework:**

```java
// Define the contract
public interface PaymentGateway {
    PaymentResult charge(BigDecimal amount, String currency, String token);
}

// Implementation 1
@Component
@Profile("production")
public class StripeGateway implements PaymentGateway {
    public PaymentResult charge(BigDecimal amount, String currency, String token) {
        // Stripe SDK calls
    }
}

// Implementation 2
@Component
@Profile("test")
public class MockPaymentGateway implements PaymentGateway {
    public PaymentResult charge(BigDecimal amount, String currency, String token) {
        return new PaymentResult("mock-payment-id", "success");
    }
}

// OrderService depends on the interface, NOT on Stripe
@Service
public class OrderService {
    private final PaymentGateway paymentGateway;

    // Spring injects the right implementation based on profile
    public OrderService(PaymentGateway paymentGateway) {
        this.paymentGateway = paymentGateway;
    }

    public Order placeOrder(OrderRequest request) {
        PaymentResult payment = paymentGateway.charge(
            request.getTotal(), "INR", request.getCardToken()
        );
        // ...
    }
}
```

**The power of DIP:**
- In production: Spring injects `StripeGateway`
- In tests: Spring injects `MockPaymentGateway`
- Switching to Razorpay: Create `RazorpayGateway`, change one configuration line
- `OrderService` never changes. Ever.

### SOLID Summary

| Principle | One-Liner | When You're Violating It |
|-----------|-----------|------------------------|
| **S**RP | One class, one job | "This class is 500 lines and I can't name what it does in one sentence" |
| **O**CP | Add new features without modifying existing code | "Every new feature requires editing the same if-else chain" |
| **L**SP | Child classes should work wherever parent is expected | "I threw UnsupportedOperationException in a subclass" |
| **I**SP | Don't force classes to implement methods they don't use | "Half my interface methods throw 'not implemented'" |
| **D**IP | Depend on abstractions, not concrete classes | "My service directly imports and uses Stripe/SendGrid/AWS SDK" |

---

# PART 2 — DESIGN PATTERNS (CREATIONAL)

Creational patterns deal with **how objects are created**. Instead of calling `new Something()` directly everywhere, these patterns give you smarter ways to create objects.

---

## 4. Singleton — One and Only One

### The Problem

Some things should exist only once in your entire application — a database connection pool, a configuration manager, a logger, a cache client. If you create multiple instances, you waste resources (multiple DB connection pools), get inconsistent state (different config values), or cause bugs (multiple loggers writing to the same file simultaneously).

### The Analogy

A country has one president at a time. You don't create a new president every time someone needs to talk to the government. Everyone talks to the same one.

### When to Use

- Database connection pools
- Configuration managers
- Logger instances
- Cache clients (Redis connection)
- Thread pools

### JavaScript Implementation

```javascript
// SIMPLE SINGLETON — using a module (Node.js modules are cached)
// database.js
const { Pool } = require('pg');

const pool = new Pool({
  host: process.env.DB_HOST,
  port: 5432,
  database: process.env.DB_NAME,
  max: 20,
});

// Every file that imports this gets the SAME pool instance
module.exports = pool;
```

In Node.js, modules are cached after the first `require()`. So `require('./database')` in 100 different files returns the exact same object. Node.js gives you Singleton for free.

```javascript
// CLASS-BASED SINGLETON (if you need more control)
class ConfigManager {
  static #instance = null;

  #config = {};

  constructor() {
    if (ConfigManager.#instance) {
      throw new Error('Use ConfigManager.getInstance() instead');
    }
    this.#config = {
      dbHost: process.env.DB_HOST || 'localhost',
      dbPort: parseInt(process.env.DB_PORT || '5432'),
      redisUrl: process.env.REDIS_URL || 'redis://localhost:6379',
    };
  }

  static getInstance() {
    if (!ConfigManager.#instance) {
      ConfigManager.#instance = new ConfigManager();
    }
    return ConfigManager.#instance;
  }

  get(key) {
    return this.#config[key];
  }
}

// Usage
const config = ConfigManager.getInstance();
console.log(config.get('dbHost'));  // 'localhost'
```

### Java (Spring Boot) Implementation

In Spring Boot, every bean is a Singleton by default. You don't need to implement the pattern manually.

```java
@Configuration
public class AppConfig {

    // This bean is created ONCE and shared across the entire application
    @Bean
    public RestTemplate restTemplate() {
        return new RestTemplate();
    }
}

// Every class that needs RestTemplate gets the SAME instance
@Service
public class UserService {
    private final RestTemplate restTemplate;  // Same instance everywhere

    public UserService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
}

@Service
public class OrderService {
    private final RestTemplate restTemplate;  // Same instance as above

    public OrderService(RestTemplate restTemplate) {
        this.restTemplate = restTemplate;
    }
}
```

### When NOT to Use

- When the object holds user-specific state (each user needs their own instance)
- When you need multiple configurations simultaneously (e.g., connections to two different databases)
- When it makes testing harder (singletons are hard to mock — prefer Dependency Injection)

---

## 5. Factory Method — Let Someone Else Decide

### The Problem

Your code needs to create objects, but which exact type of object depends on some condition (user input, configuration, environment). If you put `if-else` or `switch` blocks everywhere to decide which class to instantiate, you're coupling your code to every specific class.

### The Analogy

A pizza shop has a menu. You say "I want a pizza" and the kitchen decides how to make it (thin crust, deep dish, stuffed crust). You don't walk into the kitchen and assemble it yourself. The kitchen is the **factory** — it creates the right pizza based on your order.

### When to Use

- Creating different payment processors based on payment method
- Creating different database connectors based on configuration
- Creating different notification senders based on channel
- Creating different export formats (PDF, CSV, Excel)
- Anywhere you see: `if (type === 'x') return new X(); else if (type === 'y') return new Y();`

### JavaScript Implementation

```javascript
// THE PROBLEM — creation logic scattered everywhere
function handleExport(req, res) {
  let exporter;
  if (req.query.format === 'pdf') {
    exporter = new PdfExporter(config.pdfSettings);
  } else if (req.query.format === 'csv') {
    exporter = new CsvExporter(config.csvDelimiter);
  } else if (req.query.format === 'excel') {
    exporter = new ExcelExporter(config.excelTemplate);
  }
  const file = exporter.export(data);
  res.send(file);
}

// This same if-else is probably repeated in 5 other places
```

```javascript
// THE FIX — Factory centralises creation logic

// Step 1: Common interface
class Exporter {
  export(data) {
    throw new Error('export() must be implemented');
  }
}

class PdfExporter extends Exporter {
  export(data) {
    // Generate PDF
    return pdfLib.createDocument(data);
  }
}

class CsvExporter extends Exporter {
  export(data) {
    // Generate CSV
    return data.map(row => row.join(',')).join('\n');
  }
}

class ExcelExporter extends Exporter {
  export(data) {
    // Generate Excel
    return excelLib.createWorkbook(data);
  }
}

// Step 2: The Factory
class ExporterFactory {
  static create(format) {
    switch (format) {
      case 'pdf':   return new PdfExporter();
      case 'csv':   return new CsvExporter();
      case 'excel': return new ExcelExporter();
      default:      throw new Error(`Unsupported format: ${format}`);
    }
  }
}

// Step 3: Clean usage — no if-else, no knowledge of specific classes
function handleExport(req, res) {
  const exporter = ExporterFactory.create(req.query.format);
  const file = exporter.export(data);
  res.send(file);
}
```

### Java (Spring Boot) Implementation

```java
public interface Exporter {
    byte[] export(List<Map<String, Object>> data);
    String getFormat();
}

@Component
public class PdfExporter implements Exporter {
    public byte[] export(List<Map<String, Object>> data) { /* PDF logic */ }
    public String getFormat() { return "pdf"; }
}

@Component
public class CsvExporter implements Exporter {
    public byte[] export(List<Map<String, Object>> data) { /* CSV logic */ }
    public String getFormat() { return "csv"; }
}

@Component
public class ExporterFactory {
    private final Map<String, Exporter> exporters;

    // Spring injects ALL Exporter implementations automatically
    public ExporterFactory(List<Exporter> exporterList) {
        this.exporters = exporterList.stream()
            .collect(Collectors.toMap(Exporter::getFormat, e -> e));
    }

    public Exporter create(String format) {
        Exporter exporter = exporters.get(format);
        if (exporter == null) throw new IllegalArgumentException("Unsupported: " + format);
        return exporter;
    }
}

// Controller — clean, no if-else
@RestController
public class ExportController {
    private final ExporterFactory factory;

    @GetMapping("/export")
    public ResponseEntity<byte[]> export(@RequestParam String format) {
        Exporter exporter = factory.create(format);
        byte[] file = exporter.export(getData());
        return ResponseEntity.ok(file);
    }
}
```

**The beauty with Spring:** To add XML export, create `XmlExporter implements Exporter` with `@Component`. The factory auto-discovers it. Zero changes to existing code.

---

## 6. Abstract Factory — Factory of Factories

### The Analogy

Imagine you're furnishing a house and you want everything to match — all modern furniture or all vintage furniture. You don't pick a modern sofa with a vintage table. An **Abstract Factory** gives you a "Modern Furniture Set" factory or a "Vintage Furniture Set" factory. Each factory produces a complete, matching family of products.

### When to Use

- When you need to create families of related objects that must be consistent
- Database provider families (PostgreSQL connection + query builder + migration tool)
- UI theme families (dark buttons + dark inputs + dark cards)
- Cross-platform kits (Windows notification + Windows file dialog + Windows menu)

```javascript
// Abstract Factory for database providers
class DatabaseFactory {
  createConnection() { throw new Error('Not implemented'); }
  createQueryBuilder() { throw new Error('Not implemented'); }
  createMigrator() { throw new Error('Not implemented'); }
}

class PostgresFactory extends DatabaseFactory {
  createConnection() { return new PostgresConnection(); }
  createQueryBuilder() { return new PostgresQueryBuilder(); }
  createMigrator() { return new PostgresMigrator(); }
}

class MongoFactory extends DatabaseFactory {
  createConnection() { return new MongoConnection(); }
  createQueryBuilder() { return new MongoQueryBuilder(); }
  createMigrator() { return new MongoMigrator(); }
}

// Usage — everything is consistent
function setupDatabase(factory) {
  const connection = factory.createConnection();
  const queryBuilder = factory.createQueryBuilder();
  const migrator = factory.createMigrator();
  // All three are guaranteed to work together
}

setupDatabase(new PostgresFactory());  // All Postgres tools
setupDatabase(new MongoFactory());     // All Mongo tools
```

**In practice**, Abstract Factory is less common than regular Factory in backend development. You'll see it in cross-platform frameworks and SDKs more than in typical API code.

---

## 7. Builder — Step by Step Construction

### The Problem

You have an object with many optional parameters. The constructor becomes a monster:

```javascript
// THE NIGHTMARE — 12 parameters, which is which?
const query = new DatabaseQuery(
  'users',        // table
  ['name', 'email'], // fields
  { age: { $gt: 18 } }, // where
  null,           // join? no
  'name',         // orderBy
  'ASC',          // direction
  20,             // limit
  40,             // offset
  true,           // distinct?
  null,           // groupBy? no
  null,           // having? no
  true            // count?
);
// What's that `true` at position 9? Who remembers?
```

### The Analogy

Ordering a customised burger: "I'll have a chicken burger, add cheese, no onions, extra sauce, with a large bun." You build it step by step, adding what you want, skipping what you don't. The builder assembles everything at the end.

### When to Use

- Complex objects with many optional fields
- Query builders
- HTTP request builders
- Configuration objects
- Email/notification builders
- Anything where the constructor would need 5+ parameters

### JavaScript Implementation

```javascript
class QueryBuilder {
  #table = '';
  #fields = ['*'];
  #conditions = [];
  #orderBy = null;
  #limit = null;
  #offset = null;

  constructor(table) {
    this.#table = table;
  }

  select(...fields) {
    this.#fields = fields;
    return this;  // Return `this` to allow chaining
  }

  where(field, operator, value) {
    this.#conditions.push({ field, operator, value });
    return this;
  }

  orderBy(field, direction = 'ASC') {
    this.#orderBy = { field, direction };
    return this;
  }

  limit(count) {
    this.#limit = count;
    return this;
  }

  offset(count) {
    this.#offset = count;
    return this;
  }

  build() {
    let sql = `SELECT ${this.#fields.join(', ')} FROM ${this.#table}`;

    if (this.#conditions.length > 0) {
      const where = this.#conditions
        .map(c => `${c.field} ${c.operator} '${c.value}'`)
        .join(' AND ');
      sql += ` WHERE ${where}`;
    }

    if (this.#orderBy) {
      sql += ` ORDER BY ${this.#orderBy.field} ${this.#orderBy.direction}`;
    }

    if (this.#limit) sql += ` LIMIT ${this.#limit}`;
    if (this.#offset) sql += ` OFFSET ${this.#offset}`;

    return sql;
  }
}

// Usage — beautiful, readable, self-documenting
const query = new QueryBuilder('users')
  .select('name', 'email', 'age')
  .where('age', '>', 18)
  .where('status', '=', 'active')
  .orderBy('name', 'ASC')
  .limit(20)
  .offset(40)
  .build();

// Compare this with the 12-parameter constructor above. Night and day.
```

### Java (Spring Boot) — Lombok's @Builder

```java
// Lombok generates the Builder pattern for you
@Builder
@Getter
public class NotificationRequest {
    private final String userId;
    private final String message;
    @Builder.Default private String channel = "email";
    @Builder.Default private Priority priority = Priority.NORMAL;
    @Builder.Default private int maxRetries = 3;
    private String templateId;
    private Map<String, String> templateVars;
    private List<String> attachments;
}

// Usage
NotificationRequest notification = NotificationRequest.builder()
    .userId("user-42")
    .message("Your order has shipped!")
    .channel("sms")
    .priority(Priority.HIGH)
    .maxRetries(5)
    .build();

// Clean, readable, impossible to mix up parameter order
```

```java
// Manual Builder (for when you want full control)
public class HttpRequest {
    private final String method;
    private final String url;
    private final Map<String, String> headers;
    private final String body;
    private final int timeout;

    private HttpRequest(Builder builder) {
        this.method = builder.method;
        this.url = builder.url;
        this.headers = builder.headers;
        this.body = builder.body;
        this.timeout = builder.timeout;
    }

    public static class Builder {
        private String method = "GET";
        private String url;
        private Map<String, String> headers = new HashMap<>();
        private String body;
        private int timeout = 30000;

        public Builder(String url) { this.url = url; }

        public Builder method(String m) { this.method = m; return this; }
        public Builder header(String k, String v) { this.headers.put(k, v); return this; }
        public Builder body(String b) { this.body = b; return this; }
        public Builder timeout(int t) { this.timeout = t; return this; }

        public HttpRequest build() {
            if (url == null) throw new IllegalStateException("URL is required");
            return new HttpRequest(this);
        }
    }
}

// Usage
HttpRequest request = new HttpRequest.Builder("https://api.example.com/users")
    .method("POST")
    .header("Content-Type", "application/json")
    .header("Authorization", "Bearer token-here")
    .body("{\"name\": \"Ravi\"}")
    .timeout(5000)
    .build();
```

---

## 8. Prototype — Clone Instead of Create

### The Analogy

Filling out a form. If you need to fill 20 forms that are 90% identical (same company, same address, same department), you photocopy a filled-out template and only change the parts that differ. You don't fill out each form from scratch.

### When to Use

- Creating objects is expensive (requires database calls, API calls, complex computation)
- You need many similar objects with small variations
- Configuration templates with slight overrides
- Test data generation

```javascript
// JavaScript has built-in prototype support
const baseConfig = {
  database: 'postgres',
  host: 'localhost',
  port: 5432,
  pool: { min: 5, max: 20 },
  ssl: false,
  logging: true,
};

// Clone and customize for different environments
const productionConfig = {
  ...baseConfig,            // Clone all base properties
  host: 'prod-db.aws.com',
  ssl: true,
  logging: false,
  pool: { ...baseConfig.pool, max: 100 },
};

const stagingConfig = {
  ...baseConfig,
  host: 'staging-db.aws.com',
  ssl: true,
};
```

```java
// Java — implement Cloneable
public class ServerConfig implements Cloneable {
    private String host;
    private int port;
    private boolean ssl;
    private Map<String, String> headers;

    @Override
    public ServerConfig clone() {
        try {
            ServerConfig cloned = (ServerConfig) super.clone();
            cloned.headers = new HashMap<>(this.headers); // Deep copy mutable fields
            return cloned;
        } catch (CloneNotSupportedException e) {
            throw new RuntimeException(e);
        }
    }
}

// Usage
ServerConfig base = new ServerConfig("localhost", 8080, false);
ServerConfig production = base.clone();
production.setHost("prod.example.com");
production.setSsl(true);
```

---

# PART 3 — DESIGN PATTERNS (STRUCTURAL)

Structural patterns deal with **how objects are composed** — connecting pieces together to form larger structures.

---

## 9. Adapter — Making Incompatible Things Work Together

### The Analogy

You travel from India to the UK. Your charger has Indian pins but UK sockets are different. You use a **travel adapter** — it doesn't change your charger or the socket. It sits in between and makes them compatible.

### When to Use

- Integrating with third-party APIs that have a different interface than what your code expects
- Switching from one service provider to another (Stripe to Razorpay)
- Wrapping legacy code to work with new code
- Normalising data from different sources into a consistent format

### JavaScript Implementation

```javascript
// Your app expects this interface
class PaymentProcessor {
  charge(amount, currency, token) { throw new Error('Not implemented'); }
}

// But Stripe's SDK has a completely different interface
// stripe.charges.create({ amount, currency, source })

// And Razorpay has yet another interface
// razorpay.payments.capture(paymentId, amount, currency)

// Adapters make them compatible with YOUR interface

class StripeAdapter extends PaymentProcessor {
  constructor() {
    super();
    this.stripe = require('stripe')('sk_...');
  }

  async charge(amount, currency, token) {
    // Adapt YOUR interface to STRIPE's interface
    const result = await this.stripe.charges.create({
      amount: amount * 100,  // Stripe uses cents
      currency: currency.toLowerCase(),
      source: token,
    });

    // Return YOUR standardised format
    return {
      id: result.id,
      status: result.status === 'succeeded' ? 'success' : 'failed',
      amount: result.amount / 100,
    };
  }
}

class RazorpayAdapter extends PaymentProcessor {
  constructor() {
    super();
    this.razorpay = new Razorpay({ key_id: '...', key_secret: '...' });
  }

  async charge(amount, currency, token) {
    // Adapt YOUR interface to RAZORPAY's interface
    const result = await this.razorpay.payments.capture(token, amount * 100, currency);

    return {
      id: result.id,
      status: result.status === 'captured' ? 'success' : 'failed',
      amount: amount,
    };
  }
}

// Your code doesn't know or care which provider is behind the adapter
class OrderService {
  constructor(paymentProcessor) {
    this.paymentProcessor = paymentProcessor;
  }

  async placeOrder(order) {
    const payment = await this.paymentProcessor.charge(order.total, 'INR', order.token);
    if (payment.status === 'success') {
      // save order
    }
  }
}
```

### Java (Spring Boot)

```java
// Your interface
public interface PaymentProcessor {
    PaymentResult charge(BigDecimal amount, String currency, String token);
}

// Stripe adapter
@Component
@ConditionalOnProperty(name = "payment.provider", havingValue = "stripe")
public class StripeAdapter implements PaymentProcessor {
    private final Stripe stripeClient;

    public PaymentResult charge(BigDecimal amount, String currency, String token) {
        // Convert YOUR format to STRIPE's format
        ChargeCreateParams params = ChargeCreateParams.builder()
            .setAmount(amount.multiply(BigDecimal.valueOf(100)).longValue())
            .setCurrency(currency.toLowerCase())
            .setSource(token)
            .build();

        Charge charge = Charge.create(params);

        return new PaymentResult(
            charge.getId(),
            "succeeded".equals(charge.getStatus()) ? "success" : "failed",
            amount
        );
    }
}

// Switch to Razorpay? Just change application.properties:
// payment.provider=razorpay
```

---

## 10. Decorator — Adding Features Without Touching Original Code

### The Analogy

You buy a plain coffee. Then you add milk (decorator #1). Then whipped cream (decorator #2). Then chocolate syrup (decorator #3). Each addition wraps the previous one, adding something extra. The core is still coffee — you've just enhanced it, layer by layer, without changing the coffee itself.

### When to Use

- Adding logging to existing services without modifying them
- Adding caching to existing repositories
- Adding retry logic to API calls
- Adding authentication checks to existing handlers
- Express.js middleware is literally the Decorator pattern

### JavaScript Implementation

```javascript
// Base service
class UserService {
  async getUser(id) {
    const user = await db.query('SELECT * FROM users WHERE id = ?', id);
    return user;
  }
}

// Decorator 1: Add caching
class CachedUserService {
  constructor(userService, cache) {
    this.userService = userService;
    this.cache = cache;
  }

  async getUser(id) {
    const cached = await this.cache.get(`user:${id}`);
    if (cached) return JSON.parse(cached);

    const user = await this.userService.getUser(id);  // Delegate to original
    await this.cache.set(`user:${id}`, JSON.stringify(user), 'EX', 3600);
    return user;
  }
}

// Decorator 2: Add logging
class LoggedUserService {
  constructor(userService, logger) {
    this.userService = userService;
    this.logger = logger;
  }

  async getUser(id) {
    this.logger.info(`Fetching user ${id}`);
    const start = Date.now();
    const user = await this.userService.getUser(id);  // Delegate to original
    this.logger.info(`Fetched user ${id} in ${Date.now() - start}ms`);
    return user;
  }
}

// Stack the decorators — each wraps the previous one
const baseService = new UserService();
const cachedService = new CachedUserService(baseService, redisClient);
const loggedCachedService = new LoggedUserService(cachedService, logger);

// loggedCachedService.getUser(42) now:
// 1. Logs "Fetching user 42"
// 2. Checks cache
// 3. If cache miss, queries database
// 4. Stores in cache
// 5. Logs "Fetched user 42 in 15ms"
```

### Express.js Middleware IS the Decorator Pattern

```javascript
// Each middleware DECORATES the request handler

app.get('/api/users/:id',
  authenticate,        // Decorator 1: check auth token
  rateLimit(100),      // Decorator 2: rate limiting
  validateParams,      // Decorator 3: input validation
  cacheResponse(3600), // Decorator 4: response caching
  async (req, res) => {
    // The actual handler — the "core coffee"
    const user = await UserService.getUser(req.params.id);
    res.json(user);
  }
);
```

### Java (Spring Boot) — AOP Is the Decorator Pattern

```java
// Spring AOP decorates methods with cross-cutting concerns

@Aspect
@Component
public class LoggingAspect {

    @Around("execution(* com.myapp.service.*.*(..))")
    public Object logMethod(ProceedingJoinPoint joinPoint) throws Throwable {
        String method = joinPoint.getSignature().getName();
        log.info("Calling: {}", method);
        long start = System.currentTimeMillis();

        Object result = joinPoint.proceed();  // Call the original method

        log.info("{} completed in {}ms", method, System.currentTimeMillis() - start);
        return result;
    }
}

// @Cacheable is a decorator for caching
@Service
public class UserService {

    @Cacheable(value = "users", key = "#id")  // Decorator: caches the result
    @Retryable(maxAttempts = 3)                // Decorator: retries on failure
    public User getUser(Long id) {
        return userRepository.findById(id)
            .orElseThrow(() -> new UserNotFoundException(id));
    }
}
```

---

## 11. Facade — One Simple Door to a Complex Building

### The Analogy

When you book a holiday through a travel agent, you say "I want to go to Goa for 5 days." The travel agent handles flights, hotels, car rentals, restaurant reservations, and activity bookings. You don't call 10 different services yourself. The travel agent is the **facade** — a simple interface to a complex system.

### When to Use

- Simplifying complex subsystem interactions
- Creating a unified API for multiple microservices
- Wrapping complex third-party library APIs
- Providing a "just works" interface for common operations

```javascript
// Without Facade — the controller knows too much
app.post('/api/orders', async (req, res) => {
  const inventory = await InventoryService.check(req.body.items);
  if (!inventory.available) return res.status(400).json({ error: 'Out of stock' });

  const price = await PricingService.calculate(req.body.items, req.body.coupon);
  const tax = await TaxService.calculate(price, req.body.shippingAddress);
  const payment = await PaymentService.charge(price + tax, req.body.paymentToken);

  if (payment.status !== 'success') return res.status(402).json({ error: 'Payment failed' });

  const order = await OrderRepository.save({ ...req.body, total: price + tax });
  await InventoryService.reserve(req.body.items);
  await NotificationService.sendConfirmation(order);
  await AnalyticsService.track('order_placed', order);

  res.status(201).json(order);
});
```

```javascript
// With Facade — the controller is clean
class OrderFacade {
  constructor(inventory, pricing, tax, payment, orderRepo, notification, analytics) {
    // ... store all dependencies
  }

  async placeOrder(orderData) {
    // Step 1: Check inventory
    await this.inventory.ensureAvailable(orderData.items);

    // Step 2: Calculate total
    const price = await this.pricing.calculate(orderData.items, orderData.coupon);
    const tax = await this.tax.calculate(price, orderData.shippingAddress);
    const total = price + tax;

    // Step 3: Charge payment
    const payment = await this.payment.charge(total, orderData.paymentToken);
    if (payment.status !== 'success') throw new PaymentFailedError();

    // Step 4: Create order
    const order = await this.orderRepo.save({ ...orderData, total });

    // Step 5: Post-order actions
    await this.inventory.reserve(orderData.items);
    await this.notification.sendConfirmation(order);
    await this.analytics.track('order_placed', order);

    return order;
  }
}

// Controller is now one line
app.post('/api/orders', async (req, res) => {
  const order = await orderFacade.placeOrder(req.body);
  res.status(201).json(order);
});
```

---

## 12. Proxy — A Gatekeeper That Controls Access

### The Analogy

A celebrity's personal assistant is a proxy. Fans don't talk directly to the celebrity. The assistant screens calls, manages schedules, and decides who gets through. The assistant has the same "interface" as the celebrity (takes messages, schedules meetings) but adds a control layer.

### Types of Proxies

| Type | Purpose | Example |
|------|---------|---------|
| **Protection Proxy** | Controls access | Check permissions before allowing database operations |
| **Caching Proxy** | Caches results | Cache API responses to avoid repeated calls |
| **Logging Proxy** | Logs operations | Log every database query for debugging |
| **Virtual Proxy** | Lazy loading | Don't load expensive data until it's actually accessed |

```javascript
// Caching Proxy for an external API
class WeatherApi {
  async getWeather(city) {
    const response = await fetch(`https://api.weather.com/${city}`);
    return response.json();
  }
}

class CachedWeatherProxy {
  constructor(weatherApi, cache) {
    this.api = weatherApi;
    this.cache = cache;
  }

  async getWeather(city) {
    const cached = this.cache.get(city);
    if (cached && Date.now() - cached.timestamp < 600000) {
      return cached.data;  // Return cached if less than 10 min old
    }

    const data = await this.api.getWeather(city);
    this.cache.set(city, { data, timestamp: Date.now() });
    return data;
  }
}

// Usage — same interface, but with caching
const weatherService = new CachedWeatherProxy(new WeatherApi(), new Map());
const weather = await weatherService.getWeather('Delhi');
```

```java
// Spring's @Transactional is a Proxy
@Service
public class OrderService {

    @Transactional  // Spring creates a PROXY that wraps this method
    public Order placeOrder(OrderRequest request) {
        // Spring's proxy: BEGIN TRANSACTION
        Order order = orderRepository.save(request.toEntity());
        paymentService.charge(order);
        inventoryService.reserve(order.getItems());
        // Spring's proxy: COMMIT TRANSACTION (or ROLLBACK on exception)
        return order;
    }
}
// You never see the proxy — Spring creates it behind the scenes
```

---

## 13. Composite — Treating One and Many the Same Way

### The Analogy

A file system: a folder can contain files AND other folders. Whether you're calculating the size of a single file or an entire folder tree, you call `.getSize()` on both the same way. The folder (composite) delegates to its children.

### When to Use

- Tree structures (menus, categories, organisational hierarchies)
- Permission systems (user has permissions, group has permissions, both checked the same way)
- Pricing calculations (single item price + bundle price + discount + tax)

```javascript
// Menu system for a restaurant app
class MenuItem {
  constructor(name, price) {
    this.name = name;
    this.price = price;
  }

  getPrice() { return this.price; }
  display(indent = 0) {
    console.log(' '.repeat(indent) + `${this.name}: ₹${this.price}`);
  }
}

class MenuCategory {
  constructor(name) {
    this.name = name;
    this.items = [];  // Can contain MenuItem or MenuCategory
  }

  add(item) { this.items.push(item); }

  getPrice() {
    // Total price of all items in this category (and sub-categories)
    return this.items.reduce((sum, item) => sum + item.getPrice(), 0);
  }

  display(indent = 0) {
    console.log(' '.repeat(indent) + `[${this.name}]`);
    this.items.forEach(item => item.display(indent + 2));
  }
}

// Usage
const menu = new MenuCategory('Full Menu');

const starters = new MenuCategory('Starters');
starters.add(new MenuItem('Paneer Tikka', 250));
starters.add(new MenuItem('Spring Rolls', 180));

const mains = new MenuCategory('Main Course');
const northIndian = new MenuCategory('North Indian');
northIndian.add(new MenuItem('Dal Makhani', 280));
northIndian.add(new MenuItem('Butter Naan', 60));
mains.add(northIndian);

menu.add(starters);
menu.add(mains);

console.log(`Total menu value: ₹${menu.getPrice()}`);  // 770
menu.display();
```

---

# PART 4 — DESIGN PATTERNS (BEHAVIORAL)

Behavioral patterns deal with **how objects communicate** — the flow of responsibility between objects.

---

## 14. Strategy — Swappable Algorithms

### The Problem

This is the **most important pattern for backend developers**. You have an operation that can be done in multiple ways, and the way might change at runtime or need to be extended in the future.

### The Analogy

Google Maps gives you three route strategies: driving, walking, cycling. The destination is the same. The algorithm to get there is different. You **swap the strategy** without changing the map.

### When to Use (This Pattern Is Everywhere)

- Different payment methods (credit card, UPI, net banking, wallet)
- Different authentication strategies (JWT, API key, OAuth, session)
- Different pricing strategies (flat rate, per-use, tiered, freemium)
- Different compression algorithms (gzip, brotli, zlib)
- Different sorting algorithms based on data size
- Different shipping cost calculations by region
- **Basically: anywhere you see an if-else chain that decides HOW to do something**

### JavaScript Implementation

```javascript
// THE PROBLEM — growing if-else chain
class PricingService {
  calculatePrice(plan, usage) {
    if (plan === 'flat') {
      return 999;
    } else if (plan === 'per-use') {
      return usage * 0.5;
    } else if (plan === 'tiered') {
      if (usage <= 100) return usage * 1.0;
      if (usage <= 1000) return 100 + (usage - 100) * 0.75;
      return 100 + 675 + (usage - 1000) * 0.5;
    } else if (plan === 'freemium') {
      if (usage <= 50) return 0;
      return (usage - 50) * 0.8;
    }
    // This keeps growing with every new plan...
  }
}
```

```javascript
// THE FIX — Strategy Pattern

// Step 1: Define strategy interface
class PricingStrategy {
  calculate(usage) { throw new Error('Not implemented'); }
}

// Step 2: Implement each strategy
class FlatPricing extends PricingStrategy {
  calculate(usage) {
    return 999;
  }
}

class PerUsePricing extends PricingStrategy {
  constructor(ratePerUnit) {
    super();
    this.rate = ratePerUnit;
  }

  calculate(usage) {
    return usage * this.rate;
  }
}

class TieredPricing extends PricingStrategy {
  calculate(usage) {
    let cost = 0;
    if (usage <= 100) return usage * 1.0;
    cost += 100 * 1.0;
    if (usage <= 1000) return cost + (usage - 100) * 0.75;
    cost += 900 * 0.75;
    return cost + (usage - 1000) * 0.5;
  }
}

class FreemiumPricing extends PricingStrategy {
  calculate(usage) {
    if (usage <= 50) return 0;
    return (usage - 50) * 0.8;
  }
}

// Step 3: Context that uses the strategy
class PricingService {
  constructor(strategies) {
    this.strategies = strategies; // { flat: FlatPricing, perUse: PerUsePricing, ... }
  }

  calculatePrice(planName, usage) {
    const strategy = this.strategies[planName];
    if (!strategy) throw new Error(`Unknown plan: ${planName}`);
    return strategy.calculate(usage);
  }
}

// Step 4: Wire it up
const pricingService = new PricingService({
  flat: new FlatPricing(),
  perUse: new PerUsePricing(0.5),
  tiered: new TieredPricing(),
  freemium: new FreemiumPricing(),
});

console.log(pricingService.calculatePrice('tiered', 500));  // 400
```

### Java (Spring Boot) Implementation

```java
public interface PricingStrategy {
    BigDecimal calculate(int usage);
    String getPlanName();
}

@Component
public class FlatPricing implements PricingStrategy {
    public BigDecimal calculate(int usage) { return new BigDecimal("999"); }
    public String getPlanName() { return "flat"; }
}

@Component
public class TieredPricing implements PricingStrategy {
    public BigDecimal calculate(int usage) {
        // tiered calculation logic
    }
    public String getPlanName() { return "tiered"; }
}

@Service
public class PricingService {
    private final Map<String, PricingStrategy> strategies;

    public PricingService(List<PricingStrategy> strategyList) {
        this.strategies = strategyList.stream()
            .collect(Collectors.toMap(PricingStrategy::getPlanName, s -> s));
    }

    public BigDecimal calculatePrice(String plan, int usage) {
        PricingStrategy strategy = strategies.get(plan);
        if (strategy == null) throw new IllegalArgumentException("Unknown plan: " + plan);
        return strategy.calculate(usage);
    }
}
```

---

## 15. Observer — When Something Happens, Tell Everyone Who Cares

### The Analogy

YouTube subscriptions. When a creator uploads a video, YouTube doesn't check every user individually. Instead, subscribers have said "notify me." When a new video is published, YouTube notifies all subscribers. The creator (subject) doesn't know who the subscribers are or what they'll do with the notification.

### When to Use

- Event-driven systems (order placed → notify warehouse, email customer, update analytics, adjust inventory)
- Real-time notifications
- Audit logging
- Cache invalidation
- Anything where one action triggers multiple reactions

### JavaScript Implementation

```javascript
// Simple EventEmitter-based Observer (Node.js has this built in)
const EventEmitter = require('events');

class OrderEvents extends EventEmitter {}
const orderEvents = new OrderEvents();

// Observers (listeners) — each handles one concern
// These DON'T know about each other

orderEvents.on('order.placed', async (order) => {
  await EmailService.sendConfirmation(order);
  console.log(`Email sent for order ${order.id}`);
});

orderEvents.on('order.placed', async (order) => {
  await InventoryService.reserve(order.items);
  console.log(`Inventory reserved for order ${order.id}`);
});

orderEvents.on('order.placed', async (order) => {
  await AnalyticsService.track('order_placed', {
    orderId: order.id,
    total: order.total,
  });
});

orderEvents.on('order.placed', async (order) => {
  await NotificationService.pushToAdmin('New order received', order);
});

// The subject (publisher) — doesn't know who's listening
class OrderService {
  async placeOrder(orderData) {
    const order = await OrderRepository.save(orderData);

    // Emit the event — all observers are notified
    orderEvents.emit('order.placed', order);

    return order;
  }
}

// Adding a new reaction to order placement?
// Just add another listener. Don't touch OrderService.
orderEvents.on('order.placed', async (order) => {
  await LoyaltyService.addPoints(order.userId, order.total);
});
```

### Java (Spring Boot) — Application Events

```java
// Define the event
public class OrderPlacedEvent extends ApplicationEvent {
    private final Order order;

    public OrderPlacedEvent(Object source, Order order) {
        super(source);
        this.order = order;
    }

    public Order getOrder() { return order; }
}

// Publisher — emits the event
@Service
public class OrderService {
    private final ApplicationEventPublisher eventPublisher;

    public Order placeOrder(OrderRequest request) {
        Order order = orderRepository.save(request.toEntity());

        // Publish event — don't know or care who's listening
        eventPublisher.publishEvent(new OrderPlacedEvent(this, order));

        return order;
    }
}

// Listener 1: Send email
@Component
public class OrderEmailListener {
    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        emailService.sendConfirmation(event.getOrder());
    }
}

// Listener 2: Update inventory
@Component
public class InventoryListener {
    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        inventoryService.reserve(event.getOrder().getItems());
    }
}

// Listener 3: Track analytics (async — don't block the order flow)
@Component
public class AnalyticsListener {
    @Async
    @EventListener
    public void onOrderPlaced(OrderPlacedEvent event) {
        analyticsService.track("order_placed", event.getOrder());
    }
}

// Adding loyalty points? Create a new listener. Touch NOTHING else.
```

---

## 16. Command — Turn Actions into Objects

### The Analogy

A restaurant order slip. When you tell the waiter "one paneer tikka," the waiter doesn't cook it. They write it on a slip (the **command object**) and pass it to the kitchen. The slip can be queued, prioritised, logged, and even cancelled before the kitchen processes it.

### When to Use

- Undo/redo functionality
- Job queues (each job is a command object)
- Transaction logging
- Scheduling operations for later execution
- Macro operations (group multiple commands into one)

```javascript
// Command pattern for a task queue system
class Command {
  execute() { throw new Error('Not implemented'); }
  undo() { throw new Error('Not implemented'); }
}

class SendEmailCommand extends Command {
  constructor(emailService, to, subject, body) {
    super();
    this.emailService = emailService;
    this.to = to;
    this.subject = subject;
    this.body = body;
    this.messageId = null;
  }

  async execute() {
    this.messageId = await this.emailService.send(this.to, this.subject, this.body);
    return this.messageId;
  }

  async undo() {
    if (this.messageId) {
      await this.emailService.recall(this.messageId);
    }
  }
}

class UpdateDatabaseCommand extends Command {
  constructor(repository, id, newData) {
    super();
    this.repository = repository;
    this.id = id;
    this.newData = newData;
    this.previousData = null;
  }

  async execute() {
    this.previousData = await this.repository.findById(this.id);
    await this.repository.update(this.id, this.newData);
  }

  async undo() {
    if (this.previousData) {
      await this.repository.update(this.id, this.previousData);
    }
  }
}

// Command queue — execute, log, undo
class CommandQueue {
  constructor() {
    this.history = [];
  }

  async execute(command) {
    await command.execute();
    this.history.push(command);
  }

  async undoLast() {
    const command = this.history.pop();
    if (command) await command.undo();
  }
}
```

---

## 17. Template Method — Same Steps, Different Details

### The Analogy

Making tea and making coffee follow the same high-level steps: boil water → add the main ingredient → pour into cup → add extras. But the details differ — tea uses tea leaves, coffee uses coffee powder. The **template** defines the steps; subclasses fill in the details.

### When to Use

- Data import pipelines (validate → transform → save — but each source has different validation/transformation)
- Report generation (fetch data → process → format — but each report type differs)
- Authentication flows (receive credentials → validate → generate token — but each auth method validates differently)

```javascript
// Template: Data Import Pipeline
class DataImporter {
  // THE TEMPLATE — fixed sequence, subclasses fill in the steps
  async import(source) {
    const rawData = await this.fetchData(source);        // Step 1
    const validData = this.validate(rawData);             // Step 2
    const transformed = this.transform(validData);        // Step 3
    const result = await this.save(transformed);          // Step 4
    await this.notify(result);                            // Step 5
    return result;
  }

  // Subclasses MUST implement these
  async fetchData(source) { throw new Error('Not implemented'); }
  validate(data) { throw new Error('Not implemented'); }
  transform(data) { throw new Error('Not implemented'); }

  // Default implementations (can be overridden)
  async save(data) {
    return await db.collection(this.getCollection()).insertMany(data);
  }

  async notify(result) {
    console.log(`Imported ${result.insertedCount} records`);
  }
}

class CsvImporter extends DataImporter {
  async fetchData(source) {
    const file = fs.readFileSync(source);
    return csvParse(file);
  }

  validate(data) {
    return data.filter(row => row.email && row.name);
  }

  transform(data) {
    return data.map(row => ({
      name: row.name.trim(),
      email: row.email.toLowerCase(),
      importedAt: new Date(),
    }));
  }

  getCollection() { return 'csv_imports'; }
}

class ApiImporter extends DataImporter {
  async fetchData(source) {
    const response = await fetch(source);
    return response.json();
  }

  validate(data) {
    return data.filter(item => item.id && item.status === 'active');
  }

  transform(data) {
    return data.map(item => ({
      externalId: item.id,
      name: item.full_name,
      email: item.contact_email,
      importedAt: new Date(),
    }));
  }

  getCollection() { return 'api_imports'; }
}

// Usage
const csvImporter = new CsvImporter();
await csvImporter.import('./users.csv');

const apiImporter = new ApiImporter();
await apiImporter.import('https://api.external.com/users');
```

---

## 18. State — Behavior Changes When State Changes

### The Analogy

A traffic light changes its behavior based on its state. When it's green, cars go. When it's yellow, cars slow down. When it's red, cars stop. The traffic light doesn't have a giant if-else checking its current color — each state knows its own behavior and which state comes next.

### When to Use

- Order status workflows (pending → confirmed → shipped → delivered)
- Document approval workflows (draft → review → approved → published)
- User account states (active → suspended → banned)
- Game character states (idle → running → jumping → falling)
- **Anywhere you have an if-else chain based on the object's current state**

```javascript
// THE PROBLEM — state-based if-else chains
class Order {
  constructor() { this.status = 'pending'; }

  cancel() {
    if (this.status === 'pending') {
      this.status = 'cancelled';
    } else if (this.status === 'confirmed') {
      this.status = 'cancelled';
      this.refund();
    } else if (this.status === 'shipped') {
      throw new Error('Cannot cancel shipped order');
    } else if (this.status === 'delivered') {
      throw new Error('Cannot cancel delivered order');
    }
  }

  ship() {
    if (this.status === 'confirmed') {
      this.status = 'shipped';
    } else {
      throw new Error(`Cannot ship order in ${this.status} state`);
    }
  }
  // Every method has the same if-else pattern...
}
```

```javascript
// THE FIX — State Pattern

// Each state is its own class
class OrderState {
  cancel(order) { throw new Error(`Cannot cancel in ${this.name} state`); }
  confirm(order) { throw new Error(`Cannot confirm in ${this.name} state`); }
  ship(order) { throw new Error(`Cannot ship in ${this.name} state`); }
  deliver(order) { throw new Error(`Cannot deliver in ${this.name} state`); }
}

class PendingState extends OrderState {
  get name() { return 'pending'; }

  confirm(order) {
    order.setState(new ConfirmedState());
  }

  cancel(order) {
    order.setState(new CancelledState());
  }
}

class ConfirmedState extends OrderState {
  get name() { return 'confirmed'; }

  ship(order) {
    order.setState(new ShippedState());
  }

  cancel(order) {
    order.refund();
    order.setState(new CancelledState());
  }
}

class ShippedState extends OrderState {
  get name() { return 'shipped'; }

  deliver(order) {
    order.setState(new DeliveredState());
  }
  // cancel() inherits the error from OrderState — can't cancel shipped orders
}

class DeliveredState extends OrderState {
  get name() { return 'delivered'; }
  // All actions throw errors — delivered is a final state
}

class CancelledState extends OrderState {
  get name() { return 'cancelled'; }
}

// The Order delegates to its current state
class Order {
  constructor() {
    this.state = new PendingState();
  }

  setState(state) { this.state = state; }
  getStatus() { return this.state.name; }
  refund() { /* process refund */ }

  cancel() { this.state.cancel(this); }
  confirm() { this.state.confirm(this); }
  ship() { this.state.ship(this); }
  deliver() { this.state.deliver(this); }
}

// Usage
const order = new Order();
order.confirm();  // pending → confirmed
order.ship();     // confirmed → shipped
order.cancel();   // Error: Cannot cancel in shipped state
```

---

## 19. Chain of Responsibility — Pass It Along Until Someone Handles It

### The Analogy

A customer complaint at a company: "I want a refund of ₹50,000." The front desk agent can handle refunds up to ₹1,000. They pass it to the manager (up to ₹10,000). The manager passes it to the director (up to ₹100,000). The director approves it. Each person either handles the request or passes it up the chain.

### When to Use

- Middleware chains (Express.js, Spring interceptors)
- Approval workflows
- Request validation pipelines
- Log level filtering
- Exception handling chains

```javascript
// Express middleware IS the Chain of Responsibility pattern
app.use(cors());                    // Link 1: handle CORS
app.use(helmet());                  // Link 2: security headers
app.use(express.json());            // Link 3: parse JSON body
app.use(authenticate);              // Link 4: check auth token
app.use(rateLimiter);               // Link 5: rate limiting
app.use('/api', apiRouter);         // Link 6: actual route handling
app.use(errorHandler);              // Link 7: catch errors from above

// Each middleware either handles the request or calls next() to pass it along
function authenticate(req, res, next) {
  const token = req.headers.authorization;
  if (!token) return res.status(401).json({ error: 'Token required' });

  try {
    req.user = jwt.verify(token.replace('Bearer ', ''), SECRET);
    next();  // ← Pass to the next link in the chain
  } catch (err) {
    res.status(401).json({ error: 'Invalid token' });
    // NOT calling next() — this link handled the request (rejected it)
  }
}
```

---

## 20. Iterator — Walk Through a Collection Without Knowing Its Guts

### The Analogy

A playlist on Spotify. You press "Next" and it gives you the next song. You don't know if the playlist is stored as an array, a linked list, or fetched from a database. You just press Next and get the next item.

### When to Use

- Paginated database results
- Streaming large files line by line
- Processing items from a queue one by one
- Any time you need to traverse a collection without exposing its internal structure

```javascript
// JavaScript has built-in iterators via the Symbol.iterator protocol

// Custom iterator for paginated API results
class PaginatedResults {
  constructor(fetchPage) {
    this.fetchPage = fetchPage;
  }

  async *[Symbol.asyncIterator]() {
    let page = 1;
    let hasMore = true;

    while (hasMore) {
      const result = await this.fetchPage(page);
      for (const item of result.data) {
        yield item;
      }
      hasMore = result.hasMore;
      page++;
    }
  }
}

// Usage — seamlessly iterate through ALL pages
const users = new PaginatedResults(async (page) => {
  const res = await fetch(`/api/users?page=${page}&limit=100`);
  return res.json();
});

for await (const user of users) {
  console.log(user.name);  // Processes all users across all pages
}
```

---

## 21. Mediator — Stop Everyone from Talking to Everyone

### The Analogy

An airport control tower. Planes don't communicate with each other directly (chaos). Instead, every plane talks to the control tower, and the tower coordinates. The tower is the **mediator**.

### When to Use

- Chat rooms (users don't message each other directly — the chat room mediates)
- Form validation where fields depend on each other
- Microservice orchestration (an orchestrator coordinates services)

```javascript
// Chat room as a mediator
class ChatRoom {
  constructor() {
    this.users = new Map();
  }

  join(user) {
    this.users.set(user.name, user);
    user.chatRoom = this;
    this.broadcast(user, `${user.name} has joined the room`);
  }

  sendMessage(sender, recipientName, message) {
    const recipient = this.users.get(recipientName);
    if (recipient) {
      recipient.receive(sender.name, message);
    }
  }

  broadcast(sender, message) {
    for (const [name, user] of this.users) {
      if (name !== sender.name) {
        user.receive('System', message);
      }
    }
  }
}

class User {
  constructor(name) {
    this.name = name;
    this.chatRoom = null;
  }

  send(recipientName, message) {
    this.chatRoom.sendMessage(this, recipientName, message);
  }

  receive(from, message) {
    console.log(`[${this.name}] Message from ${from}: ${message}`);
  }
}
```

---

# PART 5 — PUTTING IT ALL TOGETHER

---

## 22. Clean Architecture & Layered Architecture

### The Problem

Without clear layers, your codebase becomes a bowl of spaghetti — controllers call databases directly, business logic lives in route handlers, and Express.js request objects leak into your database layer. Changing the web framework means rewriting everything.

### The Layered Architecture (Most Common in Backend)

```
┌─────────────────────────────────────────┐
│        Controller / Route Layer          │  ← Handles HTTP (req, res)
├─────────────────────────────────────────┤
│           Service / Use Case Layer       │  ← Business logic (pure logic, no HTTP)
├─────────────────────────────────────────┤
│         Repository / Data Access Layer   │  ← Database operations
├─────────────────────────────────────────┤
│              Domain / Entity Layer       │  ← Business objects (User, Order)
└─────────────────────────────────────────┘
```

**The Rules:**
1. Each layer only talks to the layer directly below it
2. The Controller never touches the database
3. The Service never knows about HTTP (no `req`, `res`, `status codes`)
4. The Repository never knows about business rules
5. Dependencies point inward — outer layers depend on inner layers, never the reverse

### Node.js Project Structure

```
src/
├── controllers/          ← HTTP layer (Express routes)
│   ├── userController.js
│   └── orderController.js
├── services/             ← Business logic
│   ├── userService.js
│   └── orderService.js
├── repositories/         ← Database operations
│   ├── userRepository.js
│   └── orderRepository.js
├── models/               ← Database schemas / entities
│   ├── User.js
│   └── Order.js
├── middleware/            ← Cross-cutting concerns
│   ├── auth.js
│   ├── errorHandler.js
│   └── validator.js
├── utils/                ← Shared helpers
├── config/               ← Configuration
└── app.js                ← App setup
```

```javascript
// controllers/userController.js — ONLY handles HTTP
class UserController {
  constructor(userService) {
    this.userService = userService;
  }

  async getUser(req, res, next) {
    try {
      const user = await this.userService.getUserById(req.params.id);
      res.status(200).json(user);
    } catch (error) {
      next(error);
    }
  }

  async createUser(req, res, next) {
    try {
      const user = await this.userService.createUser(req.body);
      res.status(201).json(user);
    } catch (error) {
      next(error);
    }
  }
}

// services/userService.js — ONLY business logic (no req, res, no SQL)
class UserService {
  constructor(userRepository, emailService) {
    this.userRepository = userRepository;
    this.emailService = emailService;
  }

  async getUserById(id) {
    const user = await this.userRepository.findById(id);
    if (!user) throw new NotFoundError('User not found');
    return user;
  }

  async createUser(data) {
    const existing = await this.userRepository.findByEmail(data.email);
    if (existing) throw new ConflictError('Email already registered');

    const hashedPassword = await bcrypt.hash(data.password, 10);
    const user = await this.userRepository.create({
      ...data,
      password: hashedPassword,
    });

    await this.emailService.sendWelcome(user);
    return user;
  }
}

// repositories/userRepository.js — ONLY database operations
class UserRepository {
  async findById(id) {
    return User.findById(id).select('-password');
  }

  async findByEmail(email) {
    return User.findOne({ email });
  }

  async create(data) {
    const user = new User(data);
    return user.save();
  }
}
```

### Spring Boot Project Structure

```
src/main/java/com/myapp/
├── controller/             ← HTTP layer
│   └── UserController.java
├── service/                ← Business logic
│   └── UserService.java
├── repository/             ← Data access
│   └── UserRepository.java
├── model/                  ← Entities
│   └── User.java
├── dto/                    ← Data Transfer Objects
│   ├── CreateUserRequest.java
│   └── UserResponse.java
├── exception/              ← Custom exceptions
│   ├── NotFoundException.java
│   └── GlobalExceptionHandler.java
├── config/                 ← Configuration
└── MyAppApplication.java
```

**Why DTOs?** The controller receives `CreateUserRequest` (only fields the client should send) and returns `UserResponse` (only fields the client should see — no password hash, no internal IDs). The `User` entity has everything including password, audit fields, and internal references. DTOs are the public face; entities are the internal reality.

---

## 23. Dependency Injection — Don't Create, Receive

### The One Rule

**Don't create your dependencies inside your class. Receive them from outside.**

```javascript
// BAD — class creates its own dependencies
class OrderService {
  constructor() {
    this.db = new MongoClient('mongodb://localhost:27017');     // Hardcoded
    this.emailer = new SendGridClient('SG.xxx');                // Hardcoded
    this.logger = new WinstonLogger({ level: 'info' });        // Hardcoded
  }
}

// GOOD — dependencies are injected
class OrderService {
  constructor(db, emailer, logger) {
    this.db = db;
    this.emailer = emailer;
    this.logger = logger;
  }
}

// Now you control what goes in:
// Production
const orderService = new OrderService(realDb, realEmailer, realLogger);
// Tests
const orderService = new OrderService(mockDb, mockEmailer, mockLogger);
```

In Spring Boot, DI is automatic — the framework handles it.

---

## 24. Refactoring — How to Fix Bad Code Without Breaking It

### The Process

1. **Write tests first** — before changing anything, make sure existing behavior is tested
2. **Make one small change** — rename, extract, move
3. **Run tests** — green? Continue. Red? Revert.
4. **Repeat**

### Common Refactoring Techniques

| Technique | When | Before → After |
|-----------|------|----------------|
| **Extract Method** | Long function | 50-line function → 5 short functions |
| **Extract Class** | Class does too much | God class → 3 focused classes |
| **Move Method** | Method uses another class's data | `OrderService.calculateTax()` → `TaxCalculator.calculate()` |
| **Replace Conditional with Polymorphism** | If-else based on type | If-else chain → Strategy/State pattern |
| **Introduce Parameter Object** | Too many function parameters | `fn(a, b, c, d, e)` → `fn(config)` |
| **Replace Magic Numbers** | Raw numbers in code | `if (age > 18)` → `if (age > MINIMUM_AGE)` |

---

## 25. How to Spot Which Pattern You Need — The Decision Guide

This is the section most people are looking for — the map from "I have this problem" to "use this pattern."

| Your Problem | Pattern | Why |
|-------------|---------|-----|
| I need only one instance of something | **Singleton** | Database pools, config, logger |
| I need to create objects without knowing the exact type | **Factory** | Creating payment processors, exporters, notifiers based on type |
| I need families of related objects that must be consistent | **Abstract Factory** | Database toolkits, UI theme sets |
| Object has too many constructor parameters | **Builder** | Query builders, config objects, HTTP requests |
| Creating an object is expensive and I need many similar ones | **Prototype** | Template-based configs, test data |
| Third-party library has a different interface than mine | **Adapter** | Wrapping Stripe, wrapping AWS SDK |
| I want to add behavior without modifying original code | **Decorator** | Caching, logging, retry, auth middleware |
| Complex system needs a simple interface | **Facade** | Order placement (inventory + pricing + payment + notification) |
| I need to control or enhance access to an object | **Proxy** | Caching proxy, protection proxy, lazy loading |
| I have a tree structure and need uniform treatment | **Composite** | Menus, file systems, organisational hierarchies |
| An operation can be done in multiple interchangeable ways | **Strategy** | Payment methods, pricing plans, sorting algorithms |
| One event should trigger multiple independent reactions | **Observer** | Order placed → email, inventory, analytics, notifications |
| I want to queue, undo, or log operations | **Command** | Task queues, undo/redo, transaction logging |
| Same algorithm structure, different details per type | **Template Method** | Data import pipelines, report generators |
| Object behavior changes based on its internal state | **State** | Order workflow, document approval, account status |
| Request should be handled by one of many handlers | **Chain of Responsibility** | Middleware, approval chains, validation |
| Need to traverse a collection without exposing internals | **Iterator** | Paginated results, streaming large data |
| Many objects communicating creates a tangled mess | **Mediator** | Chat rooms, form field dependencies |

### The Smell → Pattern Shortcut

| If You See This Smell... | Try This Pattern |
|--------------------------|-----------------|
| Growing if-else / switch on **type** | Strategy or Factory |
| Growing if-else / switch on **state** | State |
| One class doing everything | Extract classes (SRP) + Facade |
| Same code in many places | Template Method or Extract shared code |
| Hardcoded dependency (`new Stripe()`) | Adapter + Dependency Injection |
| Need to add logging/caching/auth without changing code | Decorator |
| Complex object creation | Builder or Factory |
| Multiple reactions to one event | Observer |

---

## 26. LLD Interview Questions (70+ Questions)

### Fundamentals

**Q1: What are SOLID principles? Explain each briefly.**
S = Single Responsibility (one class, one job). O = Open/Closed (extend without modifying). L = Liskov Substitution (child types work wherever parent is used). I = Interface Segregation (small focused interfaces, not one giant one). D = Dependency Inversion (depend on abstractions, not concrete classes). Together, they guide you toward code that's maintainable, testable, and flexible.

**Q2: What is the difference between a design pattern and a principle?**
Principles (SOLID, DRY, KISS) are guidelines about what good code looks like — they're the "rules." Patterns (Singleton, Strategy, Observer) are proven solutions to specific recurring problems — they're the "recipes." Principles tell you what to strive for; patterns tell you how to achieve it in specific situations.

**Q3: What is the difference between composition and inheritance?**
Inheritance is "is-a" (Dog IS an Animal). Composition is "has-a" (Car HAS an Engine). Inheritance creates tight coupling — changing the parent affects all children. Composition is looser — you can swap out the engine without redesigning the car. Modern best practice: "Favor composition over inheritance." Use inheritance for true type hierarchies; use composition for assembling capabilities.

**Q4: What is coupling? What is cohesion?**
Coupling is how much one class depends on another. Low coupling is good — classes can change independently. High coupling means changing one class forces changes in many others. Cohesion is how focused a class is on a single purpose. High cohesion is good — the class does one thing well. A God class has low cohesion. The ideal: high cohesion, low coupling.

**Q5: What does DRY mean? Can you over-apply it?**
Don't Repeat Yourself — avoid duplicating logic. But yes, you can over-apply it. If two pieces of code look similar but serve different purposes (e.g., user validation and admin validation), forcing them into one shared function creates coupling between unrelated features. The rule: don't repeat business logic, but don't force unrelated code together just because it looks similar.

**Q6: What is the difference between abstraction and encapsulation?**
Abstraction hides complexity by showing only the essential features. You use `fetch()` to make HTTP calls without knowing TCP/IP details. Encapsulation hides internal state and requires interaction through methods. A class has private fields and public methods — the outside world can't directly modify the internals. Abstraction is about hiding complexity; encapsulation is about hiding data.

**Q7: What are code smells? Name five.**
Code smells are signs of poor design. God Class (does everything), Long Method (50+ lines), Feature Envy (uses another class's data excessively), Primitive Obsession (strings/numbers instead of proper types), and Shotgun Surgery (one change requires editing many files). They're not bugs — the code works — but they predict future maintenance pain.

**Q8: Why is "favor composition over inheritance" recommended?**
Inheritance creates rigid hierarchies — a subclass is permanently tied to its parent. If the parent changes, all subclasses are affected. Multiple inheritance causes the "diamond problem." Composition lets you build behavior by combining objects — swap, replace, or extend pieces independently. It's like building with LEGO (composition) vs carving from one block of wood (inheritance).

---

### Pattern-Specific

**Q9: When would you use the Strategy pattern?**
When you have multiple algorithms or approaches for the same task and need to swap between them. Payment processing (credit card, UPI, wallet), sorting (by price, by rating, by date), pricing (flat, tiered, freemium), notification channels (email, SMS, push), file compression (gzip, brotli). The signal is an if-else chain that decides HOW to do something, where each branch is a different algorithm.

**Q10: When would you use Factory over Strategy?**
Factory is about creating objects — "give me the right type of object based on some condition." Strategy is about behavior — "execute this task using a specific algorithm." If you need to create a payment processor, use Factory. If you need to process a payment using a specific method, use Strategy. They often work together — a Factory creates the right Strategy.

**Q11: Explain the Observer pattern with a real example.**
When a user places an order, multiple things happen: confirmation email, inventory update, analytics tracking, loyalty points. Instead of the OrderService calling each of these directly (tight coupling), it emits an "order.placed" event. Independent listeners handle each reaction. Adding a new reaction (like sending a push notification) means adding a new listener — no change to OrderService.

**Q12: What is the Decorator pattern? How is it different from inheritance?**
Decorator adds behavior to an object at runtime by wrapping it, without modifying the original. Inheritance adds behavior at compile time by extending the class. With decorators, you can mix and match features (caching + logging + retry). With inheritance, you'd need `CachedLoggedRetryUserService` — and every combination becomes a new class. Decorators compose; inheritance explodes.

**Q13: What is the Singleton pattern? What are its problems?**
Singleton ensures only one instance exists. It's useful for DB pools and configs. Problems: hidden dependencies (classes secretly use a global instance), makes testing hard (can't substitute mock), tight coupling (every class depends on the same global), violates SRP (manages its own lifecycle AND its actual job). In modern code, prefer dependency injection over Singleton.

**Q14: Explain the Builder pattern. When would you NOT use it?**
Builder constructs complex objects step by step, letting you create different configurations using the same building process. Use it when objects have 4+ optional parameters or complex construction logic. Don't use it when the object is simple with 1-2 required parameters — a plain constructor is fine. Over-using Builder adds unnecessary complexity. If `new User(name, email)` is clear, you don't need a Builder.

**Q15: What is the Adapter pattern? Give a backend example.**
Adapter makes two incompatible interfaces work together. Backend example: your app uses a PaymentGateway interface with `charge(amount, currency, token)`. Stripe's SDK uses `charges.create({ amount, currency, source })`. A StripeAdapter implements your PaymentGateway interface and internally translates calls to Stripe's format. Switching to Razorpay means creating a new adapter, not rewriting your app.

**Q16: Explain Chain of Responsibility with a real example.**
Express.js middleware is Chain of Responsibility. A request passes through a chain: CORS handler → body parser → auth checker → rate limiter → route handler → error handler. Each middleware either handles the request (sends a response) or passes it to the next one (calls next()). Adding new cross-cutting concerns means adding new links — no changes to existing ones.

**Q17: What is the State pattern? How is it different from Strategy?**
Both eliminate if-else chains, but for different reasons. State changes behavior based on an object's internal state — an Order behaves differently when it's "pending" vs "shipped." Strategy changes behavior based on an external choice — which algorithm to use. With State, transitions happen automatically (order moves from pending to confirmed). With Strategy, the client explicitly chooses which strategy to use.

**Q18: When would you use the Template Method pattern?**
When you have a fixed sequence of steps but some steps vary across implementations. Data import pipeline: fetch → validate → transform → save. The sequence is always the same, but CSV import fetches differently from API import, and each validates differently. The template defines the skeleton; subclasses fill in the specific steps.

**Q19: What is the Facade pattern? Why is it useful in microservices?**
Facade provides a simple interface to a complex subsystem. In microservices, placing an order might need: check inventory (Service A), calculate price (Service B), process payment (Service C), send notification (Service D). Without a Facade, the client makes 4 calls. With a Facade, the client makes one call, and the Facade orchestrates the subsystem internally. This is essentially what an API Gateway or BFF does.

**Q20: Explain the Command pattern and where it's used in backends.**
Command wraps an action as an object with execute() and optionally undo(). In backends: job queues (each job is a Command), transaction logging (record each operation as a Command for replay), undo functionality (store Commands in a history stack), and scheduled tasks (Commands executed at specific times). RabbitMQ messages are essentially serialised Commands.

---

### Scenario-Based Questions

**Q21: Design a notification system that supports email, SMS, push, and WhatsApp.**
Use the Strategy pattern. Define a `NotificationChannel` interface with `send(user, message)`. Create `EmailChannel`, `SmsChannel`, `PushChannel`, `WhatsAppChannel`. Use a `NotificationService` that receives a map of channels (via DI). To add Telegram later, create `TelegramChannel` — no existing code changes. For sending via multiple channels, use the Observer pattern — emit an event, and each channel listens independently.

**Q22: Design a payment system that supports credit card, UPI, net banking, and wallets.**
Combine Factory + Strategy + Adapter. Define a `PaymentProcessor` interface. Each payment method is a Strategy. Use a `PaymentProcessorFactory` to create the right processor based on the method. If external providers have different APIs (Stripe, Razorpay, Paytm), use Adapters to standardise them behind your interface. Use the Observer pattern to trigger post-payment actions (email, inventory, analytics).

**Q23: You have a 500-line function that handles order processing. How do you refactor it?**
First, write tests to lock in current behavior. Then identify distinct responsibilities within the function (validation, pricing, payment, notification). Extract each into its own method, then its own class. Apply SRP — each class handles one responsibility. Use Facade to orchestrate them. The 500-line function becomes a 10-line orchestrator calling focused services. Run tests after each extraction.

**Q24: How would you design a file export system supporting PDF, CSV, Excel, and JSON?**
Factory + Strategy. Define an `Exporter` interface with `export(data)`. Create `PdfExporter`, `CsvExporter`, `ExcelExporter`, `JsonExporter`. Use an `ExporterFactory` to create the right one based on the requested format. In Spring Boot, all exporters are `@Component` beans — the factory auto-discovers them. Adding XML export means creating one new class.

**Q25: Design an order status workflow (pending → confirmed → shipped → delivered → returned).**
State pattern. Each status is a State class defining allowed transitions. `PendingState` allows confirm() and cancel(). `ConfirmedState` allows ship() and cancel(). `ShippedState` allows deliver(). `DeliveredState` allows return(). Invalid transitions throw errors. The Order delegates all actions to its current state. Adding a new state (like "out_for_delivery") means creating one new class and updating the transition from ShippedState.

**Q26: How would you add caching, logging, and retry logic to a database service without modifying it?**
Decorator pattern. Start with `UserRepository` (the base). Wrap it with `CachedUserRepository` (checks cache first, then delegates). Wrap that with `LoggedUserRepository` (logs before and after). Wrap that with `RetryUserRepository` (retries on failure). Each wrapper implements the same interface and delegates to the inner one. Order of wrapping matters — logging should see the retry, so logging wraps retry.

**Q27: Design a rule engine for an e-commerce discount system.**
Chain of Responsibility + Strategy. Define discount rules as a chain: `MinimumOrderRule` → `CouponCodeRule` → `MembershipRule` → `SeasonalRule` → `BulkOrderRule`. Each rule checks if it applies and adds a discount. The chain processes the order through all applicable rules. Each rule is a Strategy that calculates its specific discount. Adding a new rule is adding a new link in the chain.

**Q28: How would you design a data import system that handles CSV, JSON, XML, and API sources?**
Template Method. Define an `Importer` abstract class with a fixed pipeline: `fetchData()` → `validate()` → `transform()` → `save()` → `notify()`. Each source type (CSV, JSON, XML, API) extends this and implements the source-specific steps. The pipeline sequence is guaranteed consistent; the details vary.

**Q29: You're building a chat application. How would you design the backend classes?**
Mediator + Observer. The `ChatRoom` is the Mediator — users don't message each other directly. When a user sends a message, the ChatRoom routes it. Use Observer for online status updates — when a user goes online/offline, all their contacts are notified. Use Command for messages (they can be queued, retried, stored for history). Use Strategy for message delivery (WebSocket for online users, push notification for offline).

**Q30: Design a reporting system that generates daily/weekly/monthly reports with different formats.**
Template Method for the generation pipeline (fetch data → aggregate → format → deliver). Strategy for the aggregation logic (daily/weekly/monthly group differently). Factory for the formatter (PDF/HTML/CSV). Builder for the report configuration (date range, filters, columns, recipients). Observer to notify stakeholders when a report is ready.

---

### SOLID-Specific Questions

**Q31: Give a real example of SRP violation and how to fix it.**
A `UserController` that validates input, hashes passwords, queries the database, sends welcome emails, and returns the HTTP response violates SRP. Fix: extract `UserValidator` (validation), `UserRepository` (database), `EmailService` (email), `PasswordHasher` (hashing). The controller only handles HTTP concerns — accepting the request and sending the response. Each extracted class has one reason to change.

**Q32: How does the Open/Closed principle prevent bugs?**
When you modify existing code to add a feature, you risk breaking working features. OCP says extend, don't modify. Adding a new payment method as a new class that implements the PaymentProcessor interface means the existing credit card and UPI logic is untouched. No risk of accidentally breaking them. The new class is independently testable. Production code that already works stays stable.

**Q33: Why is Liskov Substitution important for polymorphism?**
Polymorphism only works if substituting a child for its parent doesn't surprise the calling code. If `Array.sort()` accepts a `Comparator`, every Comparator implementation must actually compare — not throw exceptions or return random values. LSP ensures that interfaces and abstract classes are meaningful contracts, not just syntax. Code that depends on the parent type can trust ALL child types.

**Q34: Give an example where Interface Segregation matters in Spring Boot.**
A fat `UserRepository` interface with CRUD methods, search methods, analytics methods, and export methods forces every implementing class to handle all of them. A read-only reporting service shouldn't depend on save/delete methods. Split into `UserReader`, `UserWriter`, `UserSearcher`, `UserAnalytics`. Each service depends only on the interfaces it actually uses. Changes to analytics don't affect the CRUD layer.

**Q35: How does Dependency Inversion enable testing?**
When `OrderService` depends on the `PaymentGateway` interface (not on Stripe directly), you can inject a `MockPaymentGateway` in tests that returns fake successful payments. No real API calls, no real charges, fast and deterministic tests. Without DIP, `OrderService` directly creates a Stripe client — you can't test without hitting the real Stripe API, which is slow, flaky, and charges real money.

---

### Quick-Fire Round

**Q36: Name the three categories of design patterns.** Creational (object creation), Structural (object composition), Behavioral (object communication).

**Q37: Which pattern does Express middleware implement?** Chain of Responsibility.

**Q38: Which pattern does Spring's @Cacheable implement?** Decorator (via AOP Proxy).

**Q39: Which pattern does Spring's DI container implement?** Factory (creating beans) + Singleton (one instance per bean by default).

**Q40: What is the difference between Factory Method and Abstract Factory?** Factory Method creates one type of object. Abstract Factory creates families of related objects that must be consistent.

**Q41: When should you NOT use a design pattern?** When the code is simple enough without it. Patterns add structure and flexibility but also complexity. If you have only 2 payment methods and will never add more, a simple if-else is fine. Don't use a pattern just because you can — use it because the problem demands it.

**Q42: What is the Hollywood Principle?** "Don't call us, we'll call you." High-level modules define the flow and call low-level modules — not the other way around. This is the foundation of frameworks (you don't call the framework, the framework calls your code), DI (the container calls your constructor), and the Observer pattern (the subject calls the observer).

**Q43: What is YAGNI? How does it relate to design patterns?** "You Aren't Gonna Need It." Don't add patterns for hypothetical future requirements. If you only have email notifications today and no plans for SMS, don't build a full Strategy/Observer system. Start simple, refactor to patterns when the complexity actually arrives. Premature abstraction is as dangerous as no abstraction.

**Q44: What is the difference between a Proxy and a Decorator?** Both wrap an object with the same interface. The difference is intent. A Proxy controls access (caching proxy, protection proxy, lazy-loading proxy). A Decorator adds behavior (logging, metrics, retry). A Proxy often creates or manages the wrapped object; a Decorator receives it from outside.

**Q45: What is the difference between Facade and Adapter?** Adapter makes one interface compatible with another (two existing things that don't match). Facade simplifies a complex subsystem into one easy interface (making something complex easier to use). Adapter converts; Facade simplifies.

**Q46: What is a Value Object?** An object defined by its value, not its identity. Two `Money(100, "INR")` objects are equal if amount and currency match — you don't care if they're the same instance. Value Objects are immutable. Examples: Email, Address, Money, DateRange. They eliminate Primitive Obsession.

**Q47: What is the difference between DTO, Entity, and Value Object?**
DTO (Data Transfer Object): carries data between layers (request/response shapes). Has no behavior. Entity: a domain object with identity and lifecycle (User with ID). Has behavior. Value Object: defined by value, not identity (Email, Money). Immutable. In a Spring Boot app, `CreateUserRequest` is a DTO, `User` is an Entity, `Email` could be a Value Object.

**Q48: What is the Principle of Least Knowledge (Law of Demeter)?**
"Only talk to your immediate friends." Don't chain method calls: `order.getCustomer().getAddress().getCity()`. Instead, `order.getDeliveryCity()`. Deep chaining creates tight coupling — if Address structure changes, every place that accesses it through Customer through Order breaks. Each object should only know about objects it directly works with.

**Q49: How do you decide between using an abstract class and an interface?**
Use an interface when: you define a contract that unrelated classes can implement, multiple inheritance is needed, or you want maximum flexibility. Use an abstract class when: you want to share code (default implementations) across related classes, you have a common base with some shared behavior and some that varies. In Java, prefer interfaces; use abstract classes when you need shared state or methods.

**Q50: What is Tell, Don't Ask?**
Instead of asking an object for its data and making decisions outside (`if (user.getRole() === 'admin') { ... }`), tell the object what to do and let it decide internally (`user.performAdminAction(action)`). This keeps logic inside the object that owns the data, improving encapsulation and reducing Feature Envy.

---

### Design & Architecture Questions

**Q51: What is the difference between a monolith and modular monolith?**
A monolith is one deployable unit — but it can be well-structured (modular) or poorly structured (big ball of mud). A modular monolith has clear internal boundaries — each module (users, orders, payments) has its own classes, interfaces, and database tables, communicating through defined interfaces. It's the benefits of microservices' separation without the operational complexity of distributed systems. Many experts recommend starting here before microservices.

**Q52: What is domain-driven design (DDD) at a high level?**
DDD is about modelling software to match the real business domain. Key concepts: Entities (objects with identity), Value Objects (objects defined by value), Aggregates (clusters of entities treated as one unit), Repositories (abstractions for data access), Services (business logic that doesn't belong to any entity), and Bounded Contexts (clear boundaries between different parts of the system). It's especially useful for complex business domains.

**Q53: What is the Repository pattern?**
An abstraction that hides data access details. Instead of your service writing SQL queries, it calls `userRepository.findByEmail(email)`. The repository handles whether the data comes from PostgreSQL, MongoDB, an API, or a cache. Benefits: service layer is testable (mock the repository), database can be swapped without changing business logic, and data access logic is centralised.

**Q54: What is an Aggregate in DDD?**
A cluster of domain objects treated as a single unit for data changes. An Order (the aggregate root) contains OrderItems, ShippingInfo, and PaymentInfo. External code can only access the aggregate through the root (the Order). You save the entire aggregate, not individual pieces. This ensures consistency — you can't have an OrderItem without an Order.

**Q55: Explain the difference between anemic and rich domain models.**
Anemic model: entities are just data containers (getters/setters only), and all logic lives in service classes. It's essentially procedural programming wrapped in classes. Rich model: entities contain behavior related to their data. `order.cancel()` instead of `orderService.cancelOrder(orderId)`. Rich models are better — they keep data and behavior together, following Tell Don't Ask and encapsulation.

---

### Testing & Patterns

**Q56: How do design patterns improve testability?**
DIP lets you inject mocks for external dependencies. Strategy lets you test each algorithm in isolation. Observer lets you test each listener independently. Factory lets you test creation logic separately from usage logic. SRP means each class has focused tests. Without patterns, testing a 500-line function requires setting up databases, APIs, email clients — slow, brittle, complex.

**Q57: What is a mock vs a stub vs a fake?**
A stub returns hardcoded responses (always returns the same user). A mock verifies interactions (checks that `sendEmail` was called exactly once with the right arguments). A fake is a simplified working implementation (an in-memory database instead of PostgreSQL). All three replace real dependencies in tests. The choice depends on what you're testing — behavior (mock), state (stub), or integration (fake).

**Q58: How would you test a class that uses the Observer pattern?**
Test the publisher and listeners separately. For the publisher (OrderService), verify it emits the right event with the right data. For each listener, pass a test event and verify the expected behavior (email sent, inventory updated). You don't test the entire chain together — that's an integration test. Unit tests test each piece in isolation.

**Q59: How do you test code that depends on time (expiry, scheduling)?**
Inject a Clock abstraction. Instead of `Date.now()`, use `clock.now()`. In production, inject a `SystemClock` that returns real time. In tests, inject a `FakeClock` where you control the time. This lets you test "what happens when the token expires" without waiting for real time to pass. Java has `Clock`; JavaScript needs a simple wrapper or library like `sinon.useFakeTimers()`.

**Q60: What is the difference between unit, integration, and end-to-end tests?**
Unit tests test a single function/class in isolation with all dependencies mocked — fast, many of these. Integration tests test multiple components working together (service + real database) — slower, fewer. End-to-end tests test the entire flow from HTTP request to database and back — slowest, fewest. The test pyramid: many unit tests, some integration, few E2E.

---

### Rapid-Fire Bonus Round

**Q61: What is the KISS principle?** Keep It Simple — choose the simplest solution that works.

**Q62: What is the difference between a library and a framework?** You call a library. A framework calls you (Inversion of Control).

**Q63: What is tight coupling? Give an example.** When a class directly depends on a concrete class: `new StripeClient()` inside your service. Changing Stripe to Razorpay requires rewriting the service.

**Q64: What is encapsulation? Why is it important?** Hiding internal state behind methods. It prevents external code from putting the object in an invalid state. `account.withdraw(amount)` validates the amount; direct access to `account.balance` doesn't.

**Q65: What is polymorphism?** One interface, multiple implementations. `paymentProcessor.charge()` behaves differently for credit card vs UPI vs wallet, but the caller uses the same method.

**Q66: What does "program to an interface, not an implementation" mean?** Your code should depend on what something does (the interface), not how it does it (the implementation). This allows swapping implementations without changing the dependent code.

**Q67: What is the difference between static and dynamic polymorphism?** Static (method overloading) is resolved at compile time. Dynamic (method overriding via interfaces/inheritance) is resolved at runtime. Dynamic is more relevant in design patterns.

**Q68: What is an anti-pattern?** A common "solution" that actually creates more problems. Examples: God Object, Spaghetti Code, Golden Hammer (using one pattern for everything), Copy-Paste programming.

**Q69: What is cohesion?** How closely related the responsibilities within a class are. High cohesion = class does one well-defined thing. Low cohesion = class does many unrelated things. Aim for high cohesion.

**Q70: When should you refactor?** When adding a feature is harder than it should be, when bug fixes create new bugs, when new team members can't understand the code, or when you notice code smells. Don't refactor "just because" — refactor when pain is real and measurable.

---

> **Final Advice for LLD Interviews:**
>
> Interviewers want to see your thought process, not memorised pattern names. When given a problem: first identify the code smells or requirements, then explain WHY a certain pattern fits (not just which one), then show the structure with classes and interfaces, and finally discuss trade-offs. A candidate who says "I'd use Strategy here because we have multiple interchangeable algorithms for pricing, and this will let us add new plans without modifying existing code" is far more impressive than one who just draws a UML diagram.
