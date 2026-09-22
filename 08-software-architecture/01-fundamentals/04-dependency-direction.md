# 04 — Dependency Direction

**Dependency Direction** defines which parts of a software system are allowed to depend on other parts.

It is one of the most important ideas in software architecture because the direction of dependencies determines:

* What can change independently
* Where business rules live
* How easy the system is to test
* How strongly the system is tied to frameworks and infrastructure

---

# 1. What Is a Dependency?

A dependency exists when one component needs another component to work.

For example:

```csharp
public class OrderService
{
    private readonly PaymentService paymentService;

    public OrderService(PaymentService paymentService)
    {
        this.paymentService = paymentService;
    }
}
```

Here:

```text
OrderService
     ↓
PaymentService
```

`OrderService` depends on `PaymentService`.

The arrow means:

> **A depends on B.**

---

# 2. Why Does Direction Matter?

Imagine:

```text
Domain
   ↓
Database
```

Now the business/domain depends directly on the database.

If the database changes:

```text
SQL Server → PostgreSQL
```

the domain may need to change too.

This is undesirable because the database is a technical detail, while the domain contains business rules.

Instead, we generally want to protect the domain from infrastructure details.

---

# 3. A Common Architecture Direction

A common structure is:

```text
Presentation
      ↓
Application
      ↓
Domain
```

Infrastructure provides technical implementations around the core:

```text
          Presentation
                ↓
           Application
                ↓
              Domain
                ↑
          Infrastructure
```

The important idea is that **business rules should not depend directly on infrastructure details**.

---

# 4. Dependency Direction vs Data Flow

These are not always the same thing.

For example, a request may flow:

```text
HTTP Request
     ↓
Endpoint
     ↓
Handler
     ↓
Database
     ↓
HTTP Response
```

This is **request/data flow**.

But the dependency relationships might be:

```text
Endpoint → Application
Application → Domain
Infrastructure → Application/Domain
```

So:

> **The direction data travels through a system is not necessarily the same as the direction dependencies point.**

This distinction becomes very important in Clean Architecture.

---

# 5. The Problem With Direct Infrastructure Dependencies

Consider:

```csharp
public class OrderService
{
    private readonly SqlOrderRepository repository;

    public OrderService(SqlOrderRepository repository)
    {
        this.repository = repository;
    }
}
```

Now the application directly depends on a SQL-specific implementation.

```text
OrderService
     ↓
SqlOrderRepository
     ↓
SQL Server
```

This creates unnecessary coupling.

The application knows too much about infrastructure.

---

# 6. Dependency Through an Abstraction

Instead, define the required behavior:

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(Guid id);
    Task AddAsync(Order order);
}
```

The application depends on the abstraction:

```text
Application
     ↓
IOrderRepository
```

Infrastructure provides the implementation:

```text
Infrastructure
     ↓
SqlOrderRepository
     ↓
IOrderRepository
```

Conceptually:

```text
        Application
             ↓
     IOrderRepository
             ↑
             │
   SqlOrderRepository
             ↓
        SQL Server
```

The important point is that the business/application side does not need to know about SQL Server.

---

# 7. Dependency Inversion

This leads to an important principle:

> **High-level business logic should not depend directly on low-level implementation details. Both should depend on appropriate abstractions.**

Instead of:

```text
Business
   ↓
SQL Server
```

we can have:

```text
Business
   ↓
Abstraction
   ↑
Infrastructure
```

This is a major idea behind **Dependency Inversion Principle (DIP)**.

---

# 8. Why Is This Useful?

Imagine your application uses:

```text
SQL Server
```

and later changes to:

```text
PostgreSQL
```

If the domain/application directly depends on SQL Server:

```text
Application
     ↓
SQL Server
```

the change can spread through the application.

With controlled dependencies:

```text
Application
     ↓
IOrderRepository
     ↑
     │
 ┌───┴──────────┐
 ↓              ↓
SQL Server   PostgreSQL
```

the infrastructure implementation can change without changing the business logic.

---

# 9. Dependency Direction Protects the Core

Think of the system as:

```text
        External World
              │
       ┌──────┴──────┐
       ↓             ↓
   Database       HTTP/API
       │             │
       └──────┬──────┘
              ↓
       Application
              ↓
           Domain
```

The **Domain** contains important business rules.

We want it to be protected from:

```text
ASP.NET Core
EF Core
SQL Server
HTTP
Redis
External APIs
```

These technologies can change.

The business rules should not be forced to change just because a technical detail changes.

---

# 10. Dependency Direction in Clean Architecture

Clean Architecture commonly represents dependencies as pointing **toward the center**:

```text
┌─────────────────────────────┐
│       Infrastructure        │
│                             │
│   ┌─────────────────────┐   │
│   │    Application      │   │
│   │                     │   │
│   │   ┌─────────────┐   │   │
│   │   │   Domain    │   │   │
│   │   └─────────────┘   │   │
│   └─────────────────────┘   │
└─────────────────────────────┘
```

The exact project structure can vary, but the principle remains:

> **Outer technical details should not control the core business rules.**

---

# 11. Dependency Direction in Your .NET Projects

A common setup could be:

```text
Presentation
      ↓
Application
      ↓
Domain
```

and:

```text
Infrastructure
      ↓
Application / Domain
```

For example:

```text
Application
    ↓
IProductRepository
    ↑
    │
ProductRepository
    ↓
EF Core
    ↓
SQL Server
```

The interface represents what the application needs.

Infrastructure decides how that need is implemented.

---

# 12. Don't Create Abstractions Everywhere

Dependency direction does **not** mean:

```text
Everything → Interface → Implementation
```

For example, creating:

```text
IStringHelper
IDateHelper
IProductMapper
IOrderCalculator
```

just because "architecture requires interfaces" can create unnecessary complexity.

Ask:

> **Is there a meaningful boundary or dependency that needs protection?**

If not, an abstraction may not provide much value.

---

# 13. Dependency Direction and Testing

Controlled dependencies also make testing easier.

Instead of:

```text
OrderHandler
     ↓
SQL Server
```

we can have:

```text
OrderHandler
     ↓
IOrderRepository
```

During testing:

```text
OrderHandler
     ↓
Fake/Test Repository
```

The business logic can be tested without requiring a real database.

This is one of the practical benefits of good dependency direction.

---

# 14. A Simple Rule

When designing a system, ask:

> **If this technology changes, should my business rules have to change?**

For example:

```text
SQL Server changes
```

Should:

```text
Order business rules
```

change?

Usually, **no**.

Similarly:

```text
ASP.NET changes
```

should not normally require rewriting core business rules.

This thinking helps determine where dependencies should point.

---

# 15. Mental Model

Remember:

```text
          Technical Details
                 │
                 ↓
          Application
                 │
                 ↓
              Domain
```

The domain is the center of business knowledge.

Dependencies should be arranged so that technical details don't unnecessarily control the core.

Another useful model:

```text
        Implementation
              ↓
          Abstraction
              ↑
        Business Logic
```

The abstraction creates a boundary between the business requirement and its technical implementation.

---

# 16. Key Takeaways

### Dependency

> A component depends on another component when it needs it to perform its work.

### Dependency Direction

> The direction in which those dependencies point is an architectural decision.

### Good direction

Generally:

```text
Business Rules
      ↑
Abstractions
      ↑
Infrastructure Implementations
```

### Remember

```text
Data Flow ≠ Dependency Direction
```

And:

> **Protect important business rules from unnecessary dependencies on technical details.**





