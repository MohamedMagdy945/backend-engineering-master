# 03 — Dependency Inversion

**Dependency Inversion Principle (DIP)** is one of the most critical structural mechanisms in modern software architecture.

It is the fifth principle of the SOLID design principles, and it provides the primary mechanism that allows Clean Architecture, Hexagonal Architecture, and Onion Architecture to exist.

> **High-level modules should not depend on low-level modules. Both should depend on abstractions.**
>
> **Abstractions should not depend on details. Details should depend on abstractions.**
>
> — Robert C. Martin (Uncle Bob)

---

# 1. The Core Problem: Traditional Dependency Direction

In traditional software design, high-level business policies naturally depend on low-level technical mechanisms:

```text
┌──────────────────────────────────────┐
│        High-Level Module             │
│   (Business Logic: OrderService)     │
└──────────────────┬───────────────────┘
                   │
                   ▼ depends on
┌──────────────────────────────────────┐
│         Low-Level Module             │
│   (Technical Detail: SqlOrderRepo)   │
└──────────────────────────────────────┘
```

When high-level business logic directly references low-level database queries, file storage, or external HTTP clients:

1. **Business rules become fragile:** A change in the database schema, ORM version, or cloud SDK breaks your business logic.
2. **Untestable code:** You cannot test `OrderService` in isolation without connecting to a real SQL database or external API.
3. **Rigid architecture:** Replacing SQL Server with PostgreSQL, CosmosDB, or an in-memory test store requires rewriting the business services.

---

# 2. Inverting the Dependency

The Dependency Inversion Principle **inverts** the compile-time dependency arrow by introducing an **abstraction** (interface) between the two modules:

```text
BEFORE DIP (Direct Dependency):
High-Level Business Logic ──────────────► Low-Level Database Detail

AFTER DIP (Inverted Dependency):
High-Level Business Logic ──────────────► [ Abstraction (Interface) ]
                                                        ▲
                                                        │ implements
                                          Low-Level Database Detail
```

Notice what happened:
* The high-level module no longer depends on the low-level module.
* Both modules depend on the abstraction.
* The compile-time dependency arrow of the low-level module has been **reversed** (inverted): it now points inward toward the abstraction!

---

# 3. Who Owns the Abstraction? (The Interface Ownership Rule)

A common mistake when applying DIP is placing the interface in the wrong layer or module:

```text
❌ WRONG: Infrastructure Owns the Interface
┌───────────────────────────────┐
│     Application Layer         │
│         OrderService          │
└───────────────┬───────────────┘
                │ depends on
                ▼
┌───────────────────────────────┐
│     Infrastructure Layer      │
│  ├── ISqlOrderRepository      │  <-- Defined by Infrastructure!
│  └── SqlOrderRepository       │
└───────────────────────────────┘
```

If the Infrastructure layer defines the interface, the Application layer still depends on the Infrastructure layer! The dependency has **not** been inverted.

### The Correct Way: The Consumer Owns the Interface

In Clean Architecture, **the consumer (the higher-level layer) defines the interface it needs**:

```text
✅ CORRECT: Application Owns the Interface
┌───────────────────────────────┐
│     Application Layer         │
│  ├── OrderService             │
│  └── IOrderRepository         │  <-- Defined by Application!
└───────────────▲───────────────┘
                │ implements (depends on)
┌───────────────┴───────────────┐
│     Infrastructure Layer      │
│  └── SqlOrderRepository       │
└───────────────────────────────┘
```

> **The client specifies the contract it needs to accomplish its task. The provider adapts to the client, not the other way around.**

---

# 4. DIP vs. DI vs. IoC (Clearing the Confusion)

These three terms are frequently confused, but they operate at different conceptual levels:

```text
┌────────────────────────────────────────────────────────┐
│ Inversion of Control (IoC)                             │
│ The broad architectural concept:                       │
│ "Don't call us, we'll call you" (Hollywood Principle). │
│                                                        │
│   ┌────────────────────────────────────────────────┐   │
│   │ Dependency Inversion Principle (DIP)           │   │
│   │ The architectural design guideline:            │   │
│   │ High-level code depends on abstractions.       │   │
│   │                                                │   │
│   │   ┌────────────────────────────────────────┐   │   │
│   │   │ Dependency Injection (DI)              │   │   │
│   │   │ The tactical design pattern / tool:    │   │   │
│   │   │ Passing dependencies via constructors  │   │   │
│   │   │ or containers at runtime.              │   │   │
│   │   └────────────────────────────────────────┘   │   │
│   └────────────────────────────────────────────────┘   │
└────────────────────────────────────────────────────────┘
```

* **IoC (Inversion of Control):** General concept of transferring control of flow to an external framework (e.g., ASP.NET Core request pipeline calling your endpoint).
* **DIP (Dependency Inversion Principle):** The architectural principle dictating that high-level policies must not depend on low-level details.
* **DI (Dependency Injection):** The practical coding technique used to provide concrete instances to classes via constructor parameters.

---

# 5. Practical C# Code Comparison

### ❌ Without Dependency Inversion (Tightly Coupled)

```csharp
// Low-level detail
public class SmtpEmailSender
{
    public void Send(string to, string body)
    {
        // Concrete SMTP connection to smtp.office365.com
    }
}

// High-level business logic
public class OrderService
{
    private readonly SmtpEmailSender _emailSender;

    public OrderService()
    {
        // Direct instantiation of technical detail!
        _emailSender = new SmtpEmailSender();
    }

    public void CompleteOrder(Order order)
    {
        order.MarkAsCompleted();
        _emailSender.Send(order.CustomerEmail, "Your order is complete!");
    }
}
```

**Why this fails:**
- `OrderService` cannot be unit tested without attempting to send real SMTP emails.
- If the company moves to SendGrid or AWS SES, `OrderService` must be modified and recompiled.

---

### ✅ With Dependency Inversion (Decoupled & Testable)

#### 1. High-Level Layer (Application) defines the abstraction:

```csharp
namespace Application.Common.Interfaces;

// The Application layer declares what it needs
public interface IEmailSender
{
    Task SendEmailAsync(string recipient, string subject, string message, CancellationToken ct = default);
}
```

#### 2. High-Level Business Service depends strictly on the abstraction:

```csharp
namespace Application.Orders.Services;

using Application.Common.Interfaces;

public class OrderService
{
    private readonly IEmailSender _emailSender;

    // Dependency supplied via Constructor Injection (DI)
    public OrderService(IEmailSender emailSender)
    {
        _emailSender = emailSender;
    }

    public async Task CompleteOrderAsync(Order order, CancellationToken ct = default)
    {
        order.MarkAsCompleted();

        await _emailSender.SendEmailAsync(
            order.CustomerEmail,
            "Order Confirmation",
            $"Order {order.Id} has been completed.",
            ct);
    }
}
```

#### 3. Low-Level Layer (Infrastructure) provides the implementation:

```csharp
namespace Infrastructure.Notifications;

using Application.Common.Interfaces;
using SendGrid;

// Infrastructure depends on Application to implement its interface!
public class SendGridEmailSender : IEmailSender
{
    private readonly ISendGridClient _client;

    public SendGridEmailSender(ISendGridClient client)
    {
        _client = client;
    }

    public async Task SendEmailAsync(string recipient, string subject, string message, CancellationToken ct = default)
    {
        // SendGrid API calls
    }
}
```

---

# 6. Runtime Call Flow vs. Compile-Time Dependency Direction

One of the most profound insights about DIP is how it separates **Runtime Execution** from **Compile-Time Dependencies**:

```text
RUNTIME CALL FLOW (Data & Execution Direction):
[Client] ──► [Controller] ──► [OrderService] ──► [SqlOrderRepo] ──► [Database]
                                     ▲
                                     │ calls at runtime
                                     ▼
                             (Executes concrete method)


COMPILE-TIME DEPENDENCY DIRECTION (Source Code References):
[Controller] ────────► [OrderService]
                            │
                            ▼ depends on
                    [IOrderRepository]  ◄──── implements (depends on) ──── [SqlOrderRepo]
                    (Application Core)                                    (Infrastructure)
```

* **At Runtime:** The CPU executes instructions in the natural forward direction: Controller → Service → Repository → Database.
* **At Compile Time:** The Infrastructure project has a project reference pointing to Application, but Application has **zero** project references pointing to Infrastructure.

---

# 7. How DIP Protects Architecture Over Time

Consider what happens over a 5-year application lifecycle:

```text
Year 1: SQL Server (EF Core)
Year 2: Redis added for caching
Year 3: Swapped SMTP Email for SendGrid
Year 4: Migrated Blob storage from Local Disk to Azure Blob Storage
Year 5: Upgraded .NET version and ORM packages
```

With Dependency Inversion:
* Every single one of these technical changes was contained entirely within the **Infrastructure** layer.
* The **Domain** and **Application** layers did not change a single line of business code.
* Business rules remain stable, protected, and deterministic.

---

# 8. When Is DIP Overkill?

Architecture is about balancing trade-offs. DIP is not mandatory for every single class in your solution:

| Scenario | Use DIP? | Rationale |
| :--- | :--- | :--- |
| **External I/O (Databases, APIs, Email, Cloud)** | **YES** | Volatile, external, slow, hard to test without abstractions. |
| **Domain Logic & Entities** | **NO** | Pure, deterministic business logic. No I/O dependencies. |
| **Standard Utilities (DateTime, Math, String helpers)** | **NO** | Deterministic, framework-native, negligible change frequency. |
| **Simple Internal Helper Functions** | **NO** | Creating an interface for a private parsing helper adds unnecessary noise. |

---

# 9. Key Takeaways

* **Dependency Inversion** decouples high-level policy from low-level technical mechanism.
* High-level modules must never depend directly on low-level implementation details; both depend on abstractions.
* **Interface Ownership:** The consumer layer (Application) owns the abstraction, and the implementation layer (Infrastructure) adapts to it.
* DIP inverts the **compile-time source code dependency** while leaving runtime execution flow unchanged.
* DIP is the architectural engine that enables Clean, Hexagonal, and Onion architectures to isolate business cores.

---

Next:

**[04 — Dependency Direction](04-dependency-direction.md)**
