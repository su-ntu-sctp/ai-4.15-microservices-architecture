# Lesson 4.15: Microservices Architecture - Part 1

## Learning Objectives

By the end of this lesson, you will be able to:

1. **Explain** the difference between monolithic and microservices architectures and when to use each
2. **Describe** how to decompose applications into microservices using business capability and domain strategies
3. **Implement** inter-service communication using synchronous REST calls with RestTemplate
4. **Apply** environment-based configuration management across multiple services

---

## Prerequisites

Before starting this lesson, ensure you have:

- Completed Lesson 4.6 (Docker Compose)
- Understanding of Spring Boot applications
- Basic knowledge of REST APIs
- Familiarity with Docker containers

---

## Introduction

So far in this module, you've built single applications (devops-demo, simple-crm). These are called **monolithic applications** — everything runs in one application.

But what happens when your application grows? When you have millions of users? When different teams work on different features? When you need to scale different parts differently?

This is where **microservices architecture** comes in.

In this lesson, you'll learn the theory behind microservices. In the next lesson, you'll build a working microservices system.

---

## Part 1: Monolithic Architecture

### What is a Monolithic Application?

A **monolith** is a single application where all components are tightly coupled and run as one unit.

**Example: E-commerce Application (Monolith)**

```
┌─────────────────────────────────────────────┐
│         E-commerce Application               │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │  User    │  │ Product  │  │  Order   │  │
│  │Management│  │  Catalog │  │Processing│  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                                              │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐  │
│  │ Payment  │  │Inventory │  │Shipping  │  │
│  │Processing│  │Management│  │Tracking  │  │
│  └──────────┘  └──────────┘  └──────────┘  │
│                                              │
│           Single Database                    │
│  ┌────────────────────────────────────────┐ │
│  │         PostgreSQL Database            │ │
│  └────────────────────────────────────────┘ │
└─────────────────────────────────────────────┘
         All deployed as ONE application
```

**Characteristics:**
- Single codebase
- Single deployment unit (one JAR/WAR file)
- Shared database
- All features in one application
- Deployed together

---

### Your Projects Are Monoliths

**devops-demo:**
```
┌─────────────────────┐
│   devops-demo.jar   │
│                     │
│  - DemoController   │
│  - Application code │
│  - Dependencies     │
└─────────────────────┘
    One application
```

**simple-crm:**
```
┌─────────────────────────┐
│     simple-crm.jar      │
│                         │
│  - CustomerController   │
│  - InteractionController│
│  - Repositories         │
│  - Services             │
│  - Single Database      │
└─────────────────────────┘
     One application
```

---

### Advantages of Monoliths

**1. Simple to Develop**
- Everything in one place
- Easy to understand
- Straightforward debugging
- All code in one IDE project

**2. Easy to Deploy**
- One JAR file
- Deploy once
- Simple CI/CD pipeline

**3. Easy to Test**
- Test entire application
- No network calls between components
- Straightforward integration tests

**4. Good Performance**
- No network latency
- Direct method calls
- Shared memory

**5. Perfect for Small Teams**
- 2-10 developers
- Everyone knows entire codebase
- Quick coordination

---

### Disadvantages of Monoliths

**1. Difficult to Scale**

Problem: Product catalog gets 1000 requests/second, but orders only get 10 requests/second.

```
Monolith scaling = Scale EVERYTHING together

┌─────────────┐  ┌─────────────┐  ┌─────────────┐
│  Monolith   │  │  Monolith   │  │  Monolith   │
│  Instance 1 │  │  Instance 2 │  │  Instance 3 │
│             │  │             │  │             │
│ All features│  │ All features│  │ All features│
└─────────────┘  └─────────────┘  └─────────────┘

Wastes resources - you scale ORDER processing even though
only PRODUCT catalog needs scaling!
```

**2. Long Deployment Times**

```
Small change in Payment module:
1. Rebuild entire application (10 minutes)
2. Run all tests (30 minutes)
3. Deploy entire application (10 minutes)
4. Restart takes 5 minutes

Total: 55 minutes for ONE small change!
```

**3. Technology Lock-in**

All components must use same:
- Programming language (e.g., all Java)
- Framework version (e.g., all Spring Boot 3.2)
- Database (e.g., all PostgreSQL)

**4. Team Coordination Nightmares**

```
50 developers working on one codebase:
- Merge conflicts constantly
- One team's bug breaks everyone's work
- Difficult to assign ownership
```

**5. Single Point of Failure**

```
Bug in Shipping module crashes entire application
     ↓
Users can't browse products, place orders, or make payments.
Everything is down!
```

---

### When to Use Monoliths?

**Good for:**
- ✅ Small applications
- ✅ Small teams (2-10 developers)
- ✅ MVPs (Minimum Viable Products)
- ✅ Prototypes and simple requirements
- ✅ Limited traffic

**Your projects (devops-demo, simple-crm) are PERFECT as monoliths!**

---

### Real-World Example: Amazon's Journey

**Amazon in 2001 (Monolith):** Single massive application. Difficult to scale, teams blocked each other, deployments took hours.

**Amazon in 2006+ (Microservices):** 100+ microservices. Each team owns services, deploys independently, and scales independently.

Reference: https://www.infoq.com/presentations/evolutionary-architecture-amazon/

---

## Part 2: Microservices Architecture

### What are Microservices?

**Microservices** = Breaking one large application into many small, independent services.

Each service:
- Runs independently
- Has its own database
- Can be deployed separately
- Communicates via APIs
- Owned by one team

---

### E-commerce as Microservices

```
┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│User Service  │  │Product Service│ │Order Service │
│   :8081      │  │    :8082      │ │    :8083     │
│              │  │               │ │              │
│ - GET /users │  │ - GET /products│ │ - POST /orders│
│ - POST /users│  │ - POST /products│ │ - GET /orders│
│              │  │               │ │              │
│  Users DB    │  │  Products DB  │ │  Orders DB   │
└──────────────┘  └──────────────┘  └──────────────┘
       ↑                  ↑                  ↑
       └──────────────────┴──────────────────┘
              HTTP/REST communication

Each service = Independent application
Each service = Own database
Each service = Can be deployed separately
```

---

### Key Characteristics of Microservices

**1. Single Responsibility** — Each service does ONE thing well.

**2. Independent Deployment** — Update one service without touching others.

**3. Decentralized Data** — Each service owns its data. Must communicate via APIs to access other services' data.

**4. Technology Diversity** — Different services can use different languages, frameworks, and databases.

**5. Fault Isolation** — One service crashing does not bring down the whole system.

---

### Advantages of Microservices

- **Independent Scaling** — Scale only the services that need it
- **Faster Development** — Teams work independently with no merge conflicts
- **Technology Flexibility** — Rewrite one service without touching others
- **Better Fault Tolerance** — One service down ≠ entire app down
- **Team Autonomy** — Each team owns, deploys, and decides technology for their service

---

### Disadvantages of Microservices

- **Increased Complexity** — 20+ services to deploy, monitor, and manage
- **Network Latency** — HTTP calls between services add overhead
- **Data Consistency Challenges** — Keeping data in sync across services requires careful design
- **Testing Complexity** — Must test service interactions, not just individual services
- **Operations Overhead** — Requires container orchestration, centralized logging, distributed tracing, and more

---

### When to Use Microservices?

**Good for:**
- ✅ Large applications with large teams (50+ developers)
- ✅ Different scaling needs per component
- ✅ Frequent, independent deployments
- ✅ High availability critical

**NOT good for:**
- ❌ Small applications and small teams
- ❌ MVPs/Prototypes
- ❌ Limited operations expertise

**Real-world examples:** Netflix (100+), Amazon (1000+), Uber (2000+) microservices.

---

## Part 3: Service Decomposition

### What is Service Decomposition?

**Decomposition** = Breaking a monolith into microservices. The key question: **How do you decide what becomes a service?**

---

### Decomposition Strategies

**1. Decompose by Business Capability**

Identify business functions and create services around them.

```
Customer Management  → User Service
Product Catalog      → Product Service
Order Processing     → Order Service
Payment Processing   → Payment Service
Inventory Management → Inventory Service
Shipping & Delivery  → Shipping Service
```

**2. Decompose by Subdomain (Domain-Driven Design)**

Identify core vs supporting domains:
```
Core Domain:       Account Management, Transaction, Loan Processing
Supporting Domain: Customer Support, Notification
Generic Domain:    Authentication, Logging
```

**3. Decompose by Use Case / Verb**
```
Post Content    → Publishing Service
Like/Comment    → Engagement Service
Search Content  → Search Service
Send Messages   → Messaging Service
```

**4. Decompose by Noun / Resource**
```
Users      → User Service
Products   → Product Service
Orders     → Order Service
```

---

### Decomposition Best Practices

**1. Start coarse-grained** — Don't create a `FirstNameService` and `LastNameService`. A `UserService` that manages all user fields is the right level.

**2. Avoid shared databases** — Services must communicate via APIs, not by reading each other's tables.

```
❌ Bad:             ✅ Good:
Both services       Each service has
share one DB        its own DB and
                    calls the other via API
```

**3. Follow Conway's Law** — "Organizations design systems that mirror their communication structure." Align services with team boundaries.

**4. Minimise inter-service calls** — Batch calls where possible to avoid chatty communication.

---

### Example: Decomposing simple-crm

**Current Monolith:**
```
simple-crm.jar → CustomerController + InteractionController + Single Database
```

**Decomposed:**
```
┌─────────────────┐       ┌─────────────────┐
│Customer Service │       │Interaction Svc  │
│    :8081        │       │    :8082        │
│                 │       │                 │
│ - GET /customers│◄──────│ Validates       │
│ - POST /customers│      │ customer exists │
│                 │       │ via REST call   │
│  Customer DB    │       │  Interaction DB │
└─────────────────┘       └─────────────────┘
```

---

### 🧑‍💻 Activity **(15 minutes)**

You are building a **School Management System**. On paper or a whiteboard, decompose it into microservices.

**The system needs to handle:**
- Student registration and profiles
- Course creation and enrolment
- Grade recording and reporting
- Attendance tracking
- Email notifications to students and parents

**Your task:**
1. Identify the microservices you would create (name them clearly)
2. List what data each service owns
3. Identify which services need to call each other and why
4. Are there any services that should NOT share a database? Why?

Be prepared to share and discuss your design with the class. There is no single correct answer — the goal is to think through the boundaries and justify your decisions.

---

## Part 4: RESTful Services in Microservices

### Why REST for Microservices?

REST is the most common way microservices communicate because it is simple, HTTP-based (firewall-friendly), language-independent, easy to test, and stateless.

### RESTful API Design Principles

**1. Resource-Based URLs**
```
✅ GET /users          ✅ POST /users
✅ GET /users/123      ✅ DELETE /users/123

❌ GET /getAllUsers     ❌ POST /createUser
```

**2. Use HTTP Methods Correctly**

| Method | Purpose | Example |
|---|---|---|
| GET | Retrieve data | GET /products |
| POST | Create new resource | POST /products |
| PUT | Update entire resource | PUT /products/123 |
| PATCH | Partial update | PATCH /products/123 |
| DELETE | Delete resource | DELETE /products/123 |

**3. Use HTTP Status Codes**

```
200 OK              - Success
201 Created         - Resource created
204 No Content      - Success, no body
400 Bad Request     - Invalid input
401 Unauthorized    - Not authenticated
403 Forbidden       - Not authorized
404 Not Found       - Resource not found
500 Internal Error  - Server error
```

**4. Version Your APIs**
```
/api/v1/users  →  /api/v2/users (when breaking changes needed)
```

**5. Use JSON for Data Exchange** — human-readable, language-independent, widely supported.

---

### Inter-Service Communication Example

**Scenario:** Order Service needs to validate a user before creating an order.

```java
@Service
public class OrderService {

    @Value("${user.service.url}")
    private String userServiceUrl;

    private RestTemplate restTemplate = new RestTemplate();

    public Order createOrder(Long userId, Long productId, int quantity) {

        // Call User Service to validate user exists
        String url = userServiceUrl + "/api/v1/users/" + userId;
        User user = restTemplate.getForObject(url, User.class);

        if (user == null) {
            throw new UserNotFoundException("User not found: " + userId);
        }

        Order order = new Order();
        order.setUserId(userId);
        order.setProductId(productId);
        order.setQuantity(quantity);
        order.setStatus("PENDING");

        return orderRepository.save(order);
    }
}
```

**Flow:**
```
Client → POST /api/v1/orders
         ↓
    Order Service
         ↓
    GET http://localhost:8081/api/v1/users/123
         ↓
    User Service returns user data
         ↓
    Order Service creates order
         ↓
    Returns order to client
```

---

## Part 5: Inter-Service Communication Patterns

### Pattern 1: Synchronous Communication (REST/HTTP)

Service waits for a response before continuing.

```
Order Service  ──GET /users/123──►  User Service
               ◄──User data──────
```

**Use when:** You need an immediate response (e.g. validating a user before creating an order).

**Pros:** Simple to implement, easy to debug, immediate feedback.
**Cons:** If the target service is down, the calling service fails. Network latency adds up.

---

### Pattern 2: Asynchronous Communication (Event-Driven)

Service publishes an event and continues without waiting.

```
Order Service  ──Publish "OrderCreated"──►  Message Queue  ──►  Email Service
               (continues immediately)                           Inventory Service
```

**Use when:** You don't need an immediate response (e.g. sending a confirmation email after an order is placed).

**Technologies:** RabbitMQ, Apache Kafka, AWS SQS.

**Pros:** Services are decoupled, better fault tolerance, scalable.
**Cons:** More complex, eventual consistency, harder to debug.

---

### Synchronous vs Asynchronous

| Aspect | Synchronous | Asynchronous |
|---|---|---|
| Coupling | Tight | Loose |
| Response | Immediate | Eventual |
| Complexity | Simple | Complex |
| Fault Tolerance | Lower | Higher |
| Use Case | Read / validate data | Notify events |

**Rule of thumb:**
- Need data right now → Synchronous (REST)
- Notifying multiple services of an event → Asynchronous (Events)

---

### Best Practices for Inter-Service Communication

**1. Use Timeouts**

```java
SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
factory.setConnectTimeout(3000); // 3 seconds
factory.setReadTimeout(3000);
restTemplate.setRequestFactory(factory);
```

**2. Implement Retries**

```java
@Retryable(
    value = {RestClientException.class},
    maxAttempts = 3,
    backoff = @Backoff(delay = 1000)
)
public User getUserById(Long userId) {
    return restTemplate.getForObject(url, User.class);
}
```

**3. Circuit Breaker (Optional — Advanced)**

Stops calling a failing service to prevent cascading failures. Implemented with libraries like Resilience4j. Not required for this module — covered in advanced DevOps topics.

---

## Part 6: Configuration Management

### The Configuration Challenge

Each microservice needs configuration — database URLs, API keys, service URLs, feature flags. With 20 services across multiple environments, managing this becomes a real challenge.

---

### Approach 1: Application Properties (Simple)

```properties
server.port=8081
spring.datasource.url=jdbc:postgresql://localhost:5432/users
user.service.url=http://localhost:8081
```

**Pros:** Simple, built into Spring Boot.
**Cons:** Hardcoded in the JAR — must rebuild and redeploy to change values.

---

### Approach 2: Environment Variables (Recommended)

```properties
server.port=${SERVER_PORT:8081}
spring.datasource.url=${DATABASE_URL}
user.service.url=${USER_SERVICE_URL}
```

Set values via Docker Compose:

```yaml
services:
  order-service:
    environment:
      SERVER_PORT: 8082
      DATABASE_URL: jdbc:postgresql://db:5432/orders
      USER_SERVICE_URL: http://user-service:8081
```

**Pros:** No rebuild needed, different values per environment, Docker-friendly.
**Cons:** Must restart the service to change values.

This is the approach we use in Lesson 4.16.

---

### Approach 3: Configuration Server (Optional — Advanced)

Spring Cloud Config Server provides centralised configuration with dynamic refresh — no restart needed. This is additional infrastructure and adds complexity. It is optional and not required for this module.

---

### Configuration Best Practices

**1. Use default values**
```properties
server.port=${SERVER_PORT:8080}
# If SERVER_PORT not set, use 8080
```

**2. Never hardcode secrets**
```properties
# ❌ Never do this:
database.password=mySecretPassword123

# ✅ Always do this:
database.password=${DATABASE_PASSWORD}
```

**3. Use Spring profiles for environment-specific settings**
```bash
java -jar app.jar --spring.profiles.active=prod
```

**4. Validate configuration at startup**
```java
@Configuration
@Validated
public class ServiceConfig {

    @NotNull
    @Value("${user.service.url}")
    private String userServiceUrl;
}
```

Application fails to start if required config is missing — catches errors early.

---

## Part 7: Microservices Challenges — Awareness Overview

> ℹ️ This section is an **awareness overview only**. These are real challenges in production microservices systems. You do not need to implement any of these in this module — they are covered in more advanced DevOps and architecture topics.

**Distributed Tracing** — When a request spans multiple services, tracing which service caused a failure or slowdown requires tools like Zipkin or Jaeger.

**Centralized Logging** — With 20+ services each producing log files, you need a centralized solution (ELK Stack, Splunk) to search and correlate logs across services.

**Data Consistency** — Distributed transactions are hard. If an order is placed but inventory update fails, you have an inconsistent state. The Saga pattern provides a solution.

**Testing Complexity** — Integration tests must cover service interactions. Contract testing (Pact) helps ensure services remain compatible.

---

## Summary

### Key Concepts

| Topic | Key Point |
|---|---|
| Monolith vs Microservices | Monolith = simple but limited; Microservices = flexible but complex |
| When to use each | Monolith for small teams/apps; Microservices for large teams/scale |
| Decomposition | Break by business capability, subdomain, use case, or resource |
| Communication | Synchronous (REST) for immediate data; Asynchronous (Events) for notifications |
| Configuration | Use environment variables; never hardcode secrets |

### Real-World Examples

**Companies using microservices:** Netflix (100+), Amazon (1000+), Uber (2000+), Spotify (hundreds).

**Success factors:** Large engineering teams, DevOps culture, strong tooling, gradual migration from monolith.

**"You can't build microservices if you can't build a monolith."**

---

## Additional Resources

- [Microservices.io](https://microservices.io/) — Patterns and best practices
- [Martin Fowler on Microservices](https://martinfowler.com/articles/microservices.html)
- [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)

---

END