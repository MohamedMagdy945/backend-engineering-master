# Modules Overview

In software architecture, **Modules** represent self-contained, cohesive units of software that encapsulate a specific business capability, domain concept, or technical subsystem behind a well-defined boundary.

While **Layers** organize code horizontally by technical concern (e.g., Domain, Application, Infrastructure), **Modules** organize code vertically by business capability (e.g., Catalog, Orders, Billing, Identity).

> **A module answers the question: Which business capability or cohesive subsystem does this code belong to, and what are its boundaries?**

---

## 01 — What Is a Module?

A module is an architectural boundary that groups related business logic, data, and operations into an independent unit.

For example, an e-commerce platform can be divided into modules:

```text
┌────────────────────────────────────────────────────────┐
│                      Application                       │
│                                                        │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │   Catalog    │  │    Orders    │  │   Billing    │  │
│  │    Module    │  │    Module    │  │    Module    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────┐  │
│  │  Customers   │  │   Shipping   │  │Notification  │  │
│  │    Module    │  │    Module    │  │    Module    │  │
│  └──────────────┘  └──────────────┘  └──────────────┘  │
└────────────────────────────────────────────────────────┘
```

Each module has:

1. **A Clear Responsibility:** Focuses on one distinct business capability.
2. **A Public Contract:** A small, explicit set of interfaces and DTOs exposed to the rest of the system.
3. **Hidden Implementation Details:** Internal classes, entities, database tables, and algorithms hidden from outside callers.

---

## 02 — Why Do We Use Modules?

Without modules, as an application grows, code organized only by technical folders degenerates into the **Big Ball of Mud**:

```text
Controllers/
Services/
Repositories/
```

In an unmodularized system:

```text
   ProductsController      OrdersController      InvoicesController
           │                      │                      │
           ▼                      ▼                      ▼
    ProductService ◄──────► OrderService ◄──────► InvoiceService
           │                      │                      │
           ▼                      ▼                      ▼
   ProductRepository      OrderRepository       InvoiceRepository
           │                      │                      │
           └──────────────────────┼──────────────────────┘
                                  ▼
                         Monolithic Database
                     (Tables joined everywhere)
```

### The Problems of an Unmodularized System:

* **High Coupling:** Any service can inject, instantiate, or call any other service.
* **Large Blast Radius:** A change in order calculation unexpectedly breaks invoicing or product inventory.
* **Cognitive Overload:** Developers must understand the entire codebase to safely touch any single part.
* **Merge Conflicts:** Multiple teams edit the same shared services and database models simultaneously.

Modules create strict **vertical walls** that prevent this chaos.

---

## 03 — Layers vs. Modules: Horizontal vs. Vertical Decomposition

Software architecture uses two fundamental axes of decomposition:

```text
               MODULE: Catalog    MODULE: Orders     MODULE: Billing
              ┌─────────────────┬─────────────────┬─────────────────┐
Presentation  │ Catalog API     │ Orders API      │ Billing API     │  ▲
              ├─────────────────┼─────────────────┼─────────────────┤  │
Application   │ Catalog UseCase │ Orders UseCase  │ Billing UseCase │  │ LAYERS
              ├─────────────────┼─────────────────┼─────────────────┤  │ (Horizontal)
Domain        │ Product Entity  │ Order Entity    │ Invoice Entity  │  │
              ├─────────────────┼─────────────────┼─────────────────┤  │
Infrastructure│ Catalog DB Repo │ Orders DB Repo  │ Payment Gateway │  ▼
              └─────────────────┴─────────────────┴─────────────────┘
                      ◄────────────── MODULES ──────────────►
                                    (Vertical)
```

### Comparison:

| Dimension | Layers (Horizontal) | Modules (Vertical) |
| :--- | :--- | :--- |
| **Organization Axis** | By technical responsibility | By business capability / domain |
| **Examples** | Domain, Application, Infrastructure, Presentation | Catalog, Orders, Billing, Identity, Shipping |
| **Core Goal** | Protect business rules from technical details | Isolate different business features from each other |
| **Dependency Rule** | Outside layers depend on inner core layers | Modules depend only on public contracts of other modules |
| **Scope of Change** | Changing a DB driver affects Infrastructure | Changing order calculation affects only Orders |

---

## 04 — Anatomy of a Well-Designed Module

A well-designed module applies the principle of **Information Hiding** (formulated by David Parnas):

> **A module should hide its internal design decisions and expose only what external consumers strictly need.**

```text
┌───────────────────────────────────────────────────────────┐
│                       MODULE BOUNDARY                     │
│                                                           │
│   PUBLIC CONTRACT (Exposed to the rest of the system)    │
│   ├── Public Interfaces (e.g., IOrdersModuleApi)          │
│   ├── Public DTOs (e.g., OrderSummaryDto, CreateOrderDto) │
│   └── Public Integration Events (e.g., OrderPlacedEvent)  │
│                                                           │
│ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ │
│                                                           │
│   INTERNAL IMPLEMENTATION (Completely Hidden)             │
│   ├── Domain Entities & Value Objects (Order, LineItem)   │
│   ├── Domain Rules & Business Invariants                  │
│   ├── Internal Handlers & Services                        │
│   ├── Database Context & Migrations                       │
│   └── Database Tables / Storage Schema                    │
│                                                           │
└───────────────────────────────────────────────────────────┘
```

### The Three Rules of Module Encapsulation:

1. **Explicit Public Interface:** The only way to interact with a module is through its declared public contracts.
2. **Hidden Internal Details:** Domain entities, EF Core models, and internal services must never be exposed outside the module.
3. **Data Ownership:** A module strictly owns its database tables. Outside modules cannot query or mutate them directly.

---

## 05 — A Module Strictly Owns Its Data

One of the most critical principles of modular architecture is **Data Encapsulation**:

```text
❌ WRONG: Shared Database Access
[Orders Module] ────────┐
                        │ queries directly
                        ▼
                [Catalog Tables] ◄────── [Catalog Module]
```

If the `Orders` module writes SQL queries directly against `Catalog` tables, any database schema change in `Catalog` silently breaks `Orders`.

```text
✅ CORRECT: Module Owns Its Data
[Orders Module] ──(Calls ICatalogModuleApi)──► [Catalog Module]
                                                     │
                                                     ▼ queries
                                              [Catalog Tables]
```

### Practical Rules for Module Data Ownership:

* Each module owns its tables (either via a separate database schema like `orders.Orders` and `catalog.Products`, or separate databases).
* Cross-module table joins in SQL (`JOIN catalog.Products ON ...`) are forbidden.
* If a module needs data from another module, it must ask via the public API or consume integration events.

---

## 06 — How Modules Communicate

Modules should communicate through controlled, explicit channels:

```text
1. Direct Synchronous In-Memory Call (via Interface)
   [Catalog Module] ──────(Calls IOrderModuleApi)──────► [Orders Module]

2. Asynchronous Event-Driven (Publish / Subscribe)
   [Orders Module] ───(Publishes OrderPlacedEvent)───► [Event Bus]
                                                            │
                                  ┌─────────────────────────┴────────────────────────┐
                                  ▼                                                  ▼
                        [Inventory Module]                                   [Billing Module]

3. In-Memory Mediator (Commands & Queries)
   [Billing Module] ───(Sends GetCustomerSummaryQuery)──► [Mediator] ──► [Customer Module]
```

### 1. Synchronous In-Memory Calls via Public Interface
* Module A references Module B's public interface and receives a DTO.
* Fast, runs inside the same process, transactional if needed.
* Introduces a compile-time dependency on Module B's contract.

### 2. Asynchronous Integration Events (Publish / Subscribe)
* When an action completes, Module A publishes an `IntegrationEvent` (e.g., `OrderPlacedEvent`).
* Module B subscribes to the event and executes its own logic independently.
* **Zero compile-time coupling** from publisher to subscriber.

### 3. In-Memory Mediator
* Dispatches commands or queries via a mediator (like MediatR) using shared request/response DTOs.
* Keeps classes decoupled while remaining in-process.

---

## 07 — Practical C# Implementation & Project Structure

Here is how a modular boundary is structured in a .NET application:

```text
src/
├── Modules/
│   ├── Ordering/
│   │   ├── Ordering.Contracts/        <-- PUBLIC (Referenced by other modules)
│   │   │   ├── IOrderingModuleApi.cs
│   │   │   ├── OrderSummaryDto.cs
│   │   │   └── OrderPlacedIntegrationEvent.cs
│   │   └── Ordering.Core/             <-- INTERNAL (Hidden implementation)
│   │       ├── Domain/
│   │       │   ├── Order.cs           (internal)
│   │       │   └── OrderItem.cs       (internal)
│   │       ├── Persistence/
│   │       │   └── OrderingDbContext.cs (internal)
│   │       ├── OrderingModuleApi.cs   (internal implementation of IOrderingModuleApi)
│   │       └── OrderingExtensions.cs  (public DI registration)
│   │
│   └── Shipping/
│       ├── Shipping.Contracts/
│       └── Shipping.Core/             <-- References ONLY Ordering.Contracts
```

---

### Step 1: The Public Contract (`Ordering.Contracts`)

```csharp
// Ordering.Contracts/IOrderingModuleApi.cs
namespace Ordering.Contracts;

public interface IOrderingModuleApi
{
    Task<OrderSummaryDto?> GetOrderSummaryAsync(Guid orderId, CancellationToken ct = default);
}

// Ordering.Contracts/OrderSummaryDto.cs
namespace Ordering.Contracts;

public record OrderSummaryDto(
    Guid OrderId,
    Guid CustomerId,
    decimal TotalAmount,
    string Status,
    DateTime CreatedAtUtc);

// Ordering.Contracts/OrderPlacedIntegrationEvent.cs
namespace Ordering.Contracts;

public record OrderPlacedIntegrationEvent(
    Guid OrderId,
    Guid CustomerId,
    decimal TotalAmount,
    DateTime PlacedAtUtc);
```

---

### Step 2: The Internal Core (`Ordering.Core`)

All entities, database contexts, and internal logic are marked **`internal`**:

```csharp
// Ordering.Core/Domain/Order.cs
namespace Ordering.Core.Domain;

// Marked internal: Cannot be referenced or instantiated by external modules
internal class Order
{
    public Guid Id { get; private set; }
    public Guid CustomerId { get; private set; }
    public decimal TotalAmount { get; private set; }
    public OrderStatus Status { get; private set; }

    internal Order(Guid customerId, decimal totalAmount)
    {
        Id = Guid.NewGuid();
        CustomerId = customerId;
        TotalAmount = totalAmount;
        Status = OrderStatus.Created;
    }

    internal void MarkAsPaid()
    {
        Status = OrderStatus.Paid;
    }
}

internal enum OrderStatus
{
    Created,
    Paid,
    Cancelled
}
```

The concrete implementation of the public API:

```csharp
// Ordering.Core/OrderingModuleApi.cs
using Ordering.Contracts;
using Ordering.Core.Persistence;
using Microsoft.EntityFrameworkCore;

namespace Ordering.Core;

internal class OrderingModuleApi : IOrderingModuleApi
{
    private readonly OrderingDbContext _dbContext;

    public OrderingModuleApi(OrderingDbContext dbContext)
    {
        _dbContext = dbContext;
    }

    public async Task<OrderSummaryDto?> GetOrderSummaryAsync(Guid orderId, CancellationToken ct = default)
    {
        var order = await _dbContext.Orders
            .AsNoTracking()
            .FirstOrDefaultAsync(o => o.Id == orderId, ct);

        if (order is null) return null;

        // Map internal entity to public DTO
        return new OrderSummaryDto(
            order.Id,
            order.CustomerId,
            order.TotalAmount,
            order.Status.ToString(),
            DateTime.UtcNow);
    }
}
```

---

### Step 3: Module Registration

```csharp
// Ordering.Core/OrderingExtensions.cs
using Microsoft.Extensions.DependencyInjection;
using Microsoft.Extensions.Configuration;
using Ordering.Contracts;
using Ordering.Core.Persistence;

namespace Ordering.Core;

public static class OrderingExtensions
{
    public static IServiceCollection AddOrderingModule(this IServiceCollection services, IConfiguration config)
    {
        services.AddDbContext<OrderingDbContext>(options =>
            options.UseSqlServer(config.GetConnectionString("OrderingDb")));

        services.AddScoped<IOrderingModuleApi, OrderingModuleApi>();

        return services;
    }
}
```

---

### Step 4: Another Module Consuming Ordering

The `Shipping` module references **only** `Ordering.Contracts`:

```csharp
// Shipping.Core/ShippingService.cs
using Ordering.Contracts; // ONLY public contract!

namespace Shipping.Core;

internal class ShippingService
{
    private readonly IOrderingModuleApi _orderingApi;

    public ShippingService(IOrderingModuleApi orderingApi)
    {
        _orderingApi = orderingApi;
    }

    public async Task PrepareShipmentAsync(Guid orderId)
    {
        var order = await _orderingApi.GetOrderSummaryAsync(orderId);

        if (order is null)
        {
            throw new InvalidOperationException($"Order {orderId} not found.");
        }

        // Process shipping safely using the DTO
    }
}
```

---

## 08 — Cohesion and Coupling in Modules

Modules are designed to maximize **Cohesion** and minimize **Coupling**:

```text
┌─────────────────────────────────┐       ┌─────────────────────────────────┐
│         Catalog Module          │       │          Orders Module          │
│                                 │       │                                 │
│  ┌───────────────┐              │       │  ┌───────────────┐              │
│  │ Product Logic │              │       │  │  Order Logic  │              │
│  └───────┬───────┘              │       │  └───────┬───────┘              │
│          │ High Cohesion        │       │          │ High Cohesion        │
│          ▼                      │       │          ▼                      │
│  ┌───────────────┐              │       │  ┌───────────────┐              │
│  │Pricing & Tax  │              │       │  │LineItem Logic │              │
│  └───────────────┘              │       │  └───────────────┘              │
└────────────────┬────────────────┘       └────────────────▲────────────────┘
                 │                                         │
                 └────────────── Low Coupling ─────────────┘
                            (via Public Contract)
```

* **High Cohesion Inside:** Everything inside the module works toward the module's single business responsibility.
* **Low Coupling Outside:** Outside modules know only about public contracts and DTOs. Changes to internal implementation do not affect other modules.

---

## 09 — Enforcing Module Boundaries

Architectural rules must be enforced automatically to prevent boundary erosion over time:

### 1. Language Modifiers (`internal` in C#)
Mark internal classes `internal` so external code cannot compile if it attempts direct access.

### 2. Separate Assemblies (.csproj)
Place contracts in `Orders.Contracts.csproj` and implementations in `Orders.Core.csproj`. Outside modules only reference `Orders.Contracts.csproj`.

### 3. Architecture Tests (NetArchTest)
Verify module isolation in your CI/CD test suite:

```csharp
[Fact]
public void ShippingModule_ShouldNotDependOn_OrderingCore()
{
    var result = Types.InAssembly(typeof(ShippingService).Assembly)
        .ShouldNot()
        .HaveDependencyOn("Ordering.Core")
        .GetResult();

    Assert.True(result.IsSuccessful, "Shipping module must not depend on Ordering.Core!");
}
```

---

## 10 — Modules as the Stepping Stone to Microservices

A system with strong modular boundaries (a **Modular Monolith**) gives you the advantages of microservices without distributed system complexity:

```text
                       STAGE 1: MODULAR MONOLITH
                 ┌───────────────────────────────────┐
                 │        Single Application         │
                 │                                   │
                 │  [Catalog]   [Orders]   [Billing] │
                 │     │           │          │      │
                 │   Schema      Schema     Schema   │
                 └───────────────────────────────────┘
                                   │
                    When a module requires independent
                   scaling or separate team ownership:
                                   ▼
                       STAGE 2: MICROSERVICES
                 ┌───────────┐  ┌───────────┐  ┌───────────┐
                 │  Catalog  │  │  Orders   │  │  Billing  │
                 │  Service  │  │  Service  │  │  Service  │
                 └─────┬─────┘  └─────┬─────┘  └─────┬─────┘
                       │              │              │
                    Catalog DB     Orders DB     Billing DB
```

> **"If you cannot build a clean modular monolith, what makes you think you can build microservices?"**
>
> — Simon Brown

Starting with a modular monolith allows rapid development, easy refactoring, and zero network latency. When needed, extracting a module into an independent service is straightforward because boundaries are already clean.

---

## 11 — Common Module Anti-Patterns

### 1. The Leaky Module
Exposing EF Core entities or domain objects instead of DTOs in public contracts. Outside modules become coupled to database column names.

### 2. Cross-Module Database Table Joins
Writing SQL queries that join tables owned by different modules. Schema changes in one module break another module silently.

### 3. The "Common" or "Shared" Dumping Ground
Creating a `Shared` or `Common` project where everything is placed when developers are unsure of its home. It quickly becomes the most tightly coupled bottleneck in the system.

### 4. Circular Module Dependencies
Module A calls Module B, and Module B calls Module A. Invert the dependency using events or extract the shared concept into a dedicated contract.

---

## 12 — Mental Model

Think of a module like a **Smartphone**:

```text
┌────────────────────────────────────────┐
│             SMARTPHONE                 │
│                                        │
│   PUBLIC CONTROLS (Screen, Buttons)    │
│   ──► Easy to use                      │
│   ──► Stable interface                 │
│                                        │
│ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─  │
│                                        │
│   INTERNAL HARDWARE (CPU, Battery)     │
│   ──► Hidden inside casing             │
│   ──► Manufacturer can upgrade CPU     │
│       without changing the buttons     │
└────────────────────────────────────────┘
```

* You interact with the phone through its screen and buttons (Public Contract).
* The manufacturer can swap internal circuitry or chips without changing how you interact with the screen (Encapsulation).

---

## 13 — Key Takeaways

* A **Module** is a vertical architectural boundary that encapsulates a single business capability.
* **Layers** decompose horizontally by technical concern; **Modules** decompose vertically by domain capability.
* A module exposes an **explicit public contract** and hides its **internal implementation**.
* A module **strictly owns its data**; cross-module database table joins are prohibited.
* Modules communicate via **public interfaces**, **integration events**, or an **in-memory mediator**.
* Use access modifiers (`internal`), project boundaries, and architecture tests to enforce isolation.
* A well-architected modular monolith is the safest foundation for software evolution.

---
