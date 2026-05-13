# Topic 3: Microservices Architecture

**[← Back to Unit 3](README.md)**

## Overview

Microservices architecture addresses monolithic limitations by decomposing applications into independent, loosely-coupled services.

---

## Definition

**Microservices:**
> Small, independent, loosely-coupled services that work together to form a complete application. Each service is owned by a small team, uses its own database, and communicates via standardized interfaces (REST APIs, message queues).

**Core Principles:**
1. **Decomposition:** Break monolith into smaller services
2. **Business Alignment:** Each service handles specific business capability
3. **Loose Coupling:** Services communicate via APIs/interfaces, not shared code
4. **Database Per Service:** Each service has own database (ensures independence)
5. **Independent Deployment:** Services deployed independently

---

## Core Concepts

### 1. Application Split into Services

**Conceptual Breakdown:**

```
Monolithic E-commerce App
    ↓
    Decompose into microservices:
    ├── User Service (authentication, profiles)
    ├── Product Service (catalog, inventory)
    ├── Order Service (order management)
    ├── Payment Service (payment processing)
    ├── Notification Service (emails, SMS)
    ├── Recommendation Service (ML, suggestions)
    └── Shipping Service (delivery tracking)
```

### 2. Business-Aligned Services

**Not Technical Split, but Business Split:**

```
❌ Wrong Way (Technical):
├── API Layer Service
├── Database Layer Service
├── Cache Layer Service
└── These are interdependent

✅ Right Way (Business):
├── User Service (owns user data, users table, user API)
├── Order Service (owns order data, orders table, order API)
├── Payment Service (owns payment data, payment table, payment API)
```

### 3. Loose Coupling

**Services Are Independent:**

```
User Service can change:
├── Internal implementation (Python → Go)
├── Database (MySQL → MongoDB)
├── API response format
└── WITHOUT affecting other services

Other services only care about:
└── User Service API contract (endpoint, format)
```

### 4. Database Per Service

**Each Service Owns Its Data:**

```
User Service → MySQL (owns user data)
Order Service → MongoDB (owns order data)
Payment Service → PostgreSQL (owns payment data)

No shared database!
Ensures tight data ownership.
```

---

## Real-World Examples

### Amazon

Amazon decomposed its monolith into **hundreds of microservices**:

| Service | Purpose |
|---|---|
| **User Service** | Authentication, profiles, preferences |
| **Product Service** | Product catalog, inventory, descriptions |
| **Cart Service** | Shopping cart management |
| **Order Service** | Order creation, management, history |
| **Payment Service** | Payment processing, transactions |
| **Recommendation Service** | "You might like" suggestions |
| **Review Service** | Product reviews and ratings |
| **Shipping Service** | Delivery tracking, logistics |
| **Notification Service** | Order emails, SMS alerts |

**Result:**
- Each service owned by small team
- Can use different tech stacks
- Deploy hundreds of times per day
- Handle massive scale (millions of concurrent users)

### Netflix

Netflix architecture for video streaming:

| Service | Purpose |
|---|---|
| **User Service** | Account management, authentication |
| **Catalog Service** | Movie/show metadata, descriptions |
| **Recommendation Engine** | Personalized recommendations (ML) |
| **Streaming Service** | Video delivery, quality adaptation |
| **Billing Service** | Subscription management, payments |
| **Search Service** | Full-text search (Elasticsearch) |
| **Analytics Service** | Watch history, user behavior |

**Benefits:**
- Can serve content reliably to millions
- Recommendation engine uses custom ML stack
- Streaming optimized independently
- New features deployed without downtime

---

## Example: Online Shopping Platform

### System Architecture

```
┌──────────────────────────────────────────────────────┐
│            API Gateway (Entry Point)               │
│  Routes requests to appropriate microservices      │
└──────────────────┬───────────────────────────────────┘
         │
     ┌───┼───┬─────────┬─────────┬────────────────┐
     │   │   │         │         │                │
     ▼   ▼   ▼         ▼         ▼                ▼
┌────────┐┌──────────┐┌────────┐┌────────┐┌──────────┐
│User    ││Product   ││Order   ││Payment ││Notif.   │
│Service ││Service   ││Service ││Service ││Service  │
└────────┘└──────────┘└────────┘└────────┘└──────────┘
    │         │           │         │           │
┌───▼──┐ ┌─────▼───┐ ┌──▼─────┐ ┌─▼────┐  ┌────▼──┐
│MySQL │ │MongoDB  │ │MongoDB │ │PgSQL │  │Redis/ │
│      │ │         │ │        │ │      │  │Queue  │
└──────┘ └─────────┘ └────────┘ └──────┘  └───────┘
```

### Service Details

| Service | Database | Owns | API Endpoints |
|---|---|---|---|
| **User Service** | MySQL | Users, authentication, profiles | `GET /users/{id}` `POST /users` `PUT /users/{id}` |
| **Product Service** | MongoDB | Products, inventory, descriptions | `GET /products` `GET /products/{id}` `PUT /products/{id}` |
| **Order Service** | MongoDB | Orders, order items, order history | `POST /orders` `GET /orders/{id}` `GET /users/{id}/orders` |
| **Payment Service** | PostgreSQL | Payments, transactions, refunds | `POST /payments/process` `GET /payments/{id}` `POST /payments/{id}/refund` |
| **Notification Service** | Redis (queue) | Notification queue, delivery status | `POST /notifications/email` `POST /notifications/sms` |

---

## API Communication Example

### User Places an Order

```
User clicks "Buy Now"
    │
    ▼
Order Service receives request:
{
  "userId": 123,
  "productId": 456,
  "quantity": 2,
  "shippingAddress": "123 Main St"
}

    │
    ├─ 1. Validate User
    │     GET http://user-service:8001/users/123
    │     Response: { "id": 123, "name": "John", ... }
    │
    ├─ 2. Check Inventory
    │     PUT http://product-service:8002/products/456/reserve?qty=2
    │     Response: { "reserved": true, "remaining": 8 }
    │
    ├─ 3. Process Payment
    │     POST http://payment-service:8003/payments/process
    │     Body: { "orderId": 789, "amount": 99.99, "userId": 123 }
    │     Response: { "status": "approved", "transactionId": "TX123" }
    │
    ├─ 4. Send Notification
    │     POST http://notification-service:8004/notifications/email
    │     Body: { "userId": 123, "type": "order_confirmation", "orderId": 789 }
    │     Response: { "status": "queued" }
    │
    ▼
Order Service returns success:
{
  "orderId": 789,
  "status": "confirmed",
  "totalAmount": 99.99
}

    ▼
User sees confirmation
```

### Asynchronous Communication (Message Queue)

**Better approach for notifications:**

```
Order Service (synchronous):
1. Creates order in database
2. Updates inventory
3. Processes payment
4. Returns response

    │
    ▼ (Publish to message queue)

Message Queue (RabbitMQ/Kafka)
    │ (Asynchronous)
    ├─ Notification Service polls queue
    ├─ Consumes "order_created" message
    └─ Sends email/SMS independently

Benefits:
✓ Decoupled (don't wait for email)
✓ Resilient (service can be down, message persists)
✓ Scalable (multiple notification consumers)
```

---

## Database Per Service Pattern

### Architecture

```
Monolithic (Shared Database):
┌─────────────────┐
│ All Services    │
└────────┬────────┘
         │
    ┌────▼────┐
    │Single DB│
    └─────────┘

Problems:
- Services tightly coupled
- Hard to scale individually
- Schema changes affect all
- Can't optimize for each service
```

```
Microservices (Database Per Service):
┌──────────┐ ┌──────────┐ ┌──────────┐
│ User Svc │ │ Order Svc│ │ Payment  │
└────┬─────┘ └────┬─────┘ │ Svc      │
     │            │       └────┬─────┘
┌────▼──┐  ┌─────▼────┐   ┌───▼────┐
│MySQL  │  │ MongoDB  │   │PostgreSQL
└───────┘  └──────────┘   └────────┘

Benefits:
✓ Independent scaling
✓ Choose best DB per service
✓ Services loosely coupled
✓ Can modify schema independently
```

### Service Database Mapping

| Service | Database Type | Why | Stores |
|---|---|---|---|
| **User Service** | MySQL (Relational) | ACID needed, complex queries | Usernames, passwords, profiles |
| **Product Service** | MongoDB (Document) | Flexible schema, product variants | Products, inventory, descriptions |
| **Order Service** | MongoDB (Document) | Nested order items, flexible schema | Orders, order items, history |
| **Payment Service** | PostgreSQL (Relational) | Financial data, strong consistency | Transactions, invoices, receipts |
| **Analytics Service** | Elasticsearch (Search) | Full-text search, complex queries | Event logs, user behavior |
| **Cache Layer** | Redis (Cache) | Performance, session data | Sessions, temporary data |

### Loose Coupling Benefit

**Schema Change Example:**

```
User Service wants to migrate from MySQL to MongoDB:

Monolithic:
├── Change database type
├── Test ALL features (payments, orders, etc. depend on it)
├── Risk: Something breaks in Order Service
└── Must redeploy entire application

Microservices:
├── Change User Service database (MySQL → MongoDB)
├── Update User Service code
├── Deploy only User Service
├── Other services: No changes! Still call same API
├── Risk: Low (isolated to User Service)
└── Result: Safe, easy migration
```

**API Contract Preserved:**
```
# Old User Service (MySQL)
GET http://user-service/users/123
Response: { "id": 123, "name": "John", "email": "j@example.com" }

# New User Service (MongoDB) - Same API
GET http://user-service/users/123
Response: { "id": 123, "name": "John", "email": "j@example.com" }

# Order Service continues working - no changes needed!
# Payment Service continues working - no changes needed!
```

---

## Advantages of Microservices

### 1. Independent Scalability

**Problem (Monolithic):**
```
Black Friday: 10x traffic overall
But: Order processing is bottleneck, Product Service has capacity

Solution: Scale entire app 10x
Cost: Wasteful, pay for unused Product Service capacity
```

**Solution (Microservices):**
```
Same scenario: Scale services independently
├── Order Service: 10x (bottleneck)
├── Payment Service: 8x (moderate load)
├── Product Service: 2x (has capacity)
├── Notification Service: 3x

Cost: Much lower, only scale what needs it
```

---

### 2. Independent Deployment

**Ability to deploy one service without deploying others:**

```
Monolithic Deployment:
├── Fix bug in Order Service
├── Rebuild entire application
├── Run full test suite
├── Deploy entire application
├── Risk: Any bug in any service crashes everything
└── Deployment window: High risk

Microservices Deployment:
├── Fix bug in Order Service
├── Rebuild Order Service only
├── Run Order Service tests only
├── Deploy Order Service only
├── Risk: Limited to Order Service impact
└── Deployment window: Safe, frequent deployments
```

**Real Example:**
```
Amazon: Deploy hundreds of times per day
└── Each team can deploy independently
    without coordinating with others
```

---

### 3. Technology Flexibility

**Each service can use best technology for its job:**

```
E-commerce Application:
├── User Service: Python (simple, prototyping fast)
├── Product Service: Java (mature, stable, good libraries)
├── Payment Service: Go (fast, concurrent, handles load)
├── Search Service: Elasticsearch (full-text search optimized)
├── ML Recommendation: Python/TensorFlow (ML ecosystem)
└── Mobile API: Node.js (JavaScript everywhere)

Monolithic Equivalent:
├── Choose single language for entire app
├── Compromise: Not optimal for any service
└── Stuck with that choice for years
```

---

### 4. Fault Isolation

**One service failure doesn't crash entire system:**

```
Monolithic:
├── Bug in Recommendation Service
├── Crashes entire application
└── Everything down: Orders, Payments, Product Browse

Microservices:
├── Bug in Recommendation Service
├── Only Recommendation Service fails
├── Users can still: Browse products, place orders, pay
├── Graceful degradation: "Recommendations unavailable"
└── Other services continue normally
```

---

### 5. Parallel Development

**Multiple teams work independently:**

```
Monolithic (Single Codebase):
├── Team A wants to refactor Product logic
├── Team B wants to refactor Order logic
├── Merge conflicts daily
├── Must coordinate changes
├── Reduced velocity

Microservices (One Service per Team):
├── Team A owns and maintains Product Service
├── Team B owns and maintains Order Service
├── No merge conflicts (separate repositories)
├── Can deploy independently
├── Full autonomy → faster development
```

---

### 6. Easier Maintenance

**Smaller, focused services:**

```
Monolithic: 500k lines of code
├── Hard to understand any part
├── Bug fix may break other parts
├── Hard to test changes
└── High risk

Microservices: 50k lines per service
├── Easy to understand service
├── Isolated testing
├── Low risk of side effects
└── Quick debugging
```

---

### Comprehensive Advantages Table

| # | Advantage | Explanation | Example | Benefit |
|---|---|---|---|---|
| **1** | **Independent Scalability** | Each service scales independently based on demand | Holiday sales: Scale Order Service 10x, Product 2x | Efficient resource use, lower costs |
| **2** | **Independent Deployment** | Deploy one service without redeploying all | Fix Payment Service without touching User Service | Faster releases, less downtime, less risk |
| **3** | **Technology Flexibility** | Each service uses optimal tech stack | ML in Python, API in Go, Search in Elasticsearch | Best-fit tools per service requirement |
| **4** | **Fault Isolation** | One service failure doesn't crash system | Recommendation fails; orders still work | Higher reliability, graceful degradation |
| **5** | **Parallel Development** | Teams work independently | Team A on Payment, Team B on Orders | Faster time-to-market, better agility |
| **6** | **Easier Maintenance** | Smaller, focused services easier to test/fix | 50k lines vs 500k lines per service | Faster debugging, lower risk |
| **7** | **Business Alignment** | Services map to business capabilities | Each team owns a business function | Better organization, clear ownership |
| **8** | **Tech Upgrade Path** | Can upgrade/replace individual services | Update Payment Service Java version | Easier tech updates, less technical debt |

---

## Microservices vs Monolithic — Complete Comparison

| Feature | Monolithic | Microservices |
|---|---|---|
| **Deployment** | Single unit deployed together | Each service deployed independently |
| **Scaling** | Scale entire application | Scale individual services as needed |
| **Tech Stack** | Single technology stack for all | Polyglot (multiple technologies) |
| **Failure Impact** | One bug → entire app down | Service failure → isolated impact |
| **Team Structure** | Single large team (50+) | Multiple small teams (5-10 each) |
| **Database** | Single shared database | Separate database per service |
| **Development Speed (Initial)** | Very fast to build MVP | Slower initial setup |
| **Development Speed (Scaling)** | Slows down significantly | Maintains fast velocity |
| **Complexity** | Simple to start, complex later | Complex from start, but manageable |
| **Deployment Risk** | High (entire app at risk) | Low (isolated service risk) |
| **Debugging** | Easier (single codebase) | Harder (distributed tracing needed) |
| **Performance** | Fast (in-process calls, milliseconds) | Slightly slower (network calls, microseconds) |
| **Dependency Management** | Tight coupling | Loose coupling |
| **Testing** | Single unit tests | Integration tests between services |
| **Monitoring** | Monitor single application | Monitor multiple services |
| **When to Use** | Small projects, startups | Large systems, multiple teams, scale |
| **Learning Curve** | Easier | Steeper (distributed systems complexity) |
| **Operational Overhead** | Low | High (more services to manage) |

---

## Decision Matrix: When to Use

### Use Monolithic When:
- ✅ Team size < 10 people
- ✅ Application simple (< 50k lines)
- ✅ Deploy once per month
- ✅ Single technology adequate
- ✅ High performance locally (in-process calls) critical
- ✅ Limited operational budget

### Use Microservices When:
- ✅ Team size > 20 people (multiple teams)
- ✅ Application complex (100k+ lines)
- ✅ Deploy multiple times per day
- ✅ Need technology flexibility
- ✅ Scaling individual components important
- ✅ High availability critical
- ✅ Can afford operational complexity

---

## Key Takeaway

Microservices architecture solves monolithic problems by decomposing applications into independent services. 

**Trade-off:**
```
Monolithic: Simple → Complex at scale
Microservices: Complex setup → Manages complexity well at scale
```

**Benefits increase as:**
- Team size grows
- Application complexity increases
- Scaling requirements increase
- Feature deployment frequency increases

---

## Next Topic

**[Read about Containers →](04_Containers.md)**

Learn how containers enable microservices by providing lightweight, isolated execution environments.
