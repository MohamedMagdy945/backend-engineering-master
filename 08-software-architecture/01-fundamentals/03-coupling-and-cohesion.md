# 03 — Coupling and Cohesion

**Coupling** and **Cohesion** are fundamental concepts used to evaluate software design.

They answer two different questions:

> **Coupling:** How dependent are components on each other?

> **Cohesion:** How closely related are the responsibilities inside a component?

A common design goal is:

```text
Low Coupling
     +
High Cohesion
```

---

# 1. Coupling

**Coupling** describes the dependency between components.

For example:

```text
OrderService
     ↓
PaymentService
```

`OrderService` depends on `PaymentService`.

The more knowledge and dependency one component has on another, the stronger the coupling.

---

## High Coupling

```csharp
public class OrderService
{
    public void CreateOrder(Order order)
    {
        var connection = new SqlConnection(connectionString);

        var stripe = new StripeClient("secret-key");

        // Database operations
        // Payment operations
        // Business logic
    }
}
```

`OrderService` now knows about:

```text
SQL Server
Stripe
Database implementation
Payment implementation
Business rules
```

This creates strong coupling.

If the database or payment provider changes, `OrderService` may also need to change.

---

## Lower Coupling

We can introduce a meaningful abstraction:

```csharp
public interface IPaymentService
{
    Task<Result> PayAsync(decimal amount);
}
```

Then:

```csharp
public class OrderService
{
    private readonly IPaymentService paymentService;

    public OrderService(IPaymentService paymentService)
    {
        this.paymentService = paymentService;
    }

    public async Task<Result> CreateOrder(Order order)
    {
        // Business logic

        return await paymentService.PayAsync(order.Total);
    }
}
```

Now:

```text
             OrderService
                  │
                  ↓
          IPaymentService
             /          \
            ↓            ↓
        Stripe        Another Provider
```

`OrderService` doesn't need to know the concrete payment implementation.

---

# 2. Low Coupling Does Not Mean Zero Dependencies

This is important.

We cannot build useful software without dependencies.

For example:

```text
Endpoint
   ↓
Handler
   ↓
Domain
```

These are still dependencies.

The goal is:

> **Keep dependencies necessary, intentional, and controlled.**

Not:

> Remove every dependency.

---

# 3. Cohesion

**Cohesion** describes how closely related the responsibilities inside a component are.

For example:

```csharp
public class Order
{
    public decimal CalculateTotal()
    {
        // Calculate order total
    }

    public Result Confirm()
    {
        // Confirm order
    }

    public Result Cancel()
    {
        // Cancel order
    }
}
```

All these operations are related to an **Order**.

Therefore, the class has relatively **high cohesion**.

---

# 4. Low Cohesion

Consider:

```csharp
public class ApplicationService
{
    public void CreateOrder()
    {
    }

    public void SendEmail()
    {
    }

    public void ResizeImage()
    {
    }

    public void CalculateSalary()
    {
    }
}
```

These responsibilities are unrelated:

```text
Orders
Email
Images
Payroll
```

This is **low cohesion**.

A better design would separate them:

```text
OrderService
EmailService
ImageService
PayrollService
```

Each component now has a clearer purpose.

---

# 5. Coupling vs Cohesion

The easiest way to remember the difference:

```text
Coupling
   ↓
Between components
```

```text
Cohesion
   ↓
Inside a component
```

Example:

```text
       Component A
      ┌─────────────┐
      │ Related     │
      │ operations  │  ← Cohesion
      └─────────────┘
             │
             │ Dependency
             ↓
       Component B
```

Ask:

**Coupling:**

> How dependent is A on B?

**Cohesion:**

> Do the things inside A actually belong together?

---

# 6. Why They Matter

### High coupling can cause:

* Difficult changes
* Difficult testing
* More dependencies
* Changes spreading across the system
* Strong dependency on infrastructure or frameworks

### Low cohesion can cause:

* Large classes
* Unclear responsibilities
* Difficult maintenance
* Difficult testing
* God classes

A common target is therefore:

```text
┌──────────────────────────┐
│      Good Design         │
│                          │
│   Low Coupling           │
│         +                │
│   High Cohesion          │
└──────────────────────────┘
```

---

# 7. Coupling and Change

One practical way to identify coupling is to ask:

> **If this component changes, what else needs to change?**

For example:

```text
Payment Provider changes
        ↓
Only payment implementation changes
```

This indicates that the payment dependency is relatively well isolated.

But if changing the payment provider requires changes to:

```text
Order
Inventory
Customers
Reports
Authentication
```

then the system likely has excessive coupling between those areas.

---

# 8. Cohesion and Responsibility

A useful question when designing a class is:

> **Do these responsibilities have a strong reason to exist together?**

Good:

```text
Order
├── CalculateTotal()
├── Confirm()
└── Cancel()
```

These all represent order behavior.

Questionable:

```text
OrderService
├── CreateOrder()
├── SendEmail()
├── GeneratePdf()
├── ResizeImage()
└── CalculateSalary()
```

The responsibilities have little relationship.

---

# 9. Relationship With Separation of Concerns

We previously learned:

> **Separation of Concerns means keeping different responsibilities appropriately separated.**

Coupling and cohesion help us evaluate the resulting design.

```text
Separation of Concerns
          ↓
Clear Responsibilities
          ↓
Good Boundaries
          ↓
Low Coupling + High Cohesion
```

They are not completely independent concepts.

Good separation often helps us achieve better cohesion and lower unnecessary coupling.

---

# 10. Practical Architecture Example

Consider a .NET application:

```text
Presentation
     ↓
Application
     ↓
Domain
     ↑
Infrastructure
```

We want:

* Presentation to handle presentation concerns.
* Application to coordinate use cases.
* Domain to contain business rules.
* Infrastructure to handle technical implementations.

This separation helps control coupling.

At the same time, each area should contain related responsibilities, giving us better cohesion.

---

# 11. Common Warning Signs

### Possible high coupling

```text
Many direct dependencies
Concrete implementations everywhere
Database code inside business rules
External API details inside domain logic
One change requires many unrelated changes
```

### Possible low cohesion

```text
Very large classes
"God" classes
Unrelated methods in the same class
Generic classes such as CommonService doing many unrelated things
```

These are warning signs, not automatic proof of bad design.

---

# 12. Mental Model

Think about it this way:

```text
Inside a component:

Related responsibilities
        ↓
     Stay together
        ↓
   High Cohesion
```

Between components:

```text
Unnecessary dependencies
        ↓
       Avoid
        ↓
   Lower Coupling
```

Therefore:

```text
       Good Architecture
              │
       ┌──────┴──────┐
       ↓             ↓
High Cohesion   Low Coupling
```

---

# 13. Key Takeaways

### Coupling

> **The degree of dependency between components.**

### Cohesion

> **The degree to which responsibilities inside a component belong together.**

Remember:

```text
Coupling  → Between
Cohesion  → Inside
```

And the common design goal:

```text
Low Coupling
     +
High Cohesion
```

The purpose isn't to eliminate all dependencies or split everything into tiny classes.

The goal is to create **clear responsibilities and controlled dependencies**.

---