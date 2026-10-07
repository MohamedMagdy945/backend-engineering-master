# 01 — Domain Layer

The Domain Layer is the **innermost layer** of Clean Architecture.

It represents the **core of the application** — the business rules, business concepts, and business behavior that exist independently of any technical framework or external system.

> **The Domain Layer answers the question: What are the business rules?**

---

## 01 — What Is the Domain Layer?

The Domain Layer contains the most important part of the application: the **business logic**.

It defines:

* What business concepts exist (Entities, Value Objects)
* What rules govern those concepts (Business Rules, Invariants)
* What behavior belongs to those concepts (Domain Methods)

```text
Domain Layer
├── Entities
├── Value Objects
├── Domain Events
├── Enumerations
├── Exceptions
└── Business Rules
```

The Domain Layer has **no dependency on any other layer**.

It does not know about:

```text
❌ Databases
❌ HTTP
❌ Frameworks
❌ External Services
❌ UI
```

---

## 02 — Why Is the Domain Layer Important?

The Domain Layer is the reason the application exists.

An e-commerce application exists because of:

```text
Orders
Products
Customers
Payments
Shipping
```

These are **business concepts**, not technical concepts.

If we remove the database, the business concepts still exist.
If we remove the API, the business concepts still exist.
If we change the framework, the business concepts still exist.

> **The Domain Layer should survive any technical change.**

That is why it sits at the center of the architecture:

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
│   │   │   │   Domain ◄───┼───┼───┼── Core
│   │   │   │              │   │   │   │
│   │   │   └──────────────┘   │   │   │
│   │   └──────────────────────┘   │   │
│   └──────────────────────────────┘   │
└──────────────────────────────────────┘
```

---

## 03 — What Belongs in the Domain Layer?

### Entities

An Entity is a business object that has a **unique identity**.

Two entities are the same if they have the same identity, even if their properties differ.

```csharp
public class Order
{
    public Guid Id { get; private set; }
    public Guid CustomerId { get; private set; }
    public DateTime CreatedAt { get; private set; }
    public OrderStatus Status { get; private set; }

    private readonly List<OrderItem> _items = new();
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();

    public Order(Guid customerId)
    {
        Id = Guid.NewGuid();
        CustomerId = customerId;
        CreatedAt = DateTime.UtcNow;
        Status = OrderStatus.Pending;
    }
}
```

Key characteristics of an Entity:

```text
✔ Has a unique identity (Id)
✔ Has a lifecycle (created, modified, deleted)
✔ Equality is based on identity, not properties
✔ Contains business behavior
```

---

### Value Objects

A Value Object is defined entirely by its **properties**, not by identity.

Two value objects are the same if all their properties are the same.

```csharp
public class Money
{
    public decimal Amount { get; }
    public string Currency { get; }

    public Money(decimal amount, string currency)
    {
        if (amount < 0)
            throw new DomainException("Amount cannot be negative.");

        if (string.IsNullOrWhiteSpace(currency))
            throw new DomainException("Currency is required.");

        Amount = amount;
        Currency = currency;
    }

    public Money Add(Money other)
    {
        if (Currency != other.Currency)
            throw new DomainException("Cannot add different currencies.");

        return new Money(Amount + other.Amount, Currency);
    }
}
```

Key characteristics of a Value Object:

```text
✔ No unique identity
✔ Immutable
✔ Equality is based on properties
✔ Self-validating
✔ Can contain behavior
```

---

### Domain Events

A Domain Event represents something meaningful that happened in the domain.

```csharp
public class OrderCreatedEvent
{
    public Guid OrderId { get; }
    public Guid CustomerId { get; }
    public DateTime OccurredAt { get; }

    public OrderCreatedEvent(Guid orderId, Guid customerId)
    {
        OrderId = orderId;
        CustomerId = customerId;
        OccurredAt = DateTime.UtcNow;
    }
}
```

Domain Events communicate that:

```text
Something happened
     ↓
Other parts of the system may react
```

For example:

```text
Order Created
     ↓
Send Confirmation Email
     ↓
Update Inventory
     ↓
Notify Warehouse
```

---

### Enumerations

Enumerations represent a fixed set of domain values.

```csharp
public enum OrderStatus
{
    Pending,
    Confirmed,
    Shipped,
    Delivered,
    Cancelled
}
```

Or using a richer enumeration pattern:

```csharp
public class OrderStatus
{
    public static readonly OrderStatus Pending = new("Pending");
    public static readonly OrderStatus Confirmed = new("Confirmed");
    public static readonly OrderStatus Shipped = new("Shipped");
    public static readonly OrderStatus Delivered = new("Delivered");
    public static readonly OrderStatus Cancelled = new("Cancelled");

    public string Name { get; }

    private OrderStatus(string name)
    {
        Name = name;
    }
}
```

---

### Domain Exceptions

Domain Exceptions represent violations of business rules.

```csharp
public class DomainException : Exception
{
    public DomainException(string message)
        : base(message)
    {
    }
}
```

Used when a business rule is violated:

```csharp
public void Cancel()
{
    if (Status == OrderStatus.Delivered)
        throw new DomainException("Cannot cancel a delivered order.");

    Status = OrderStatus.Cancelled;
}
```

---

## 04 — Business Rules Live in the Domain

The Domain Layer is where **business rules are enforced**.

Not in the Controller.
Not in the Service.
Not in the Database.

```text
❌ Controller validates business rules
❌ Application Service validates business rules
❌ Database constraint validates business rules

✔ Domain Entity/Value Object validates business rules
```

For example:

```csharp
public class Order
{
    private readonly List<OrderItem> _items = new();

    public void AddItem(Product product, int quantity)
    {
        if (quantity <= 0)
            throw new DomainException("Quantity must be greater than zero.");

        if (Status != OrderStatus.Pending)
            throw new DomainException("Cannot add items to a non-pending order.");

        var item = new OrderItem(product.Id, product.Name, product.Price, quantity);
        _items.Add(item);
    }

    public Money GetTotal()
    {
        var total = _items.Sum(item => item.Price.Amount * item.Quantity);
        return new Money(total, "USD");
    }
}
```

The business rules are:

```text
1. Quantity must be greater than zero.
2. Items can only be added to pending orders.
3. Total is calculated from item prices and quantities.
```

These rules belong in the Domain — not anywhere else.

---

## 05 — The Domain Layer Has No Dependencies

This is one of the most important rules of Clean Architecture.

```text
Domain Layer Dependencies:
→ Nothing
```

The Domain Layer does **not** depend on:

```text
❌ Application Layer
❌ Infrastructure Layer
❌ Presentation Layer
❌ EF Core
❌ ASP.NET
❌ Any NuGet package (ideally)
❌ Any external library
```

The Domain Layer depends only on:

```text
✔ The programming language (C#)
✔ The standard library (.NET Base Class Library)
```

This means:

```text
Domain.csproj
├── No reference to Application
├── No reference to Infrastructure
├── No reference to Presentation
├── No reference to EF Core
└── No reference to external packages
```

> **The Domain Layer is the most stable part of the application.**

---

## 06 — Why No Dependencies?

Because the Domain Layer represents the **business**, not the technology.

If the Domain Layer depends on EF Core:

```text
Domain
   ↓
EF Core
   ↓
SQL Server
```

Then changing the database technology forces changes in the Domain.

If the Domain Layer has no dependencies:

```text
Domain
   ↓
(nothing)
```

Then the business rules remain stable regardless of technology changes.

```text
Change Database    → Domain unchanged
Change Framework   → Domain unchanged
Change API         → Domain unchanged
Change Cloud       → Domain unchanged
```

> **The Domain Layer should be the last thing that changes.**

---

## 07 — Abstractions in the Domain Layer

The Domain Layer can define **abstractions** (interfaces) that other layers implement.

For example:

```csharp
// Defined in the Domain Layer
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(Guid id);
    Task AddAsync(Order order);
    Task SaveChangesAsync();
}
```

The implementation lives in Infrastructure:

```csharp
// Defined in the Infrastructure Layer
public class OrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;

    public OrderRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<Order?> GetByIdAsync(Guid id)
    {
        return await _context.Orders
            .Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id);
    }

    public async Task AddAsync(Order order)
    {
        await _context.Orders.AddAsync(order);
    }

    public async Task SaveChangesAsync()
    {
        await _context.SaveChangesAsync();
    }
}
```

The dependency direction:

```text
Infrastructure
      ↓
IOrderRepository (Domain)
      ↑
Application
```

The Domain defines **what** is needed.
Infrastructure provides **how** it is done.

---

## 08 — Domain Layer Folder Structure

A typical Domain Layer structure:

```text
Domain/
├── Entities/
│   ├── Order.cs
│   ├── OrderItem.cs
│   ├── Product.cs
│   └── Customer.cs
├── ValueObjects/
│   ├── Money.cs
│   ├── Address.cs
│   └── Email.cs
├── Events/
│   ├── OrderCreatedEvent.cs
│   └── OrderCancelledEvent.cs
├── Enums/
│   └── OrderStatus.cs
├── Exceptions/
│   └── DomainException.cs
├── Interfaces/
│   ├── IOrderRepository.cs
│   └── IProductRepository.cs
└── Rules/
    └── OrderRules.cs
```

This is one way to organize the Domain Layer.

The exact structure can vary, but the responsibility remains the same:

> **Everything in this layer is about the business.**

---

## 09 — Entities Should Protect Their State

Entities should not expose setters freely.

Bad example:

```csharp
// ❌ Public setters allow external code to break business rules
public class Order
{
    public Guid Id { get; set; }
    public OrderStatus Status { get; set; }
    public List<OrderItem> Items { get; set; }
}
```

The problem:

```text
Anyone can set Status to anything.
Anyone can modify Items directly.
No business rules are enforced.
```

Good example:

```csharp
// ✔ Private setters enforce business rules through methods
public class Order
{
    public Guid Id { get; private set; }
    public OrderStatus Status { get; private set; }

    private readonly List<OrderItem> _items = new();
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();

    public void Confirm()
    {
        if (Status != OrderStatus.Pending)
            throw new DomainException("Only pending orders can be confirmed.");

        if (!_items.Any())
            throw new DomainException("Cannot confirm an order with no items.");

        Status = OrderStatus.Confirmed;
    }
}
```

> **The entity controls its own state transitions.**

---

## 10 — Rich Domain Model vs Anemic Domain Model

### Anemic Domain Model

An anemic model has entities that are just data containers with no behavior:

```csharp
// ❌ Anemic — no behavior, just data
public class Order
{
    public Guid Id { get; set; }
    public OrderStatus Status { get; set; }
    public List<OrderItem> Items { get; set; }
    public decimal Total { get; set; }
}
```

The behavior lives elsewhere:

```csharp
// ❌ Business logic outside the entity
public class OrderService
{
    public void ConfirmOrder(Order order)
    {
        if (order.Status != OrderStatus.Pending)
            throw new Exception("Cannot confirm.");

        if (!order.Items.Any())
            throw new Exception("No items.");

        order.Status = OrderStatus.Confirmed;
        order.Total = order.Items.Sum(i => i.Price * i.Quantity);
    }
}
```

### Rich Domain Model

A rich model keeps behavior inside the entity:

```csharp
// ✔ Rich — behavior is part of the entity
public class Order
{
    public Guid Id { get; private set; }
    public OrderStatus Status { get; private set; }

    private readonly List<OrderItem> _items = new();
    public IReadOnlyCollection<OrderItem> Items => _items.AsReadOnly();

    public void Confirm()
    {
        if (Status != OrderStatus.Pending)
            throw new DomainException("Only pending orders can be confirmed.");

        if (!_items.Any())
            throw new DomainException("Cannot confirm an order with no items.");

        Status = OrderStatus.Confirmed;
    }

    public Money GetTotal()
    {
        var total = _items.Sum(i => i.Price.Amount * i.Quantity);
        return new Money(total, "USD");
    }
}
```

Comparison:

```text
Anemic Domain Model
├── Entities are data containers
├── Business logic lives in services
├── Rules are scattered
└── Easy to break invariants

Rich Domain Model
├── Entities contain behavior
├── Business logic lives with the data
├── Rules are centralized
└── Invariants are enforced
```

> **Clean Architecture favors Rich Domain Models.**

---

## 11 — Domain Layer and Testing

Because the Domain Layer has **no dependencies**, it is the easiest layer to test.

```csharp
[Fact]
public void Order_AddItem_ShouldAddItemToOrder()
{
    // Arrange
    var order = new Order(Guid.NewGuid());
    var product = new Product("Laptop", new Money(999.99m, "USD"));

    // Act
    order.AddItem(product, 2);

    // Assert
    Assert.Single(order.Items);
    Assert.Equal(2, order.Items.First().Quantity);
}
```

```csharp
[Fact]
public void Order_AddItem_ShouldThrowWhenQuantityIsZero()
{
    // Arrange
    var order = new Order(Guid.NewGuid());
    var product = new Product("Laptop", new Money(999.99m, "USD"));

    // Act & Assert
    Assert.Throws<DomainException>(() => order.AddItem(product, 0));
}
```

```csharp
[Fact]
public void Order_Confirm_ShouldThrowWhenNoItems()
{
    // Arrange
    var order = new Order(Guid.NewGuid());

    // Act & Assert
    Assert.Throws<DomainException>(() => order.Confirm());
}
```

No mocking is required.
No database is required.
No HTTP is required.

```text
Domain Tests
├── No mocks
├── No database
├── No HTTP
├── Fast execution
└── Pure business logic verification
```

> **If your Domain Layer is hard to test, it probably has dependencies it should not have.**

---

## 12 — Common Mistakes

### Mistake 1: Domain depends on Infrastructure

```text
❌ Domain → EF Core
❌ Domain → SQL Server
❌ Domain → HttpClient
```

The Domain should never reference infrastructure packages.

---

### Mistake 2: Business rules in the Application Layer

```text
❌ Application Service checks if an order can be cancelled
✔ Order Entity checks if it can be cancelled
```

Business rules belong in the Domain, not in Application Services.

---

### Mistake 3: Anemic Entities with logic in Services

```text
❌ Order has public setters, OrderService has all the logic
✔ Order has private setters and methods that enforce rules
```

---

### Mistake 4: Using framework types in the Domain

```text
❌ Using EF Core attributes ([Key], [Required]) in Domain Entities
❌ Using ASP.NET types in Domain classes
✔ Domain classes use only standard C# types
```

Configuration should be done externally (e.g., EF Core Fluent API in Infrastructure).

---

### Mistake 5: Domain Layer becomes a shared library

```text
❌ Putting DTOs in the Domain
❌ Putting API models in the Domain
❌ Putting utility classes in the Domain
```

The Domain Layer should contain **only business concepts**.

---

## 13 — Mental Model

Think of the Domain Layer as the **heart** of the application.

```text
┌──────────────────────────────────────┐
│    External World (HTTP, UI, etc.)   │
│                                      │
│   ┌──────────────────────────────┐   │
│   │  Technical Details (DB, etc) │   │
│   │                              │   │
│   │   ┌──────────────────────┐   │   │
│   │   │  Application Logic   │   │   │
│   │   │                      │   │   │
│   │   │   ┌──────────────┐   │   │   │
│   │   │   │              │   │   │   │
│   │   │   │    Domain    │   │   │   │
│   │   │   │   (Heart)    │   │   │   │
│   │   │   │              │   │   │   │
│   │   │   └──────────────┘   │   │   │
│   │   └──────────────────────┘   │   │
│   └──────────────────────────────┘   │
└──────────────────────────────────────┘
```

Everything else exists to **serve** the Domain.

The Domain does not serve anyone — it **defines the business**.

```text
Presentation  → serves the Domain
Application   → orchestrates the Domain
Infrastructure → supports the Domain
Domain        → IS the business
```

---

## 14 — Summary

```text
Domain Layer
├── Contains business rules and concepts
├── Has no external dependencies
├── Defines Entities, Value Objects, Events, Enums, Exceptions
├── Can define abstractions (interfaces)
├── Enforces invariants and business rules
├── Is the most stable layer
├── Is the easiest layer to test
└── Is the core of Clean Architecture
```

> **The Domain Layer is the most important layer. Protect it.**

---
