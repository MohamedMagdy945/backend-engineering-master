# Combining Architectural Styles

In real-world software engineering, systems almost never use a single architectural style in pure textbook form.

A common beginner mistake is viewing architectural styles as mutually exclusive choices:

```text
Mistake:
"Should we use Clean Architecture OR Vertical Slices OR a Modular Monolith?"
```

The professional approach recognizes that these styles operate on **different, orthogonal dimensions** and can be combined to solve distinct engineering problems:

```text
Reality:
"We use a Modular Monolith for deployment boundaries,
 Vertical Slices for feature organization,
 and Clean Architecture principles for complex business rules."
```

Understanding how to compose architectural styles creates systems that are both simple to develop and capable of scaling.

---

## 01 — The Three Orthogonal Architectural Dimensions

Architectural styles operate at three distinct scopes in a system:

```text
┌────────────────────────────────────────────────────────────────────────┐
│ 1. Macro Architecture: Deployment & Distribution                       │
│    • How is code deployed and run?                                     │
│    • Monolith │ Modular Monolith │ Microservices │ Serverless          │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ contains
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 2. Meso Architecture: Organization & Modularity                        │
│    • How are files and use cases grouped inside the deployable unit?   │
│    • Technical Layers (Controllers/Services/Repos) │ Vertical Slices   │
└───────────────────────────────────┬────────────────────────────────────┘
                                    │ contains
                                    ▼
┌────────────────────────────────────────────────────────────────────────┐
│ 3. Micro Architecture: Dependency & Domain Modeling                    │
│    • How are business rules and external dependencies structured?      │
│    • Transaction Script │ Active Record │ Clean / Hexagonal / Onion    │
└────────────────────────────────────────────────────────────────────────┘
```

Because these three dimensions address completely different concerns, you do not choose between them; you select **one style for each dimension**.

---

## 02 — Blueprint 1: Modular Monolith + Vertical Slices + Clean Core

This is the most productive and resilient architecture pattern for modern enterprise backends. It scales smoothly from small teams to dozens of developers without distributed systems overhead.

### How the Styles Combine

```text
┌───────────────────────────────────────────────────────────────────────────────┐
│                           Modular Monolith (Macro)                            │
│                                                                               │
│  ┌─────────────────────────┐  In-Process  ┌────────────────────────────────┐  │
│  │     Catalog Module      │ ───────────► │        Ordering Module         │  │
│  └─────────────────────────┘    Contracts └───────────────┬────────────────┘  │
│                                                           │                   │
│                                   ┌───────────────────────┴────────────────┐  │
│                                   │       Vertical Slices (Meso)           │  │
│                                   │                                        │  │
│                                   │  ┌─────────────────┐ ┌───────────────┐ │  │
│                                   │  │ GetOrderSummary │ │  PlaceOrder   │ │  │
│                                   │  │ (Simple Read)   │ │ (Complex Write│ │  │
│                                   │  └────────┬────────┘ └───────┬───────┘ │  │
│                                   └───────────┼──────────────────┼─────────┘  │
│                                               │                  │            │
│                  ┌────────────────────────────┘                  │            │
│                  ▼                                               ▼            │
│       Fast Pragmatic Read                            Clean Core (Micro)       │
│       • Direct SQL (Dapper / EF)                     • Domain Aggregate       │
│       • Zero domain hydration                        • Business Invariants    │
│       • Direct DTO return                            • Dependency Inversion   │
└───────────────────────────────────────────────────────────────────────────────┘
```

### 1. Macro: Modular Monolith
The application is a single deployable artifact (one process, one database instance), but strict compilation boundaries divide the code into isolated business modules (`Catalog`, `Ordering`, `Billing`).

### 2. Meso: Vertical Slices
Inside each module, code is organized by use cases (`Features/Orders/PlaceOrder`, `Features/Orders/GetOrderById`) rather than global technical folders (`Services`, `Repositories`). Each slice owns its endpoint, request, response, and handler.

### 3. Micro: Selective Clean Core
Not every feature needs complex domain modeling:
- **Simple reads (e.g., `GetOrderSummary`):** Bypass domain abstractions completely. Query the database directly via Dapper or EF Core projection to return the DTO.
- **Complex writes (e.g., `PlaceOrder`):** Encapsulate business invariants inside a rich Domain Entity/Aggregate using Clean Architecture dependency inversion rules.

### Concrete Project Structure

```text
src/
├── Modules/
│   ├── Ordering/
│   │   ├── Domain/                      <── Clean Core (Rich rules)
│   │   │   ├── Order.cs
│   │   │   ├── OrderStatus.cs
│   │   │   └── Events/OrderPlacedEvent.cs
│   │   ├── Features/                    <── Vertical Slices
│   │   │   ├── PlaceOrder/
│   │   │   │   ├── PlaceOrderCommand.cs
│   │   │   │   ├── PlaceOrderHandler.cs (uses Domain.Order)
│   │   │   │   └── PlaceOrderEndpoint.cs
│   │   │   └── GetOrderById/
│   │   │       ├── GetOrderByIdQuery.cs
│   │   │       ├── GetOrderByIdHandler.cs (uses Dapper direct SQL)
│   │   │       └── GetOrderByIdEndpoint.cs
│   │   ├── Infrastructure/
│   │   │   ├── OrderingDbContext.cs
│   │   │   └── OrderingRepository.cs
│   │   └── PublicApi/                   <── Public Module Contract
│   │       ├── IOrderingModule.cs
│   │       └── OrderSummaryDto.cs
│   │
│   └── Catalog/                         <── Separate Module
│       └── ...
```

---

## 03 — Blueprint 2: Microservices + Hexagonal Architecture (Ports and Adapters)

When services are distributed across independent networks, **Hexagonal Architecture** becomes the ideal micro-architecture inside each microservice.

```text
                              Order Microservice Boundary
        ┌─────────────────────────────────────────────────────────────────────┐
        │                                                                     │
        │    Driving Adapters                                                 │
        │   ┌────────────────┐                                                │
HTTP ──────►│ REST Controller│────┐                                           │
            └────────────────┘    │ Inbound Ports                             │
                                  ▼                                           │
            ┌────────────────┐   ┌─────────────────┐    Driven Ports          │
Message ───►│ Rabbit Consumer│──►│  Domain Core    │───► IOrderRepository     │
(Events)    └────────────────┘   │  (Business      │───► IPaymentGateway      │
                                 │   Logic Only)   │                          │
                                 └─────────────────┘        │          │      │
                                                            │          │      │
                                            Driven Adapters ▼          ▼      │
                                            ┌──────────────────┐ ┌──────────┐ │
                                            │ Postgres (EF/SQL)│ │ StripeAPI│ │
                                            └──────────────────┘ └──────────┘ │
        └─────────────────────────────────────────────────────────────────────┘
```

### Why this combination works:
1. **Network independence:** The domain core knows nothing about HTTP, gRPC, or message queues. Swapping an HTTP endpoint for an event subscriber does not affect business logic.
2. **Infrastructure mockability:** The database and external payment APIs sit behind driven ports (interfaces). Unit testing the domain core requires zero external infrastructure.
3. **Isolation:** The distributed network concerns (retries, timeouts, circuit breakers) live strictly inside the adapters, keeping business logic clean.

---

## 04 — Blueprint 3: Asymmetric CQRS

A common failure in Clean Architecture is forcing every query through domain aggregates and repositories.

**Command-Query Responsibility Segregation (CQRS)** solves this by using two different architectural styles within the same application:

```text
                    Incoming Request
                           │
            ┌──────────────┴──────────────┐
            ▼                             ▼
   Command (Write State)          Query (Read Data)
   ─────────────────────          ─────────────────
   Style: Clean / DDD             Style: Vertical Slice / Direct SQL
   • Hydrate Order Aggregate      • Direct SELECT with Dapper/EF
   • Validate state invariants    • Read-only, no tracking
   • Raise Domain Events          • No domain entities allocated
   • Commit Transaction           • Straight to UI DTO
            │                             │
            ▼                             ▼
       Write Store                   Read Store / View
```

### The Rule for CQRS Combination:
- **Commands require safety and rules:** Use Clean/Onion models to guarantee that no business invariant is violated.
- **Queries require speed and projection:** Use simple vertical slices that bypass domain layers and query database views directly.

---

## 05 — The "Frankenstein Architecture" Anti-Pattern

Combining styles requires strict discipline. When styles are combined arbitrarily without clear boundaries, you create a **Frankenstein Architecture**:

```text
The Frankenstein Anti-Pattern:
• Slice A calls Slice B's internal repository directly.
• Module X reaches into Module Y's database tables.
• Some features use 5 layers of interfaces for a single SELECT query.
• Other features put 2,000 lines of raw SQL inside a controller.
• Circular dependencies form between layers and slices.
```

### The Golden Rule of Combined Architectures

> **Uniformity within a boundary, flexibility across boundaries.**

1. **Inside a single feature slice:** Choose either a direct query or a domain command. Do not invent a third ad-hoc pattern.
2. **Inside a single module:** Modularity rules apply equally to all slices within that module.
3. **Across modules:** Modules may only communicate through explicitly exposed contracts (`PublicApi`) or asynchronous domain events. Never share internal implementation classes.

```text
Allowed Cross-Module Flow:
Ordering Slice ──► IInventoryModule (Public Contract) ──► Inventory Module

Forbidden Cross-Module Flow:
Ordering Slice ──► InventoryDbContext.Inventories.Update(...) (Direct Leak!)
```

---

## 06 — Architecture Combination Decision Matrix

| Project Profile | Recommended Deployment (Macro) | Recommended Organization (Meso) | Recommended Core (Micro) | Why this combination? |
|---|---|---|---|---|
| **Early Stage Startup / Small Team** | Monolith | Vertical Slices | Pragmatic Transaction Script | Maximum delivery speed, minimum boilerplate, easy to navigate. |
| **Medium Business / 5–20 Engineers** | Modular Monolith | Vertical Slices | Clean Core on Writes, Direct SQL on Reads | Fast feature development, clean business boundaries, zero network latency. |
| **Large Enterprise / High Scale Subsystems** | Hybrid (Modular Monolith + 2-3 Microservices) | Vertical Slices | Hexagonal (Ports & Adapters) inside services | Allows high-load bottlenecks to scale independently while keeping the core business simple. |
| **Complex Domain Rules (Finance, Healthcare)** | Modular Monolith | Vertical Slices | Full DDD Aggregates & Event Sourcing | Complete audit trail and rock-solid state invariants without distributed systems pain. |

---

## 07 — Summary

- Architectural styles are **complementary dimensions**, not mutually exclusive options.
- The most balanced pattern for modern systems is:
  - **Modular Monolith** for zero-latency deployment boundaries.
  - **Vertical Slices** for clear feature development and low coupling.
  - **Clean / Hexagonal Core** for complex business invariants on write commands.
  - **Direct SQL / Thin Queries** for high-performance read queries.
- Combine styles intentionally at defined boundaries. Never allow cross-boundary shortcuts that degrade the system into accidental complexity.
