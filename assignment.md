# Assignment (Optional)

## Brief

Analyze and understand microservices architecture concepts by decomposing a monolithic application and designing inter-service communication.

1. **Service Decomposition Exercise**
   - Consider a simple **Library Management System** monolithic application with the following features:
     - User registration and login
     - Book catalog browsing and searching
     - Book borrowing and returning
     - Fine calculation for late returns
     - Book recommendations based on user history
     - Email notifications for due dates
   - Decompose this monolith into microservices by:
     - Identifying at least 4-5 separate microservices
     - For each service, define:
       - Service name
       - Primary responsibility (what it manages)
       - Main API endpoints (at least 2-3 endpoints per service)
       - Which database/data it owns
     - Example format:
```
       Service: User Service
       Responsibility: Manages user accounts and authentication
       Endpoints:
         - POST /api/v1/users (Create user)
         - GET /api/v1/users/{id} (Get user details)
         - PUT /api/v1/users/{id} (Update user)
       Database: Users DB (stores user profiles, credentials)
```
   - Draw or describe the relationships between services (which services need to communicate)
   - Write a brief explanation (4-5 sentences) justifying your decomposition choices

2. **Communication Pattern Analysis**
   - Using your Library Management System microservices from Question 1:
   - Identify at least 3 scenarios where services need to communicate
   - For each scenario, decide whether to use synchronous or asynchronous communication
   - Example format:
```
     Scenario 1: Borrowing a book
     Services involved: Borrowing Service → Book Service
     Communication type: Synchronous (REST)
     Reason: Need immediate confirmation that book is available
     
     Scenario 2: Send due date reminder
     Services involved: Borrowing Service → Notification Service
     Communication type: Asynchronous (Event)
     Reason: Don't need to wait for email to be sent
```
   - Write a brief explanation (3-4 sentences) describing:
     - Why you chose synchronous vs asynchronous for each scenario
     - What would happen if you used the wrong communication pattern
     - Which pattern provides better fault tolerance for your use cases

## Submission (Optional)

- Submit the URL of the GitHub Repository that contains your work to NTU black board.
- Should you reference the work of your classmate(s) or online resources, give them credit by adding either the name of your classmate or URL.

## References
- Java: https://docs.oracle.com/javase/
- Spring Boot: https://docs.spring.io/spring-boot/docs/current/reference/html/
- PostgreSQL: https://www.postgresql.org/docs/
- OWASP: https://cheatsheetseries.owasp.org/