# 02 — Separation of Concerns

Separation of Concerns (SoC) is a fundamental software design and architecture principle.

It means:

> **Different responsibilities should be separated so that each part of the system focuses on a clear and specific concern.**

The goal is not to create more classes or more layers.

The goal is to prevent unrelated responsibilities from becoming unnecessarily mixed together.

---

# 1. What is a Concern?

A **concern** is a responsibility or area of functionality within a software system.

For example, a backend application may have these concerns:

```text
Authentication
Validation
Business Rules
Database Access
Logging
HTTP Responses
Email
Caching
Authorization
```

These are different responsibilities.

The problem starts when many unrelated concerns are placed in the same class, method, or layer.

---

# 2. Bad Example

Consider an API endpoint responsible for creating an order:

```csharp
public async Task<IResult> CreateOrder(CreateOrderRequest request)
{
    // Validate request

    if (string.IsNullOrWhiteSpace(request.CustomerId))
        return Results.BadRequest();

    // Connect to database

    using var connection = new SqlConnection(connectionString);

    // Get customer
    // Check stock
    // Calculate price
    // Create order
    // Save order

    // Send email

    // Write log

    return Results.Ok();
}
```

This single method contains many different concerns:

```text
CreateOrder
│
├── Validation
├── Database Access
├── Business Rules
├── Persistence
├── Email
├── Logging
└── HTTP Response
```

The code may work, but the responsibilities are mixed together.

---

# 3. Why Is This a Problem?

Imagine the application changes.

### Database changes

SQL Server is replaced with PostgreSQL.

### Email changes

The application changes its email provider.

### Business rules change

The rules for confirming orders change.

### API changes

The application exposes the same functionality through another interface.

If everything is mixed together, these changes can affect the same large piece of code.

This increases:

* Complexity
* Coupling
* Testing difficulty
* Maintenance cost
* Risk of breaking unrelated functionality

---

# 4. Separating the Concerns

Instead, we can separate responsibilities:

```text
Presentation
     │
     ▼
Application
     │
     ▼
Domain
     │
     ▼
Infrastructure
     │
     ▼
Database
```

Each part has a clearer responsibility.

### Presentation

Responsible for HTTP/API concerns:

```text
Request
Response
Route
HTTP Status Codes
```

### Application

Responsible for application use cases:

```text
Use Cases
Orchestration
Application Workflows
Transaction Coordination
```

### Domain

Responsible for business rules:

```text
Entities
Value Objects
Business Rules
Domain Behavior
```

### Infrastructure

Responsible for technical concerns:

```text
Database
External APIs
Email
File Storage
Caching
```

---

# 5. Separation of Concerns Is a Principle

Separation of Concerns is **not an architecture style**.

It is a general principle that can be applied using different architectural styles.

For example:

```text
Layered Architecture
Clean Architecture
Hexagonal Architecture
Onion Architecture
Vertical Slice Architecture
```

All of them can apply Separation of Concerns differently.

Therefore:

> **Separation of Concerns is a principle, not a specific folder structure or architecture style.**

---

# 6. Separation of Concerns vs Layers

These concepts are related but different.

### Separation of Concerns

A principle:

> Keep different responsibilities separated.

### Layers

One way to organize those responsibilities:

```text
Presentation
Application
Domain
Infrastructure
```

Other architectural styles may organize the same responsibilities differently.

---

# 7. Vertical Slice Example

Vertical Slice Architecture organizes code primarily around **features/use cases** instead of technical layers.

For example:

```text
Features/
│
├── Orders/
│   ├── Create/
│   │   ├── CreateOrderRequest.cs
│   │   ├── CreateOrderHandler.cs
│   │   ├── CreateOrderValidator.cs
│   │   └── CreateOrderEndpoint.cs
│   │
│   └── Cancel/
│       ├── CancelOrderRequest.cs
│       ├── CancelOrderHandler.cs
│       └── CancelOrderEndpoint.cs
│
└── Products/
    └── Create/
        ├── CreateProductRequest.cs
        ├── CreateProductHandler.cs
        └── CreateProductEndpoint.cs
```

This can still follow Separation of Concerns.

For example:

```text
Endpoint
    ↓
Handler
    ↓
Domain
    ↓
Persistence
```

Each component still has a clear responsibility.

---

# 8. Separation Does NOT Mean One Class Per Responsibility

A common mistake is thinking:

> "Every responsibility must have its own class."

This can lead to unnecessary complexity:

```text
ValidationService
LoggingService
CalculationService
DatabaseService
EmailService
OrderService
OrderManager
OrderProvider
OrderProcessor
...
```

More classes do not automatically mean better architecture.

The real question is:

> **Are the responsibilities appropriately separated?**

Not:

> **How many classes do I have?**

---

# 9. Separation of Concerns and Business Rules

Business rules are especially important.

For example:

```text
An order cannot be confirmed if there is not enough stock.
```

This is a **business concern**.

It should not be hidden inside an HTTP endpoint or a SQL query.

Conceptually, the business rule belongs to the business/domain model.

For example:

```csharp
public Result Confirm()
{
    if (!CanConfirm())
        return Result.Failure("Order cannot be confirmed.");

    Status = OrderStatus.Confirmed;

    return Result.Success();
}
```

The endpoint should not need to understand the internal details of this rule.

---

# 10. Separation of Concerns and Change

One of the biggest benefits of Separation of Concerns is **localized change**.

Imagine the application uses:

```text
Email Provider A
```

and later changes to:

```text
Email Provider B
```

If email concerns are isolated:

```text
Application
     │
     ▼
Email Abstraction
     │
     ▼
Infrastructure
     │
     ├── Provider A
     └── Provider B
```

The rest of the application does not need to know the implementation details of the email provider.

This reduces the impact of change.

---

# 11. Separation of Concerns and Testing

Separation also makes testing easier.

Consider this business rule:

```text
An order cannot be confirmed without enough stock.
```

If this rule is mixed with:

```text
HTTP
SQL Server
Email
Logging
```

testing the rule becomes difficult.

If the business rule is separated:

```text
Order
  ↓
Confirm()
```

we can test the business behavior without starting:

```text
ASP.NET
SQL Server
Email Server
```

This is one reason architecture and testing are closely connected.

---

# 12. Separation Does Not Mean Zero Dependencies

Software components need to communicate.

We are not trying to eliminate every dependency.

For example:

```text
Endpoint
    ↓
Handler
    ↓
Domain
```

is a normal dependency relationship.

The goal is:

> **Keep responsibilities separate while making necessary dependencies explicit and controlled.**

---

# 13. Three Levels of Separation

Separation of Concerns can be applied at different levels.

## Method Level

Avoid one method doing everything:

```text
Validate
Calculate
Save
Notify
```

## Class Level

Avoid one class being responsible for unrelated areas:

```text
Order
Email
Payment
Logging
Database
```

## Architectural Level

Separate major system responsibilities:

```text
Presentation
Application
Domain
Infrastructure
```

So Separation of Concerns applies from **small methods to the entire system**.

---

# 14. A Practical Mental Model

When looking at a class, method, feature, or architecture, ask:

```text
What responsibilities exist here?

Are they related?

Should they be together?

Are unrelated concerns mixed together?

If one responsibility changes, what else will be affected?
```

The last question is especially important:

> **If this responsibility changes, how much unrelated code must change with it?**

If changing one concern forces changes throughout the system, the concerns may not be properly separated.

---

# 15. Common Mistakes

### Mistake 1 — More classes = better architecture

False.

```text
More Classes ≠ Better Architecture
```

### Mistake 2 — More layers = better architecture

False.

```text
More Layers ≠ Better Architecture
```

### Mistake 3 — Every responsibility needs an interface

False.

```text
More Interfaces ≠ Better Architecture
```

### Mistake 4 — Components should never communicate

False.

Components need controlled communication.

### Mistake 5 — Separation means complete independence

False.

The goal is **controlled dependencies**, not zero dependencies.

---

# 16. Benefits

Good Separation of Concerns can improve:

### Maintainability

Changes are easier to locate.

### Testability

Responsibilities can be tested independently.

### Readability

It is easier to understand what each part does.

### Changeability

Changes have a more limited impact.

### Reusability

A responsibility can sometimes be reused without bringing unrelated functionality with it.

### Dependency Management

Dependencies become easier to understand and control.

---

# 17. Mental Model

Keep this model in mind:

```text
                    SYSTEM
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
    Presentation    Business    Infrastructure
          │            │            │
          ↓            ↓            ↓
        HTTP          Rules       Database
```

Each area has a clear responsibility.

The goal is not to prevent communication.

The goal is to prevent this:

```text
                 EVERYTHING
                     │
                     ▼
              One Giant Class
```

---

# 18. Key Takeaways

> **Separation of Concerns means organizing software so that different responsibilities are handled separately instead of being unnecessarily mixed together.**

Remember:

```text
Different Responsibility
          ↓
Separate Appropriate Concern
          ↓
Clear Boundary
          ↓
Controlled Dependency
          ↓
Easier Change
```

And remember:

```text
Separation of Concerns
        ≠
More Classes

Separation of Concerns
        ≠
More Interfaces

Separation of Concerns
        ≠
More Layers
```

The real goal is:

> **Keep responsibilities clear and prevent changes in one concern from unnecessarily affecting unrelated concerns.**

---

This will give us a practical way to evaluate whether our separation is actually good.