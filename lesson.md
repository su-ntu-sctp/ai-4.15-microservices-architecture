# Lesson 4.15: Microservices Architecture - Part 1 

## Learning Objectives

By the end of this lesson, you will be able to:

1. **Explain** the difference between monolithic and microservices architectures
2. **Understand** when to use microservices vs monoliths
3. **Describe** how to decompose applications into microservices
4. **Explain** RESTful services and their role in microservices
5. **Understand** inter-service communication patterns
6. **Describe** configuration management in distributed systems

---

## Prerequisites

Before starting this lesson, ensure you have:

- Completed Lesson 4.6 (Docker Compose)
- Understanding of Spring Boot applications
- Basic knowledge of REST APIs
- Familiarity with Docker containers

---

## Introduction

So far in this module, you've built single applications (devops-demo, simple-crm). These are called **monolithic applications** - everything runs in one application.

But what happens when your application grows? When you have millions of users? When different teams work on different features? When you need to scale different parts differently?

This is where **microservices architecture** comes in.

In this lesson, you'll learn the theory behind microservices. In the next lesson, you'll build a working microservices system.

---

## Part 1: Monolithic Architecture (20 minutes)

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

Can't use:
- Python for machine learning module
- Node.js for real-time chat
- MongoDB for product catalog

**4. Team Coordination Nightmares**

```
50 developers working on one codebase:
- Merge conflicts constantly
- One team's bug breaks everyone's work
- Difficult to assign ownership
- Testing changes affect everyone
- Deploy coordination needed
```

**5. Single Point of Failure**

```
Bug in Shipping module crashes entire application
     ↓
Users can't:
- Browse products ✗
- Place orders ✗
- Make payments ✗
- Everything down!
```

---

### When to Use Monoliths?

**Good for:**
- ✅ Small applications
- ✅ Small teams (2-10 developers)
- ✅ MVPs (Minimum Viable Products)
- ✅ Prototypes
- ✅ Simple requirements
- ✅ Limited traffic

**Examples:**
- Internal company tools
- Small e-commerce sites
- Simple CRM systems
- Portfolio websites
- Small SaaS products

**Your projects (devops-demo, simple-crm) are PERFECT as monoliths!**

---

### Real-World Example: Amazon's Journey

**Amazon in 2001 (Monolith):**
- Single massive application
- Difficult to scale
- Teams blocked each other
- Deployments took hours

**Amazon in 2006+  (Microservices):**
- 100+ microservices
- Each team owns services
- Deploy independently
- Scale independently

**Reference:**
https://www.infoq.com/presentations/evolutionary-architecture-amazon/

---

## Part 2: Microservices Architecture (25 minutes)

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

**Same application, different architecture:**

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

┌──────────────┐  ┌──────────────┐  ┌──────────────┐
│Payment Service│ │Inventory Svc  │ │Shipping Svc  │
│   :8084      │  │    :8085      │ │    :8086     │
└──────────────┘  └──────────────┘  └──────────────┘

Each service = Independent application
Each service = Own database
Each service = Can be deployed separately
```

---

### Key Characteristics of Microservices

**1. Single Responsibility**

Each service does ONE thing well.

```
✅ Good:
- User Service → Manages users only
- Product Service → Manages products only
- Order Service → Manages orders only

❌ Bad:
- User-Product-Order Service → Does everything
  (This is just a monolith!)
```

**2. Independent Deployment**

```
Update Payment Service:
1. Build Payment Service (2 min)
2. Test Payment Service (5 min)
3. Deploy Payment Service (2 min)
4. Other services keep running!

Total: 9 minutes
NO impact on other services! ✅
```

**3. Decentralized Data**

Each service owns its data.

```
┌─────────────┐         ┌─────────────┐
│User Service │         │Order Service│
│             │         │             │
│  Users DB   │         │  Orders DB  │
└─────────────┘         └─────────────┘

User Service CANNOT directly access Orders DB
Order Service CANNOT directly access Users DB

Must communicate via APIs!
```

**4. Technology Diversity**

Different services can use different technologies:

```
User Service:     Java + Spring Boot + PostgreSQL
Product Service:  Python + Flask + MongoDB
Payment Service:  Node.js + Express + Redis
Shipping Service: Go + PostgreSQL
```

Each team chooses best tool for their service!

**5. Fault Isolation**

```
Payment Service crashes:
- Users can still browse products ✅
- Users can add to cart ✅
- Users can view orders ✅
- Payment fails gracefully ✗
- Show friendly error message
- Other services unaffected!
```

---

### Advantages of Microservices

**1. Independent Scaling**

```
Product catalog: 10,000 requests/sec → Scale to 10 instances
Order processing: 100 requests/sec → Run 2 instances

┌────────┐ ┌────────┐ ┌────────┐ ... (10 instances)
│Product │ │Product │ │Product │
│Service │ │Service │ │Service │
└────────┘ └────────┘ └────────┘

┌────────┐ ┌────────┐
│Order   │ │Order   │  (Only 2 instances)
│Service │ │Service │
└────────┘ └────────┘

Cost-effective scaling! ✅
```

**2. Faster Development**

```
Team A works on User Service:
- Changes don't affect Team B
- Deploy independently
- No coordination needed
- Faster development

Team B works on Order Service:
- Different codebase
- Different deployment schedule
- Independent testing
```

**3. Technology Flexibility**

```
Old Payment Service (Java):
- Slow
- Complex codebase
- Hard to maintain

Rewrite in Node.js:
- Replace ONLY Payment Service
- Other services unchanged
- Gradual migration possible
```

**4. Better Fault Tolerance**

```
Monolith: One bug = Entire app down
Microservices: One service down = 90% still works
```

**5. Team Autonomy**

```
Each team owns their services:
- Choose technology
- Set own deployment schedule
- Make architectural decisions
- Responsible for their service
```

---

### Disadvantages of Microservices

**1. Increased Complexity**

```
Monolith:
- 1 application to deploy
- 1 database to manage
- 1 server to monitor

Microservices:
- 20+ services to deploy
- 20+ databases to manage
- 20+ servers to monitor
- Network communication to handle
- Service discovery needed
```

**2. Network Latency**

```
Monolith:
getUserOrder() → Direct method call (1ms)

Microservices:
Order Service → HTTP call to User Service (50ms)
50x slower!
```

**3. Data Consistency Challenges**

```
Monolith:
Database transaction ensures consistency

Microservices:
- User Service has user data
- Order Service has order data
- How to keep them in sync?
- Eventual consistency needed
```

**4. Testing Complexity**

```
Monolith:
- Test one application
- Integration tests straightforward

Microservices:
- Test 20 services
- Test service interactions
- Mock external services
- End-to-end testing complex
```

**5. Operations Overhead**

```
Need:
- Container orchestration (Kubernetes)
- Service mesh
- Centralized logging
- Distributed tracing
- API Gateway
- Service discovery
- Configuration management

Lot of infrastructure! 
```

---

### When to Use Microservices?

**Good for:**
- ✅ Large applications
- ✅ Large teams (50+ developers)
- ✅ Different scaling needs
- ✅ Frequent deployments
- ✅ Technology diversity needed
- ✅ High availability critical

**Examples:**
- Netflix (100+ microservices)
- Amazon (1000+ microservices)
- Uber (2000+ microservices)
- Large e-commerce platforms
- Banking systems
- Social media platforms

**NOT good for:**
- ❌ Small applications
- ❌ Small teams
- ❌ MVPs/Prototypes
- ❌ Simple requirements
- ❌ Limited operations expertise

---

### Microservices Architecture Diagram

```
┌────────────────────────────────────────────────────────────┐
│                  Microservices Ecosystem                    │
└────────────────────────────────────────────────────────────┘

                    ┌─────────────┐
                    │ API Gateway │ ← Single entry point
                    └──────┬──────┘
                           │
        ┌──────────────────┼──────────────────┐
        │                  │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │ User    │       │Product  │       │ Order   │
   │ Service │◄─────►│ Service │◄─────►│ Service │
   └────┬────┘       └────┬────┘       └────┬────┘
        │                 │                  │
   ┌────▼────┐       ┌────▼────┐       ┌────▼────┐
   │ User DB │       │Product  │       │ Order   │
   └─────────┘       │   DB    │       │   DB    │
                     └─────────┘       └─────────┘

Each service:
- Independent codebase
- Own database
- Separate deployment
- Communicates via REST APIs
```

**Reference Image:**
https://microservices.io/i/Microservice_Architecture.png

---

## Part 3: Service Decomposition (30 minutes)

### What is Service Decomposition?

**Decomposition** = Breaking a monolith into microservices.

**Question:** How do you decide what becomes a service?

---

### Decomposition Strategies

**1. Decompose by Business Capability**

Identify business functions and create services around them.

**E-commerce business capabilities:**
```
┌──────────────────────────────────────────┐
│         E-commerce Business              │
├──────────────────────────────────────────┤
│                                          │
│  Customer Management  → User Service     │
│  Product Catalog      → Product Service  │
│  Order Processing     → Order Service    │
│  Payment Processing   → Payment Service  │
│  Inventory Management → Inventory Service│
│  Shipping & Delivery  → Shipping Service │
│  Notifications        → Notification Svc │
│                                          │
└──────────────────────────────────────────┘

Each business capability = One microservice
```

**Benefits:**
- ✅ Clear boundaries
- ✅ Aligned with business
- ✅ Easy to understand

---

**2. Decompose by Subdomain (Domain-Driven Design)**

Use Domain-Driven Design (DDD) to identify subdomains.

**Example: Online Banking**

```
Core Domain (Critical):
- Account Management Service
- Transaction Service
- Loan Processing Service

Supporting Domain:
- Customer Support Service
- Notification Service

Generic Domain:
- Authentication Service
- Logging Service
```

**Benefits:**
- ✅ Focuses on core business
- ✅ Clear priorities
- ✅ Better resource allocation

---

**3. Decompose by Verb/Use Case**

Create services based on actions/use cases.

**Example: Social Media Platform**

```
Use Cases:
- Post Content      → Publishing Service
- Like/Comment      → Engagement Service
- Follow Users      → Social Graph Service
- Search Content    → Search Service
- Send Messages     → Messaging Service
- Upload Media      → Media Service
```

**Benefits:**
- ✅ User-centric design
- ✅ Clear responsibilities
- ✅ Easy to map features

---

**4. Decompose by Noun/Resource**

Create services around data entities.

**Example: E-commerce**

```
Resources:
- Users             → User Service
- Products          → Product Service
- Orders            → Order Service
- Reviews           → Review Service
- Inventory         → Inventory Service
```

**Benefits:**
- ✅ Natural boundaries
- ✅ Clear data ownership
- ✅ Easy to understand

---

### Decomposition Best Practices

**1. Start with Coarse-Grained Services**

❌ **Don't:**
```
- FirstNameService
- LastNameService
- EmailService
- PhoneService
(Too granular!)
```

✅ **Do:**
```
- User Service
  - Manages first name, last name, email, phone
  (Right granularity!)
```

**2. Consider Team Structure**

```
Team A (5 people) → User Service + Auth Service
Team B (5 people) → Product Service + Inventory Service
Team C (5 people) → Order Service + Payment Service

Each team owns 1-2 services
```

**Conway's Law:** "Organizations design systems that mirror their communication structure."

**3. Identify Bounded Contexts**

```
"Customer" in different contexts:

User Service:
- Customer = Login credentials, profile

Order Service:
- Customer = Shipping address, order history

Payment Service:
- Customer = Payment methods, billing info

Same concept, different data = Different services!
```

**4. Avoid Shared Databases**

```
❌ Bad:
┌────────┐         ┌────────┐
│User    │         │Order   │
│Service │         │Service │
└────┬───┘         └────┬───┘
     └──────┬───────────┘
         ┌──▼────┐
         │Shared │
         │  DB   │
         └───────┘
Tight coupling!

✅ Good:
┌────────┐         ┌────────┐
│User    │         │Order   │
│Service │◄───────►│Service │
└────┬───┘   API   └────┬───┘
  ┌──▼──┐           ┌───▼──┐
  │User │           │Order │
  │ DB  │           │  DB  │
  └─────┘           └──────┘
Loose coupling via APIs!
```

**5. Minimize Inter-Service Calls**

```
❌ Bad Design:
Order Service → calls User Service
             → calls Product Service
             → calls Inventory Service
             → calls Payment Service
             → calls Shipping Service
(5 network calls for one order!)

✅ Better Design:
Order Service → calls Payment Service only
             → Stores necessary user/product data locally
(1 network call, data duplication accepted)
```

---

### Example: Decomposing simple-crm

**Current Monolith (simple-crm):**
```
┌─────────────────────────┐
│     simple-crm.jar      │
│                         │
│  - CustomerController   │
│  - InteractionController│
│  - Repositories         │
│  - Services             │
│  - Database             │
└─────────────────────────┘
```

**Decomposed into Microservices:**
```
┌─────────────────┐       ┌─────────────────┐
│Customer Service │       │Interaction Svc  │
│    :8081        │       │    :8082        │
│                 │       │                 │
│ - GET /customers│       │ - GET /interactions│
│ - POST /customers│      │ - POST /interactions│
│                 │       │                 │
│  Customer DB    │       │  Interaction DB │
└─────────────────┘       └─────────────────┘
         ↑                         ↑
         └─────────────────────────┘
              REST communication

When creating interaction:
- Interaction Service calls Customer Service
- Validates customer exists via API
- Stores interaction in own DB
```

**Benefits:**
- Customer team works independently
- Interaction team works independently
- Scale each separately
- Deploy each separately

---

## Part 4: RESTful Services in Microservices (25 minutes)

### Why REST for Microservices?

**REST (Representational State Transfer)** is the most common way microservices communicate.

**Advantages:**
- ✅ Simple and well-understood
- ✅ HTTP-based (firewall-friendly)
- ✅ Language-independent
- ✅ Easy to test (Postman, curl)
- ✅ Cacheable
- ✅ Stateless

---

### RESTful API Design Principles

**1. Resource-Based URLs**

```
✅ Good:
GET    /users           - List all users
GET    /users/123       - Get user 123
POST   /users           - Create user
PUT    /users/123       - Update user 123
DELETE /users/123       - Delete user 123

❌ Bad:
GET    /getAllUsers
GET    /getUserById?id=123
POST   /createUser
POST   /updateUser
POST   /deleteUser
```

**2. Use HTTP Methods Correctly**

| Method | Purpose | Example |
|--------|---------|---------|
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
503 Service Unavailable - Service down
```

**4. Version Your APIs**

```
✅ Good:
/api/v1/users
/api/v2/users  (when breaking changes needed)

Benefits:
- Old clients keep working
- Gradual migration
- Backward compatibility
```

**5. Use JSON for Data Exchange**

```json
{
  "id": 123,
  "firstName": "John",
  "lastName": "Doe",
  "email": "john@example.com"
}
```

**Why JSON?**
- Human-readable
- Language-independent
- Easy to parse
- Widely supported

---

### Microservices API Example

**User Service API:**

```
Base URL: http://localhost:8081

GET    /api/v1/users           - Get all users
GET    /api/v1/users/{id}      - Get user by ID
POST   /api/v1/users           - Create user
PUT    /api/v1/users/{id}      - Update user
DELETE /api/v1/users/{id}      - Delete user
```

**Order Service API:**

```
Base URL: http://localhost:8082

GET    /api/v1/orders          - Get all orders
GET    /api/v1/orders/{id}     - Get order by ID
POST   /api/v1/orders          - Create order
GET    /api/v1/orders/user/{userId} - Get orders by user
```

---

### Inter-Service Communication Example

**Scenario:** Create an order

**Order Service needs user information:**

```java
// OrderService.java
@Service
public class OrderService {
    
    @Value("${user.service.url}")
    private String userServiceUrl; // http://localhost:8081
    
    private RestTemplate restTemplate = new RestTemplate();
    
    public Order createOrder(Long userId, Long productId, int quantity) {
        
        // 1. Call User Service to validate user exists
        String url = userServiceUrl + "/api/v1/users/" + userId;
        User user = restTemplate.getForObject(url, User.class);
        
        if (user == null) {
            throw new UserNotFoundException("User not found: " + userId);
        }
        
        // 2. Create order
        Order order = new Order();
        order.setUserId(userId);
        order.setProductId(productId);
        order.setQuantity(quantity);
        order.setStatus("PENDING");
        
        // 3. Save to database
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
    User Service
         ↓
    Returns user data
         ↓
    Order Service creates order
         ↓
    Returns order to client
```

---

### REST Best Practices for Microservices

**1. Keep APIs Simple**

```
✅ Good:
GET /users/{id}

❌ Too complex:
GET /users/{id}/orders/{orderId}/items/{itemId}/details
```

**2. Paginate Large Results**

```
GET /products?page=1&size=20

Response:
{
  "data": [...],
  "page": 1,
  "size": 20,
  "totalPages": 50
}
```

**3. Filter and Sort**

```
GET /products?category=electronics&sort=price&order=asc
```

**4. Use Clear Error Messages**

```json
{
  "error": "User not found",
  "message": "User with ID 123 does not exist",
  "timestamp": "2026-02-02T10:30:00Z",
  "status": 404
}
```

**5. Document Your APIs**

Use Swagger/OpenAPI:
```
http://localhost:8081/swagger-ui.html
```

Shows all endpoints, parameters, responses.

---

## Part 5: Inter-Service Communication Patterns (30 minutes)

### Communication Patterns Overview

Microservices need to communicate. Two main patterns:

1. **Synchronous Communication** (Request-Response)
2. **Asynchronous Communication** (Event-Driven)

---

### Pattern 1: Synchronous Communication

**Request-Response:** Service waits for response.

**Example: REST/HTTP**

```
Order Service                     User Service
     │                                 │
     ├─ GET /users/123 ───────────────>│
     │                                 │
     │<─────── User data ──────────────┤
     │                                 │
```

**Characteristics:**
- Service waits for response
- Immediate result
- Simple to understand
- Tight coupling

**Use when:**
- ✅ Need immediate response
- ✅ Simple data retrieval
- ✅ Few services involved

**Technologies:**
- REST/HTTP
- gRPC (faster than REST)
- GraphQL

---

### Synchronous Communication Example

```java
// Order Service calls User Service
@Service
public class OrderService {
    
    private RestTemplate restTemplate;
    
    public Order createOrder(CreateOrderRequest request) {
        
        // Synchronous call - waits for response
        User user = restTemplate.getForObject(
            "http://user-service:8081/api/v1/users/" + request.getUserId(),
            User.class
        );
        
        if (user == null) {
            throw new Exception("User not found");
        }
        
        // Create order
        Order order = new Order();
        order.setUserId(user.getId());
        order.setUserEmail(user.getEmail());
        
        return orderRepository.save(order);
    }
}
```

**Pros:**
- ✅ Simple to implement
- ✅ Easy to debug
- ✅ Immediate feedback

**Cons:**
- ❌ If User Service is down, Order Service fails
- ❌ Slower (network latency)
- ❌ Tight coupling

---

### Pattern 2: Asynchronous Communication

**Event-Driven:** Service publishes events, doesn't wait.

**Example: Message Queue**

```
Order Service                     Message Queue                User Service
     │                                 │                           │
     ├─ Publish "OrderCreated" ───────>│                           │
     │   event                          │                           │
     │                                  ├─────────────────────────>│
     │                                  │  Consume event           │
     │                                  │                           │
     │<─ Continue processing            │                           │
     │                                  │                           │
```

**Characteristics:**
- Service doesn't wait
- Decoupled services
- Better fault tolerance
- More complex

**Use when:**
- ✅ Don't need immediate response
- ✅ Long-running operations
- ✅ Multiple services interested in event

**Technologies:**
- RabbitMQ
- Apache Kafka
- AWS SQS
- Azure Service Bus

---

### Asynchronous Communication Example

```java
// Order Service publishes event
@Service
public class OrderService {
    
    private MessagePublisher messagePublisher;
    
    public Order createOrder(CreateOrderRequest request) {
        
        // 1. Create order
        Order order = new Order();
        order.setUserId(request.getUserId());
        order.setStatus("PENDING");
        orderRepository.save(order);
        
        // 2. Publish event (doesn't wait!)
        OrderCreatedEvent event = new OrderCreatedEvent(
            order.getId(),
            order.getUserId(),
            LocalDateTime.now()
        );
        messagePublisher.publish("order.created", event);
        
        // 3. Return immediately
        return order;
    }
}

// User Service consumes event
@Component
public class OrderEventListener {
    
    @EventListener("order.created")
    public void handleOrderCreated(OrderCreatedEvent event) {
        // Update user statistics
        userService.incrementOrderCount(event.getUserId());
        
        // Send email notification
        emailService.sendOrderConfirmation(event.getUserId());
    }
}
```

**Pros:**
- ✅ Services decoupled
- ✅ Better fault tolerance
- ✅ Scalable

**Cons:**
- ❌ More complex
- ❌ Eventual consistency
- ❌ Harder to debug

---

### Synchronous vs Asynchronous

| Aspect | Synchronous | Asynchronous |
|--------|-------------|--------------|
| Coupling | Tight | Loose |
| Response | Immediate | Eventual |
| Complexity | Simple | Complex |
| Fault Tolerance | Lower | Higher |
| Use Case | Read data | Notify events |

**Example:**

```
Get User Data:
✅ Synchronous (need data now)

Order Placed:
✅ Asynchronous (notify multiple services)
   - Email service sends confirmation
   - Inventory service updates stock
   - Analytics service logs event
```

---

### Communication Challenges

**1. Network Failures**

```
Order Service → User Service
                    ↓
              Network timeout!
              
What to do?
- Retry with exponential backoff
- Circuit breaker pattern
- Fallback response
```

**2. Service Discovery**

```
Problem: Services run on different machines with different IPs

Order Service needs User Service:
❌ Hardcode: http://192.168.1.100:8081
❌ Changes when redeployed!

✅ Service Discovery:
   - Services register with registry
   - Query registry for service location
   - Dynamic discovery
```

**3. Load Balancing**

```
Multiple instances of User Service:

┌────────┐ ┌────────┐ ┌────────┐
│User    │ │User    │ │User    │
│Service │ │Service │ │Service │
│:8081   │ │:8082   │ │:8083   │
└────────┘ └────────┘ └────────┘

Which one to call?
- Round robin
- Least connections
- Random
```

**4. Data Consistency**

```
Order created in Order Service
User statistics in User Service

How to keep in sync?
- Eventual consistency (async events)
- Saga pattern (distributed transactions)
- CQRS (separate read/write models)
```

---

### Best Practices for Inter-Service Communication

**1. Use Timeouts**

```java
RestTemplate restTemplate = new RestTemplate();

// Set timeout
SimpleClientHttpRequestFactory factory = new SimpleClientHttpRequestFactory();
factory.setConnectTimeout(3000); // 3 seconds
factory.setReadTimeout(3000);    // 3 seconds

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

**3. Use Circuit Breaker**

```
Closed State → Service works normally
               ↓
         Multiple failures
               ↓
Open State   → Stop calling service (fail fast)
               ↓
         Wait timeout
               ↓
Half-Open    → Try one request
               ↓
         Success → Back to Closed
         Failure → Back to Open
```

**4. Minimize Inter-Service Calls**

```
❌ Chatty:
For each order item:
  - Call Product Service
  - Call Inventory Service
  - Call Pricing Service
(3 calls per item, 10 items = 30 calls!)

✅ Batch:
  - Call Product Service with all IDs (1 call)
  - Call Inventory Service with all IDs (1 call)
  - Call Pricing Service with all IDs (1 call)
(3 calls total)
```

---

## Part 6: Configuration Management (25 minutes)

### The Configuration Challenge

Each microservice needs configuration:
- Database URLs
- API keys
- Service URLs
- Feature flags
- Environment-specific settings

**Problem:**
- 20 microservices = 20 configuration files
- Different values per environment (dev, staging, prod)
- How to update without redeploying?

---

### Configuration Approaches

**1. Application Properties (Simple)**

```properties
# application.properties
server.port=8081
spring.datasource.url=jdbc:postgresql://localhost:5432/users
user.service.url=http://localhost:8081
```

**Pros:**
- ✅ Simple
- ✅ Built into Spring Boot

**Cons:**
- ❌ Hardcoded in JAR
- ❌ Must rebuild/redeploy to change
- ❌ Different values per environment

---

**2. Environment Variables (Better)**

```properties
# application.properties
server.port=${SERVER_PORT:8081}
spring.datasource.url=${DATABASE_URL}
user.service.url=${USER_SERVICE_URL}
```

**Set via Docker Compose:**

```yaml
services:
  order-service:
    environment:
      SERVER_PORT: 8082
      DATABASE_URL: jdbc:postgresql://db:5432/orders
      USER_SERVICE_URL: http://user-service:8081
```

**Pros:**
- ✅ No rebuild needed
- ✅ Different per environment
- ✅ Docker-friendly

**Cons:**
- ❌ Must restart service to change
- ❌ Hard to manage many variables

---

**3. External Configuration Files**

**Mount configuration files:**

```yaml
# docker-compose.yml
services:
  order-service:
    volumes:
      - ./config/application-prod.properties:/app/config/application.properties
```

**Pros:**
- ✅ Easy to update
- ✅ Version controlled

**Cons:**
- ❌ Still need restart
- ❌ File management overhead

---

**4. Configuration Server (Advanced)**

**Spring Cloud Config Server:**

```
┌────────────────┐
│  Config Server │ ← Centralized configuration
│   :8888        │
└────────┬───────┘
         │
    ┌────┴─────┬──────────┬─────────┐
    │          │          │         │
┌───▼───┐  ┌──▼───┐  ┌───▼───┐  ┌──▼───┐
│User   │  │Order │  │Product│  │Payment│
│Service│  │Service│  │Service│  │Service│
└───────┘  └──────┘  └───────┘  └──────┘

All services fetch config from Config Server
```

**Features:**
- Centralized configuration
- Dynamic refresh (no restart!)
- Environment-specific configs
- Version history

**Cons:**
- Additional infrastructure
- More complexity
- Single point of failure (if not HA)

---

### Configuration Best Practices

**1. Use Profiles**

```properties
# application.properties (common)
server.port=8080

# application-dev.properties
spring.datasource.url=jdbc:postgresql://localhost:5432/dev_db

# application-prod.properties
spring.datasource.url=jdbc:postgresql://prod-db:5432/prod_db
```

**Activate profile:**
```bash
java -jar app.jar --spring.profiles.active=prod
```

**2. Use Default Values**

```properties
server.port=${SERVER_PORT:8080}
# If SERVER_PORT not set, use 8080
```

**3. Externalize Secrets**

❌ **Never:**
```properties
database.password=mySecretPassword123
```

✅ **Do:**
```properties
database.password=${DATABASE_PASSWORD}
```

**Set via:**
- Environment variables
- Secrets management (HashiCorp Vault, AWS Secrets Manager)
- Kubernetes Secrets

**4. Validate Configuration**

```java
@Configuration
@Validated
public class ServiceConfig {
    
    @NotNull
    @Value("${user.service.url}")
    private String userServiceUrl;
    
    @Min(1000)
    @Max(9999)
    @Value("${server.port}")
    private int serverPort;
}
```

Application fails to start if config invalid!

**5. Document Configuration**

```properties
# Server Configuration
# Port on which service listens (default: 8080)
server.port=${SERVER_PORT:8080}

# Database Configuration
# PostgreSQL database URL
# Format: jdbc:postgresql://host:port/database
spring.datasource.url=${DATABASE_URL}
```

---

### Configuration in Docker Compose

**Example: Order Service Configuration**

```yaml
version: "3.9"

services:
  order-service:
    build: ./order-service
    ports:
      - "8082:8080"
    environment:
      # Server config
      SERVER_PORT: 8080
      
      # Database config
      DATABASE_URL: jdbc:postgresql://order-db:5432/orders
      DATABASE_USERNAME: orderuser
      DATABASE_PASSWORD: orderpass
      
      # Other service URLs
      USER_SERVICE_URL: http://user-service:8081
      PRODUCT_SERVICE_URL: http://product-service:8083
      
      # Feature flags
      ENABLE_CACHING: "true"
      ENABLE_METRICS: "true"
    
    depends_on:
      - order-db
      - user-service
      - product-service
  
  order-db:
    image: postgres:15
    environment:
      POSTGRES_DB: orders
      POSTGRES_USER: orderuser
      POSTGRES_PASSWORD: orderpass
```

**Benefits:**
- ✅ Clear configuration
- ✅ Easy to modify
- ✅ Version controlled
- ✅ Environment-specific

---

## Part 7: Microservices Challenges (10 minutes)

### Distributed System Challenges

**1. Distributed Tracing**

```
User Request
    ↓
API Gateway
    ↓
Order Service → User Service
    ↓
Payment Service → Bank API

Where did the request fail?
Which service is slow?

Solution: Distributed tracing (Zipkin, Jaeger)
- Traces request across services
- Shows timing for each hop
- Identifies bottlenecks
```

**2. Monitoring and Logging**

```
Monolith:
- One log file
- Easy to search

Microservices:
- 20+ log files
- Scattered across services
- Hard to correlate

Solution: Centralized logging (ELK Stack, Splunk)
- Aggregate logs from all services
- Searchable
- Correlated by request ID
```

**3. Data Consistency**

```
Order created → Payment processed → Inventory updated

What if Payment succeeds but Inventory update fails?
- Inconsistent state!
- Order shows paid, but items not reserved

Solution: Saga pattern
- Distributed transaction management
- Compensating transactions on failure
```

**4. Testing Complexity**

```
Monolith:
- Test one application
- Integration tests straightforward

Microservices:
- Test 20 services
- Test service interactions
- Mock dependencies
- Contract testing needed
```

---

### When NOT to Use Microservices

**Don't use microservices if:**

❌ You have a small team (< 10 developers)
❌ Your application is small/medium sized
❌ You're building an MVP
❌ You lack DevOps expertise
❌ You don't need independent scaling
❌ You can't afford the operational overhead

**Start with a monolith:**
1. Build monolith
2. Understand domain
3. Identify boundaries
4. Extract services when needed

**"You can't build microservices if you can't build a monolith."**

---

## Summary

### Key Concepts

**1. Monolith vs Microservices**
- Monolith = One application, simple but limited scaling
- Microservices = Many services, complex but flexible

**2. When to Use Each**
- Monolith: Small teams, simple apps, MVPs
- Microservices: Large teams, complex apps, high scale

**3. Decomposition Strategies**
- By business capability
- By subdomain (DDD)
- By use case
- By resource

**4. Communication**
- Synchronous (REST) for immediate responses
- Asynchronous (Events) for decoupling

**5. Configuration**
- Environment variables
- External config files
- Configuration servers (advanced)

---

### Advantages Recap

**Microservices:**
- ✅ Independent scaling
- ✅ Technology diversity
- ✅ Fault isolation
- ✅ Fast deployment
- ✅ Team autonomy

**Challenges:**
- ❌ Complexity
- ❌ Network latency
- ❌ Data consistency
- ❌ Operations overhead
- ❌ Testing complexity

---

### Real-World Examples

**Companies Using Microservices:**
- Netflix: 100+ microservices
- Amazon: 1000+ microservices
- Uber: 2000+ microservices
- Spotify: Hundreds of microservices

**Success Factors:**
- Large engineering teams
- DevOps culture
- Strong tooling
- Gradual migration from monolith

---

### Preparing for Next Lesson

**In Lesson 4.16, you will:**
1. Build 2 microservices from scratch
   - User Service (manages users)
   - Order Service (manages orders, calls User Service)
2. Implement REST communication
3. Deploy with Docker Compose
4. Test the entire system

**What you need:**
- Understanding of concepts from today
- Your Docker knowledge (Lesson 4.6)
- Spring Boot basics
- Ready to code!

---

## Additional Resources

### Documentation
- [Microservices.io](https://microservices.io/) - Patterns and best practices
- [Martin Fowler on Microservices](https://martinfowler.com/articles/microservices.html)
- [Spring Cloud Documentation](https://spring.io/projects/spring-cloud)

### Books
- "Building Microservices" by Sam Newman
- "Microservices Patterns" by Chris Richardson
- "Domain-Driven Design" by Eric Evans

### Videos
- [What are Microservices?](https://www.youtube.com/results?search_query=microservices+explained)
- [Microservices vs Monolith](https://www.youtube.com/results?search_query=microservices+vs+monolith)
- [Netflix Microservices Architecture](https://www.youtube.com/results?search_query=netflix+microservices)

### Reference Diagrams
- [Microservices Architecture](https://microservices.io/i/Microservice_Architecture.png)
- [Monolith vs Microservices](https://www.redhat.com/rhdc/managed-files/monolithic-vs-microservices.png)
- [Communication Patterns](https://microservices.io/patterns/communication-style/messaging.html)

---

**Congratulations!** You now understand microservices architecture fundamentals! In the next lesson, you'll build a real microservices system! 

**Next Lesson:** Lesson 4.16 - Microservices Architecture Part 2 (Practical Demo)
