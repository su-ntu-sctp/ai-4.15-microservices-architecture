# Self Studies: Microservices Architecture - Part 1

## Overview

This lesson introduces microservices — a major architectural shift from the monolithic applications you have built so far. The self-study materials below will help you arrive with a clear understanding of why microservices exist, how services communicate, and how to think about decomposing an application. This is important because Lesson 4.16 immediately puts these concepts into practice by building a real microservices system.

**Estimated Prep Time:** 60–80 minutes

---

## Task 1: Microservices vs Monolith

This video gives you a clear, visual comparison of monolithic and microservices architectures — including when to use each and real-world examples. Watch it before reading the lesson so the concepts feel familiar during class.

**Watch:** Microservices vs Monolith — Which Should You Choose?
🎬 https://www.youtube.com/watch?v=6-Wu178sOEE

**Then read:** Lesson 4.15 — Parts 1 and 2

**Guiding Questions:**
- What are the main reasons a company would move from a monolith to microservices?
- What are the hidden costs of microservices that are easy to overlook?
- Why is a monolith often the right choice for small teams and MVPs?
- What does "independent deployment" mean in practice?

---

## Task 2: Spring Boot Microservices in Practice

This video shows how microservices are built in Spring Boot — including service decomposition, inter-service communication with RestTemplate, and configuration. This directly prepares you for the hands-on implementation in Lesson 4.16.

**Watch:** Spring Boot Microservices Tutorial
🎬 https://www.youtube.com/watch?v=rp0H85kWZf4&t=26s

**Then read:** Lesson 4.15 — Parts 3, 4, and 5

**Guiding Questions:**
- How does RestTemplate allow one Spring Boot service to call another?
- Why does each microservice need its own database?
- What is the difference between synchronous and asynchronous inter-service communication?
- How does environment variable configuration differ from hardcoding values in `application.properties`?

---

## Task 3: Think Through Decomposition

This is a read-only task to prepare you for the in-class activity.

**Read:** Lesson 4.15 — Part 3 (Service Decomposition) and Part 6 (Configuration Management)

**Guiding Questions:**
- What are the four main decomposition strategies and when would you use each?
- Why should microservices never share a database?
- What is Conway's Law and how does it affect microservices design?
- What does the `${VARIABLE:default}` syntax in `application.properties` do?

> 💡 **Pre-class prep:** Think about how you would decompose the `simple-crm` project into microservices. What services would you create? What data would each own? You may be asked to discuss this in class.

---

## Active Engagement Strategies

- As you watch the first video, make a list of the pros and cons of microservices in your own words — not copied from slides
- After watching the second video, sketch a simple diagram of two services communicating via REST — show the request and response flow
- Before class, revisit your `simple-crm` project and identify which controllers, services, and repositories would belong to different microservices if you were to split it up

---

## Additional Reading Material

- [Microservices.io — Patterns and Best Practices](https://microservices.io/)
- [Martin Fowler on Microservices](https://martinfowler.com/articles/microservices.html)
- [Spring RestTemplate Guide — Baeldung](https://www.baeldung.com/rest-template)
- [12-Factor App — Configuration Best Practices](https://12factor.net/config)