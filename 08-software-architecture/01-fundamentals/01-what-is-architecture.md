# 01 — What is Software Architecture?

Software architecture is the **high-level structure of a software system**.

It defines:

* The major parts of the system
* The responsibilities of each part
* How those parts communicate
* How dependencies are organized
* Where important business rules live
* Which parts are allowed to depend on other parts

The goal of architecture is not to create more folders or more classes.

The goal is to **control complexity, manage dependencies, and make the system easier to understand, maintain, test, and change**.

---

## 1. Simple Definition

> **Software architecture is the set of important structural decisions that define how a software system is organized and how its major components depend on and communicate with each other.**

For example, in a backend application:

```text
Client
  ↓
API
  ↓
Application
  ↓
Domain
  ↑
Infrastructure
  ↓
Database
```

The architecture defines the boundaries between these parts and the direction of their dependencies.

---

# 2. Why Do We Need Software Architecture?

Small applications can sometimes work without carefully designed architecture.

For example:

```csharp
public class OrderService
{
    public void CreateOrder(Order order)
    {
        // Validate order
        // Calculate price
        // Check stock
        // Save to database
        // Send email
        // Send notification
    }
}
```

This might work initially.

But as the application grows, the class can become responsible for too many things.

Now imagine that we need to:

* Change SQL Server to PostgreSQL
* Change the email provider
* Change the payment provider
* Add another API
* Test business rules independently
* Add background processing
* Change authentication
* Add new business rules

If everything is tightly connected, every change becomes more difficult.

Good architecture helps us **control these dependencies**.

---

# 3. The Main Goal of Architecture

The main goal is not:

```text
More Layers
More Classes
More Interfaces
More Design Patterns
```

Instead, the goal is:

```text
Clear Responsibilities
        +
Controlled Dependencies
        +
Good Boundaries
        +
Easy Change
```

A useful way to think about architecture is:

> **Architecture is about managing change.**

We want changes in one part of the system to have as little unnecessary impact as possible on unrelated parts.

---

# 4. Architecture Is About Boundaries

Consider an ERP system.

It may contain:

```text
Authentication
Products
Inventory
Orders
Customers
Payments
Notifications
Reporting
```

We don't want every part of the system to directly depend on every other part.

Instead, we establish boundaries.

For example:

```text
        Presentation
             ↓
        Application
             ↓
          Domain
             ↑
        Infrastructure
```

Each part has a specific responsibility.

The important question is not:

> "How many layers should I create?"

The important question is:

> **"Where should the boundaries exist, and what is each part allowed to depend on?"**

---

# 5. Architecture vs Code

Architecture is not the same thing as writing classes and methods.

### Code

```csharp
public decimal CalculateTotal(Order order)
{
    return order.Items.Sum(x => x.Price * x.Quantity);
}
```

This is implementation.

### Architecture

Architecture asks:

```text
Where should CalculateTotal live?

Who is allowed to call it?

Should it depend on the database?

Should it depend on ASP.NET?

Should it depend on an external service?

Can it be tested independently?
```

Architecture deals with the **structure and relationships** around the code.

---

# 6. Architecture vs Folder Structure

A common mistake is thinking that architecture means creating folders such as:

```text
Controllers/
Services/
Repositories/
Models/
```

Folders alone do not create good architecture.

You can have a large project with many folders and still have bad architecture.

For example:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
Database
```

does not automatically mean the system is well designed.

We need to understand:

* Responsibilities
* Dependencies
* Coupling
* Cohesion
* Dependency direction
* Boundaries

These concepts determine whether the architecture is actually good.

---

# 7. Architecture and Dependencies

One of the most important questions in architecture is:

> **Who depends on whom?**

For example:

```text
Presentation
     ↓
Application
     ↓
Domain
```

This means Presentation depends on Application, and Application depends on Domain.

But we may not want:

```text
Domain
   ↓
ASP.NET Core
   ↓
SQL Server
```

because the core business rules would now depend on external technologies.

Instead, we want to protect the important parts of the system from unnecessary external dependencies.

This concept will become much clearer when we study:

* Dependency Direction
* Clean Architecture
* Hexagonal Architecture
* Onion Architecture

---

# 8. Architecture and Change

Imagine that your application currently uses SQL Server:

```text
Application
     ↓
SQL Server
```

Later, the company decides to use PostgreSQL.

If business logic is tightly coupled to SQL Server, the change can affect many parts of the system.

A better architecture tries to isolate infrastructure concerns:

```text
             Business
                ↑
                │
          Abstraction
                ↑
                │
        Infrastructure
           /         \
      SQL Server   PostgreSQL
```

The business rules should not need to know which database implementation is being used.

---

# 9. Architecture Is Not About Making Everything Abstract

Another common mistake is:

> "Good architecture means creating an interface for everything."

Not necessarily.

For example:

```csharp
IProductService
IProductRepository
IProductValidator
IProductFactory
IProductManager
IProductProvider
IProductHandler
```

Adding abstractions without a real architectural reason can increase complexity.

The purpose of abstraction is to **protect a boundary or control a dependency**, not simply to create more interfaces.

We should always ask:

> **What problem does this abstraction solve?**

---

# 10. Architecture and Business Rules

In business applications, the most important part is often the business logic.

For example, an ERP may have rules such as:

```text
An order cannot be confirmed without stock.

A cancelled order cannot be shipped.

A product price cannot be negative.

A user cannot perform an operation without permission.
```

These rules are more important than the framework or database.

Good architecture tries to keep important business rules from becoming unnecessarily dependent on:

```text
ASP.NET Core
EF Core
SQL Server
HTTP
Angular
Redis
External APIs
```

Technologies can change.

Business rules usually represent the actual purpose of the system.

---

# 11. The Four Fundamental Questions

When looking at any architecture, ask:

### 1. Separation of Concerns

> What is each part responsible for?

### 2. Coupling

> How dependent are these parts on each other?

### 3. Cohesion

> Are related responsibilities kept together?

### 4. Dependency Direction

> Which part is allowed to depend on which part?

These four concepts form the foundation for many architectural styles.

---

# 12. Example

Consider this structure:

```text
┌─────────────────────┐
│    Presentation     │
│   API / Endpoints   │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│     Application     │
│ Use Cases / Logic   │
└──────────┬──────────┘
           │
           ↓
┌─────────────────────┐
│       Domain        │
│ Business Rules      │
└─────────────────────┘

┌─────────────────────┐
│   Infrastructure    │
│ DB / External APIs  │
└─────────────────────┘
```

This structure is not automatically "good" just because it looks organized.

We still need to ask:

```text
Who depends on whom?

Where are the business rules?

Can the business rules be tested independently?

Can infrastructure change without rewriting the domain?

Are responsibilities clearly separated?
```

These questions are the beginning of architectural thinking.

---

# 13. Key Principles

Remember these ideas:

### Architecture is about structure

```text
Components
Boundaries
Dependencies
Communication
```

### Architecture is about responsibilities

```text
Each part should have a clear purpose.
```

### Architecture is about dependencies

```text
Dependencies should be intentional.
```

### Architecture is about change

```text
Changes should have controlled impact.
```

### Architecture is not complexity

```text
More patterns ≠ Better Architecture
More layers   ≠ Better Architecture
More classes  ≠ Better Architecture
```

---

# 14. Mental Model

Keep this mental model while studying architecture:

```text
                 SOFTWARE SYSTEM
                       │
          ┌────────────┼────────────┐
          ↓            ↓            ↓
   Responsibilities  Boundaries  Dependencies
          │            │            │
          └────────────┼────────────┘
                       ↓
                Controlled Change
```

The fundamental question is:

> **How can I organize this system so that its important business rules remain understandable and changes remain manageable?**

---

## Summary

Software architecture is not about creating complicated projects.

It is about making **intentional structural decisions**.

The key things architecture controls are:

```text
Responsibilities
      ↓
Boundaries
      ↓
Dependencies
      ↓
Communication
      ↓
Change
```

### Remember

> **Good architecture makes the important parts of the system easier to understand and change without unnecessary impact.**
