# Layers Overview

In Clean Architecture, the application is divided into multiple **layers**, where each layer has a specific responsibility.

The purpose of layers is to organize the application so that different responsibilities are separated and dependencies remain controlled.

Before studying each layer individually, it is important to understand **what a layer is, why layers exist, and how layers work together**.

---

## 01 — What Is a Layer?

A layer is a logical boundary that groups code with a similar responsibility.

For example:

```text
Domain
Application
Infrastructure
Presentation
```

Each layer answers a different question:

```text
Domain
→ What are the business rules?

Application
→ What does the application do?

Infrastructure
→ How are technical operations implemented?

Presentation
→ How does the outside world communicate with the application?
```

The goal is to prevent all of these responsibilities from being mixed together.

---

## 02 — Why Do We Use Layers?

Without clear boundaries, an application can easily become tightly coupled.

For example:

```text
Controller
    ↓
Business Logic
    ↓
EF Core
    ↓
SQL Server
```

If everything is mixed together, changing one part can affect many other parts.

Layers create boundaries:

```text
┌─────────────────────────────┐
│       Presentation          │
├─────────────────────────────┤
│       Application           │
├─────────────────────────────┤
│          Domain             │
├─────────────────────────────┤
│       Infrastructure        │
└─────────────────────────────┘
```

Each part has a clearer responsibility.

---

## 03 — A Layer Has a Responsibility

A layer should have a clear reason to exist.

For example:

```text
Domain
→ Business concepts and business rules

Application
→ Application use cases

Infrastructure
→ Technical implementations

Presentation
→ Communication with external clients
```

The important idea is:

> **A layer should contain responsibilities that belong together.**

We should avoid putting unrelated responsibilities into the same layer.

---

## 04 — Layers Are About Boundaries

The most important purpose of layers is not creating folders or projects.

It is creating **boundaries**.

For example:

```text
Business Rules
      │
      │ boundary
      ▼
Application Logic
      │
      │ boundary
      ▼
Technical Details
      │
      │ boundary
      ▼
External Systems
```

These boundaries help control how parts of the application interact.

---

## 05 — Communication Between Layers

Layers communicate with each other through defined boundaries.

A simplified example:

```text
Request
   ↓
Presentation
   ↓
Application
   ↓
Domain
   ↓
Infrastructure
   ↓
Database
```

But this does **not** mean every layer should directly depend on every other layer.

The important question is:

> **Who is allowed to depend on whom?**

That is one of the central ideas of Clean Architecture.

---

## 06 — Dependency Direction

Clean Architecture is primarily concerned with controlling dependency direction.

A simplified view is:

```text
Outer Layers
     ↓
Inner Layers
```

For example:

```text
Presentation
     ↓
Application
     ↓
Domain
```

Infrastructure can provide implementations for abstractions required by the inner layers.

The key idea is:

```text
Dependencies → Toward the Core
```

The core should not become dependent on external technical details.

---

## 07 — Layers vs Responsibilities

Do not think of layers simply as:

```text
Project 1
Project 2
Project 3
Project 4
```

Instead, think:

```text
Responsibility
      ↓
Boundary
      ↓
Dependency
      ↓
Implementation
```

For example:

```text
Business rule
      ↓
Domain

Application operation
      ↓
Application

Database implementation
      ↓
Infrastructure

HTTP communication
      ↓
Presentation
```

The layer exists because the responsibility exists.

---

## 08 — Typical Clean Architecture Layers

A common representation is:

```text
┌──────────────────────────────────┐
│       Presentation / API         │
├──────────────────────────────────┤
│          Application             │
├──────────────────────────────────┤
│             Domain               │
├──────────────────────────────────┤
│          Infrastructure          │
└──────────────────────────────────┘
```

Another common representation is:

```text
┌─────────────────────────────────────┐
│        Frameworks & Drivers         │
├─────────────────────────────────────┤
│        Interface Adapters           │
├─────────────────────────────────────┤
│        Application / Use Cases      │
├─────────────────────────────────────┤
│             Entities               │
└─────────────────────────────────────┘
```

The names can vary between projects.

The important thing is understanding the **responsibilities and dependency boundaries**, not memorizing project names.

---

## 09 — What Belongs in Each Layer?

At a high level:

### Domain

Contains the application's core business concepts and rules.

```text
Entities
Value Objects
Domain Rules
Domain-specific behavior
```

---

### Application

Contains application-specific operations and use cases.

```text
Use Cases
Application Services
Interfaces / Abstractions
DTOs
Application-specific rules
```

---

### Infrastructure

Contains technical implementations.

```text
Database
EF Core
Repositories
External APIs
Email
File Storage
Authentication implementations
```

---

### Presentation

Handles communication with external clients.

```text
Controllers
Endpoints
HTTP Requests
HTTP Responses
API Models
Middleware
```

These are only high-level responsibilities.

Each layer will be studied separately later.

---

## 10 — A Simple Example

Imagine an e-commerce application.

A customer wants to create an order.

Different responsibilities exist:

```text
HTTP Request
      ↓
Presentation
```

The application needs to execute the operation:

```text
Create Order
      ↓
Application
```

The order itself follows business rules:

```text
Order
OrderItem
Total
Discount
      ↓
Domain
```

The order eventually needs to be persisted:

```text
Database
      ↓
Infrastructure
```

So conceptually:

```text
HTTP
 ↓
Presentation
 ↓
Application
 ↓
Domain
 ↓
Infrastructure
 ↓
Database
```

Each part has a different responsibility.

---

## 11 — A Layer Should Not Know Everything

A common mistake is allowing a layer to know about every other layer.

For example:

```text
Domain
 ↓
EF Core
 ↓
SQL Server
```

This would make the business core dependent on technical details.

Instead, the goal is to keep the core focused on business concerns.

```text
Domain
  │
  │
  └── Business Rules

Infrastructure
  │
  ├── EF Core
  ├── SQL Server
  └── External Services
```

Technical details remain outside the core.

---

## 12 — Layers Do Not Mean Every Request Must Pass Through Everything

Layers are boundaries, not necessarily a mandatory pipeline.

For example, you should not assume:

```text
Controller
 ↓
Service
 ↓
Manager
 ↓
Handler
 ↓
Repository
 ↓
Database
```

for every operation.

Instead, the architecture should use the layers according to their responsibilities.

The important question is:

> **Which responsibility belongs where?**

Not:

> **How many layers can this request pass through?**

---

## 13 — Layers and Dependency Inversion

One of the important mechanisms behind Clean Architecture is **Dependency Inversion**.

For example, the Application layer may need persistence:

```text
Application
      ↓
IProductRepository
```

Infrastructure can provide the implementation:

```text
Application
      ↓
IProductRepository
      ↑
ProductRepository
      ↓
EF Core
```

The Application layer depends on an abstraction rather than the concrete infrastructure implementation.

This allows the boundary between layers to remain stable.

---

## 14 — Layers and Projects

A layer can be represented by a project:

```text
src/
├── Domain/
├── Application/
├── Infrastructure/
└── API/
```

But a layer and a project are not exactly the same concept.

A **layer is an architectural boundary**.

A **project is a code organization/build boundary**.

They often map to each other, but the concepts are different.

For example:

```text
Architecture
     ↓
Layers
     ↓
Projects
     ↓
Folders
     ↓
Classes
```

Projects and folders should support the architecture rather than define it.

---

## 15 — Layers and Separation of Concerns

Layers help achieve **Separation of Concerns**.

Instead of having one large area responsible for everything:

```text
Everything
├── Business Logic
├── Database
├── HTTP
├── Authentication
├── External APIs
└── Validation
```

we separate responsibilities:

```text
Domain
→ Business rules

Application
→ Use cases

Infrastructure
→ Technical details

Presentation
→ External communication
```

This makes the system easier to understand and change.

---

## 16 — Mental Model

Think of the application as a protected core surrounded by technical details.

```text
┌──────────────────────────────────────┐
│          Presentation                │
│                                      │
│   ┌──────────────────────────────┐   │
│   │        Infrastructure        │   │
│   │                              │   │
│   │   ┌──────────────────────┐   │   │
│   │   │     Application      │   │   │
│   │   │                      │   │   │
│   │   │   ┌──────────────┐   │   │   │
│   │   │   │    Domain    │   │   │   │
│   │   │   │              │   │   │   │
│   │   │   │ Business     │   │   │   │
│   │   │   │ Rules         │   │   │   │
│   │   │   └──────────────┘   │   │   │
│   │   └──────────────────────┘   │   │
│   └──────────────────────────────┘   │
└──────────────────────────────────────┘
```

The exact visual arrangement can differ depending on the project.

The important concept is:

```text
Core
 ↓
Business Rules

Outside
 ↓
Application Operations
 ↓
Technical Implementations
 ↓
External Systems
```

---
