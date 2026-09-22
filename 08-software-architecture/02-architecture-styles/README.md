# 02 — Architecture Styles

**Architecture Style** is a general way of organizing a software system.

It defines things such as:

* How the system is divided
* How components communicate
* Where business logic lives
* How dependencies should flow
* How technical details are separated from business rules

Different architecture styles solve different problems.

---

# 1. Why Do We Need Architecture Styles?

As an application grows, putting everything together becomes difficult.

Without a clear structure:

```text
Controllers
Services
Database
Business Logic
External APIs
Authentication
Validation
```

can become tightly connected.

Architecture styles give us rules for organizing these parts.

For example:

```text
Layered Architecture
        ↓
Clean Architecture
        ↓
Vertical Slice Architecture
```

Each style approaches organization differently.

---

# 2. Architecture Style vs Architecture Pattern

These terms are sometimes used interchangeably, but they are not exactly the same.

### Architecture Style

A broader way of structuring the entire system.

Examples:

```text
Layered
Clean
Hexagonal
Onion
Vertical Slice
Microservices
```

### Architecture Pattern

A reusable solution to a specific architectural problem.

Examples:

```text
Repository
CQRS
Unit of Work
Mediator
Result Pattern
```

So:

```text
Architecture Style
        ↓
Defines overall organization

Architecture Pattern
        ↓
Solves a specific problem
```

---

# 3. Main Architecture Styles

Our roadmap covers:

## Layered Architecture

Organizes the system into horizontal layers.

```text
Presentation
      ↓
Business
      ↓
Data Access
      ↓
Database
```

---

## Clean Architecture

Organizes the system around business rules and dependency direction.

```text
Infrastructure
      ↓
Application
      ↓
Domain
```

The core should not depend directly on technical details.

---

## Hexagonal Architecture

Also called **Ports and Adapters**.

The application core communicates with the outside world through ports.

```text
       Database
          ↓
      Adapter
          ↓
       Port
          ↓
    Application
          ↑
       Port
          ↑
      Adapter
          ↑
       HTTP/API
```

---

## Onion Architecture

Organizes the application in concentric layers.

```text
Infrastructure
      ↓
Application
      ↓
Domain Services
      ↓
Domain Model
```

The domain is at the center.

---

## Vertical Slice Architecture

Organizes code around **features/use cases** instead of technical layers.

Instead of:

```text
Controllers
Services
Repositories
DTOs
```

you might have:

```text
Products
 ├── CreateProduct
 ├── UpdateProduct
 ├── GetProduct
 └── DeleteProduct

Orders
 ├── CreateOrder
 ├── CancelOrder
 └── GetOrder
```

Each feature contains what it needs.

---

## Modular Monolith vs Microservices

This is about how the application is deployed and divided into larger business modules.

### Modular Monolith

One application:

```text
┌───────────────────────────────┐
│          Application          │
│                               │
│ Orders │ Products │ Inventory │
└───────────────────────────────┘
```

### Microservices

Multiple independently running services:

```text
Orders Service
      │
Products Service
      │
Inventory Service
```

They communicate through APIs or messaging.

---

# 4. Why Are There Different Styles?

There is no architecture style that is automatically correct for every project.

A small CRUD application may not need the same structure as a large ERP system.

For example:

```text
Small App
   ↓
Simple Layered Architecture
```

while a larger system may benefit from:

```text
Clean Architecture
+
Vertical Slices
+
Modular Monolith
```

The important thing is understanding **why** you choose a style.

---

# 5. The Main Differences

| Style            | Main Idea                                           |
| ---------------- | --------------------------------------------------- |
| Layered          | Organize by technical layers                        |
| Clean            | Protect business rules through dependency direction |
| Hexagonal        | Isolate the core using ports and adapters           |
| Onion            | Put the domain at the center                        |
| Vertical Slice   | Organize around features/use cases                  |
| Modular Monolith | One deployable application with strong modules      |
| Microservices    | Multiple independently deployable services          |

These styles can also be combined.

For example, a system can use:

```text
Modular Monolith
      +
Clean Architecture
      +
Vertical Slices
```

They are not always mutually exclusive.

---

# 6. What You Should Focus On

Don't memorize diagrams.

For every architecture style, ask:

### 1. How is the code organized?

```text
Layers?
Features?
Modules?
```

### 2. Where is the business logic?

```text
Domain?
Application?
Services?
```

### 3. How do dependencies flow?

```text
Who depends on whom?
```

### 4. What problem does the style solve?

```text
Maintainability?
Coupling?
Scalability?
Testability?
Team organization?
```

### 5. What problems can it introduce?

Every architecture style has trade-offs.

---

# 7. Mental Model

Think of architecture styles as different ways of answering:

> **"How should I organize this software so it remains understandable and changeable as it grows?"**

```text
Architecture Style
        ↓
System Organization
        ↓
Dependency Rules
        ↓
Maintainability
```

The goal is not to use the most complicated architecture.

The goal is to use a structure that fits the problem.

---

# 8. Key Takeaways

* Architecture styles provide a way to organize a software system.
* Different styles solve different architectural problems.
* **Layered** focuses on horizontal technical layers.
* **Clean/Onion** focus heavily on dependency direction and protecting the domain.
* **Hexagonal** focuses on ports and adapters.
* **Vertical Slice** focuses on features/use cases.
* **Modular Monolith/Microservices** focus more on system/module boundaries and deployment.
* Styles can be combined.
* Architecture should solve real problems, not add complexity for its own sake.

---


Next:

**05 — Layered Architecture**
