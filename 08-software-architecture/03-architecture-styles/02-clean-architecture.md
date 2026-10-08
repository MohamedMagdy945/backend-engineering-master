تمام. هنخليه **عن Clean Architecture فقط**، بنفس مستوى وتنسيق ملف Layered Architecture، ومن غير مقارنات مع باقي الـ architectures.

# Clean Architecture

Clean Architecture is a software architecture approach that organizes an application around its **business rules and use cases**, while keeping external concerns such as databases, frameworks, and UI details independent from the core of the application.

The main goal is to make the core business logic independent from technical implementation details.

---

## 01 — Core Idea

The central idea of Clean Architecture is:

> **Dependencies should point inward toward the application core.**

A common representation is:

```text
┌─────────────────────────────────────┐
│        Frameworks & Drivers         │
│   API / UI / Database / External    │
├─────────────────────────────────────┤
│        Interface Adapters           │
│ Controllers / Repositories / DTOs   │
├─────────────────────────────────────┤
│        Application / Use Cases      │
│       Application Business Rules    │
├─────────────────────────────────────┤
│          Enterprise Rules            │
│       Entities / Domain Rules       │
└─────────────────────────────────────┘

             Dependencies
                  ↓
              Inward
```

The inner layers should not depend on the outer layers.

---

## 02 — Main Layers

Clean Architecture is commonly represented using four major layers:

```text
Entities
   ↑
Use Cases
   ↑
Interface Adapters
   ↑
Frameworks & Drivers
```

The exact names and project structure can vary, but the dependency rule remains the key concept.

---

## 03 — Entities

Entities contain the most important business rules of the application.

They represent concepts that are meaningful to the business.

Examples:

```text
User
Product
Order
Invoice
Employee
Account
```

An entity should contain behavior that belongs naturally to the business concept.

Example:

```csharp
public class Order
{
    public decimal Total { get; private set; }

    public void ApplyDiscount(decimal percentage)
    {
        if (percentage < 0 || percentage > 100)
            throw new ArgumentOutOfRangeException();

        Total -= Total * percentage / 100;
    }
}
```

The entity does not need to know about:

* HTTP
* SQL Server
* Entity Framework Core
* ASP.NET Core
* Controllers
* UI frameworks

The business rule should remain independent from those details.

---

## 04 — Use Cases

The Use Case layer contains the application's application-specific business rules.

It describes **what the application does**.

Examples:

```text
Create Product
Update Product
Delete Product
Create Order
Cancel Order
Transfer Stock
Register User
```

A use case coordinates the required business operations.

Example:

```text
Create Order
     │
     ├── Validate Customer
     │
     ├── Validate Products
     │
     ├── Calculate Total
     │
     ├── Create Order
     │
     └── Save Order
```

Example structure:

```text
Application/
├── Orders/
│   ├── CreateOrder/
│   ├── CancelOrder/
│   └── GetOrder/
│
└── Products/
    ├── CreateProduct/
    └── UpdateProduct/
```

Use cases should not depend directly on infrastructure implementations.

---

## 05 — Interface Adapters

Interface Adapters convert data between the application's internal representation and external representations.

Examples include:

* Controllers
* API endpoints
* Presenters
* DTOs
* Repository implementations
* Mappers

For example:

```text
HTTP Request
     ↓
Controller
     ↓
Request DTO
     ↓
Use Case
```

And:

```text
Use Case
     ↓
Response DTO
     ↓
Controller
     ↓
HTTP Response
```

The adapter acts as a boundary between the application and the outside world.

---

## 06 — Frameworks & Drivers

This is the outermost layer.

It contains implementation details and external technologies.

Examples:

* ASP.NET Core
* Entity Framework Core
* SQL Server
* PostgreSQL
* Redis
* Angular
* External APIs
* Message brokers

These technologies are details surrounding the application's core.

For example:

```text
Infrastructure/
├── Persistence/
│   ├── AppDbContext.cs
│   └── ProductRepository.cs
│
├── Authentication/
│   └── JwtService.cs
│
└── ExternalServices/
    └── PaymentService.cs
```

The core application should not become dependent on these implementation details.

---

## 07 — The Dependency Rule

The most important rule in Clean Architecture is the **Dependency Rule**.

```text
Outer layers
     ↓
Inner layers
```

Dependencies point inward.

For example:

```text
Infrastructure
      ↓
Application
      ↓
Domain
```

But the domain should not depend on infrastructure:

```text
Domain
   ✕
   ↓
Infrastructure
```

The core should remain independent.

---

## 08 — Dependency Inversion

Clean Architecture uses dependency inversion to allow the inner layers to define abstractions while the outer layers provide implementations.

For example:

```csharp
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
    Task AddAsync(Product product);
}
```

The application depends on the abstraction:

```text
Application
     ↓
IProductRepository
```

Infrastructure provides the implementation:

```csharp
public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task<Product?> GetByIdAsync(int id)
    {
        return await _context.Products.FindAsync(id);
    }

    public async Task AddAsync(Product product)
    {
        await _context.Products.AddAsync(product);
    }
}
```

The dependency direction becomes:

```text
Application
     ↓
IProductRepository
     ↑
ProductRepository
     ↓
EF Core
```

The application knows the abstraction, while Infrastructure knows the implementation.

---

## 09 — Typical Project Structure

A Clean Architecture application can be organized like this:

```text
src/
│
├── Domain/
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Enums/
│   └── Exceptions/
│
├── Application/
│   ├── Interfaces/
│   ├── UseCases/
│   ├── DTOs/
│   └── Behaviors/
│
├── Infrastructure/
│   ├── Persistence/
│   ├── Identity/
│   ├── ExternalServices/
│   └── Repositories/
│
└── API/
    ├── Controllers/
    ├── Endpoints/
    └── Middleware/
```

A common dependency structure is:

```text
API
 ↓
Application
 ↓
Domain

Infrastructure
 ↓
Application
 ↓
Domain
```

The important point is that **Domain remains independent**.

---

## 10 — Request Flow

Consider:

```text
POST /products
```

A typical flow can be:

```text
HTTP Request
      ↓
Controller / Endpoint
      ↓
CreateProduct Use Case
      ↓
Product Entity
      ↓
IProductRepository
      ↓
ProductRepository
      ↓
EF Core
      ↓
Database
```

Notice that the use case does not need to know that the repository uses Entity Framework Core.

It only knows about the abstraction:

```text
IProductRepository
```

---

## 11 — Example

### Domain

```csharp
public class Product
{
    public int Id { get; private set; }
    public string Name { get; private set; }
    public decimal Price { get; private set; }

    public Product(string name, decimal price)
    {
        if (string.IsNullOrWhiteSpace(name))
            throw new ArgumentException();

        if (price <= 0)
            throw new ArgumentException();

        Name = name;
        Price = price;
    }
}
```

---

### Application

```csharp
public interface IProductRepository
{
    Task AddAsync(Product product);
}
```

Use case:

```csharp
public class CreateProduct
{
    private readonly IProductRepository _repository;

    public CreateProduct(IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task Execute(string name, decimal price)
    {
        var product = new Product(name, price);

        await _repository.AddAsync(product);
    }
}
```

---

### Infrastructure

```csharp
public class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task AddAsync(Product product)
    {
        await _context.Products.AddAsync(product);
        await _context.SaveChangesAsync();
    }
}
```

---

### API

```csharp
[ApiController]
[Route("products")]
public class ProductsController : ControllerBase
{
    private readonly CreateProduct _createProduct;

    public ProductsController(CreateProduct createProduct)
    {
        _createProduct = createProduct;
    }

    [HttpPost]
    public async Task<IActionResult> Create(CreateProductRequest request)
    {
        await _createProduct.Execute(request.Name, request.Price);

        return Ok();
    }
}
```

The flow is:

```text
API
 ↓
Application
 ↓
Domain

Infrastructure
     ↑
     │
Application Abstractions
```

---

## 12 — Independence from Frameworks

One of the main goals of Clean Architecture is keeping the core independent from frameworks.

For example, the domain should not require:

```csharp
using Microsoft.EntityFrameworkCore;
using Microsoft.AspNetCore.Mvc;
```

Business rules should be written without depending on infrastructure technologies.

This makes the core easier to test and change.

---

## 13 — Independence from Database

The application should not be tightly coupled to a specific database implementation.

For example:

```text
Application
     ↓
IProductRepository
```

Infrastructure can implement it using:

```text
SQL Server
```

or:

```text
PostgreSQL
```

or another persistence mechanism.

The use case does not need to change simply because the persistence technology changes.

---

## 14 — Independence from UI

The business logic should not depend on how users interact with the system.

The same application core could potentially be used by:

```text
REST API
   │
   ├── Web Application
   │
   ├── Mobile Application
   │
   └── Background Worker
```

The external interface can change while the core business logic remains largely independent.

---

## 15 — Testing

Clean Architecture makes it easier to test the core of the application independently.

For example:

```text
CreateProduct Use Case
        ↓
Mock IProductRepository
        ↓
Test Business Behavior
```

The test does not necessarily require:

* SQL Server
* Entity Framework Core
* HTTP
* ASP.NET Core

Example:

```csharp
[Fact]
public async Task Should_Create_Product()
{
    var repository = new FakeProductRepository();

    var useCase = new CreateProduct(repository);

    await useCase.Execute("Laptop", 1000);

    Assert.Single(repository.Products);
}
```

The important part is that the business behavior can be tested without the real infrastructure.

---

## 16 — Benefits

### Separation of Concerns

Each part of the application has a clear responsibility.

### Independent Business Logic

Core business rules are protected from infrastructure details.

### Testability

Use cases and domain logic can be tested without requiring external systems.

### Replaceable Infrastructure

Infrastructure implementations can be changed without rewriting the entire application core.

### Framework Independence

The core does not need to be tightly coupled to a particular framework.

### Maintainability

Clear boundaries make large applications easier to understand and modify.

---

## 17 — Common Problems

Clean Architecture can also be implemented poorly.

### Too Many Abstractions

Creating interfaces for everything can make a simple application unnecessarily complicated.

```text
IProductService
IProductManager
IProductHandler
IProductRepository
IProductProvider
IProductFactory
```

Not every class needs an abstraction.

---

### Unnecessary Layers

Adding layers without a real architectural reason increases complexity.

```text
Controller
 ↓
Service
 ↓
Manager
 ↓
Handler
 ↓
Processor
 ↓
Repository
```

The architecture should solve real problems rather than simply create more projects and folders.

---

### Anemic Domain

If entities contain only properties while all business rules exist elsewhere, the domain model may become weak.

```csharp
public class Product
{
    public string Name { get; set; }
    public decimal Price { get; set; }
}
```

Business behavior should be placed where it naturally belongs.

---

### Infrastructure Leakage

The core should not become dependent on infrastructure details.

For example:

```text
Domain
  ↓
EF Core
```

would violate the intended dependency direction.

---

## 18 — Key Characteristics

Clean Architecture focuses on:

```text
Business Rules
      ↓
Use Cases
      ↓
Abstractions
      ↓
External Implementations
```

The key characteristics are:

* Dependency Rule
* Dependency Inversion
* Independent business rules
* Use-case-oriented application logic
* Clear boundaries
* Replaceable infrastructure
* Testable core
* Framework independence

---

## 19 — Mental Model

Think of Clean Architecture as a protected core:

```text
┌───────────────────────────────────────────┐
│          Frameworks & Drivers              │
│                                           │
│  ┌─────────────────────────────────────┐  │
│  │       Interface Adapters            │  │
│  │                                     │  │
│  │  ┌───────────────────────────────┐  │  │
│  │  │          Use Cases            │  │  │
│  │  │                               │  │  │
│  │  │    ┌─────────────────────┐    │  │  │
│  │  │    │      Entities       │    │  │  │
│  │  │    │                     │    │  │  │
│  │  │    │   Business Rules    │    │  │  │
│  │  │    └─────────────────────┘    │  │  │
│  │  └───────────────────────────────┘  │  │
│  └─────────────────────────────────────┘  │
└───────────────────────────────────────────┘

        Dependencies → Inward
```

The closer something is to the center, the more important it is to the business and the less it should depend on external details.

---

## 20 — When to Use

Clean Architecture is a good fit when:

* The application has **complex business rules**.
* The project is expected to **grow and be maintained for a long time**.
* Business logic should be independent from frameworks and infrastructure.
* **Testability** is important.
* The application has multiple external interfaces or integrations.
* Clear dependency boundaries are important.

---

## 21 — When Not to Use

Clean Architecture may not be a good fit when:

* The application is **very small or simple**.
* The project is mainly straightforward CRUD.
* It is a **short-lived prototype**.
* The additional layers and abstractions would add unnecessary complexity.
* The team does not need strong architectural boundaries.

The goal is not to add Clean Architecture everywhere, but to use it when the **complexity of the system justifies the additional structure**.

---

## 22 — Summary

Clean Architecture organizes an application around its **business rules and use cases**, while keeping external technologies separate from the core.

The key idea is:

```text
Dependencies → Inward
```

A typical structure is:

```text
Domain
   ↑
Application
   ↑
Infrastructure / API
```

The goal is to keep the core business logic **independent, testable, and protected from external implementation details**.

