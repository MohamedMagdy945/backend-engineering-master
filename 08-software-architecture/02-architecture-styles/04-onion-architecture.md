# Onion Architecture

Onion Architecture is a software architecture style that organizes an application into **concentric layers**, with the **Domain Model at the center**.

The main principle is that **dependencies always point inward**.

The inner layers contain the most important business logic and should not depend on outer layers such as:

* Database
* Web API
* UI
* External services
* Frameworks
* Infrastructure

Onion Architecture is closely related to the **Dependency Inversion Principle** and is designed to keep the core of the application independent from implementation details.

---

## 01 — Overview

The basic idea is to place the domain at the center and surround it with additional layers.

```text
            ┌───────────────────────────────┐
            │        Infrastructure         │
            │                               │
            │   Database / External APIs    │
            │                               │
            ├───────────────────────────────┤
            │       Presentation            │
            │                               │
            │        Web API / UI           │
            │                               │
            ├───────────────────────────────┤
            │        Application            │
            │                               │
            │       Use Cases / Services    │
            │                               │
            ├───────────────────────────────┤
            │           Domain              │
            │                               │
            │   Entities / Rules / Logic    │
            │                               │
            └───────────────────────────────┘
```

A simpler representation is:

```text
        Infrastructure
              ↓
         Presentation
              ↓
         Application
              ↓
           Domain
```

The important rule is:

```text
Dependencies point inward.
```

---

## 02 — Core Idea

The architecture is usually organized into four main areas:

```text
Domain
   ↑
Application
   ↑
Presentation
   ↑
Infrastructure
```

The exact number of projects or layers can vary, but the main concept remains the same:

```text
                    DOMAIN
                      ▲
                      │
                 APPLICATION
                      ▲
                      │
                PRESENTATION
                      ▲
                      │
               INFRASTRUCTURE
```

Outer layers can depend on inner layers.

Inner layers should not depend on outer layers.

---

## 03 — Domain Layer

The **Domain Layer** is the center of the Onion.

It contains the most important business concepts and rules.

Examples:

* Entities
* Value Objects
* Domain Services
* Business Rules
* Domain Events
* Domain Exceptions

For example:

```csharp
public class Product
{
    public int Id { get; private set; }

    public string Name { get; private set; }

    public decimal Price { get; private set; }

    public Product(string name, decimal price)
    {
        if (string.IsNullOrWhiteSpace(name))
            throw new ArgumentException("Product name is required.");

        if (price <= 0)
            throw new ArgumentException("Price must be greater than zero.");

        Name = name;
        Price = price;
    }
}
```

The domain should not know about:

```text
EF Core
SQL Server
ASP.NET Core
HTTP
JSON
Controllers
Repositories implementations
External APIs
```

The domain represents the **business**, not the technology.

---

## 04 — Application Layer

The **Application Layer** contains the application's use cases.

It coordinates domain objects to perform a specific business operation.

Examples:

```text
Create Product
Update Product
Delete Product
Place Order
Approve Restaurant
Create Menu
Review Restaurant
```

The Application Layer answers:

> What does the application need to do?

It should not contain infrastructure implementation details.

Example:

```csharp
public class CreateProductService
{
    private readonly IProductRepository _repository;

    public CreateProductService(IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task ExecuteAsync(string name, decimal price)
    {
        var product = new Product(name, price);

        await _repository.AddAsync(product);
    }
}
```

The application uses an abstraction:

```text
IProductRepository
```

It does not care whether the implementation uses:

```text
EF Core
Dapper
MongoDB
HTTP
In-Memory Storage
```

---

## 05 — Repository Interfaces

A common pattern in Onion Architecture is to define repository abstractions inside an inner layer.

For example:

```csharp
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);

    Task AddAsync(Product product);
}
```

The Application Layer depends on the interface:

```text
Application
     ↓
IProductRepository
```

The actual implementation exists in Infrastructure:

```text
IProductRepository
        ▲
        │
ProductRepository
        │
        ▼
     EF Core
```

This is an example of **Dependency Inversion**.

The application defines what it needs.

Infrastructure provides the implementation.

---

## 06 — Presentation Layer

The **Presentation Layer** is responsible for communicating with clients.

Examples:

* ASP.NET Core Web API
* MVC
* Razor Pages
* GraphQL
* gRPC

For example:

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    private readonly CreateProductService _service;

    public ProductsController(CreateProductService service)
    {
        _service = service;
    }

    [HttpPost]
    public async Task<IActionResult> Create(CreateProductRequest request)
    {
        await _service.ExecuteAsync(
            request.Name,
            request.Price);

        return Ok();
    }
}
```

The controller should focus on presentation concerns:

```text
HTTP
Request
Response
Status Codes
Model Binding
Authentication Context
```

It should not contain important business rules.

The flow becomes:

```text
HTTP Request
      ↓
Controller
      ↓
Application Service
      ↓
Domain
```

---

## 07 — Infrastructure Layer

The **Infrastructure Layer** contains technical implementations.

Examples:

```text
EF Core
SQL Server
Email
File Storage
Payment Gateway
Redis
RabbitMQ
External APIs
Logging
```

For example:

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
        return await _context.Products
            .FirstOrDefaultAsync(x => x.Id == id);
    }

    public async Task AddAsync(Product product)
    {
        await _context.Products.AddAsync(product);
        await _context.SaveChangesAsync();
    }
}
```

Infrastructure knows about:

```text
EF Core
DbContext
SQL Server
```

But the Domain does not know any of them.

---

## 08 — Dependency Direction

This is the most important rule in Onion Architecture.

Dependencies must point toward the center.

```text
Infrastructure
       ↓
Presentation
       ↓
Application
       ↓
Domain
```

Or conceptually:

```text
┌───────────────────────────────┐
│        Infrastructure         │
│                               │
│   ┌───────────────────────┐   │
│   │     Presentation      │   │
│   │                       │   │
│   │  ┌─────────────────┐  │   │
│   │  │   Application   │  │   │
│   │  │                 │  │   │
│   │  │  ┌───────────┐  │  │   │
│   │  │  │  Domain   │  │  │   │
│   │  │  └───────────┘  │  │   │
│   │  └─────────────────┘  │   │
│   └───────────────────────┘   │
└───────────────────────────────┘
```

The Domain is completely isolated from the outer layers.

---

## 09 — Example Project Structure

A typical .NET Onion Architecture solution can look like:

```text
MyApplication/
│
├── src/
│   │
│   ├── MyApplication.Domain/
│   │   ├── Entities/
│   │   ├── ValueObjects/
│   │   ├── Services/
│   │   ├── Events/
│   │   └── Exceptions/
│   │
│   ├── MyApplication.Application/
│   │   ├── Interfaces/
│   │   ├── Services/
│   │   ├── Features/
│   │   ├── DTOs/
│   │   └── Behaviors/
│   │
│   ├── MyApplication.Infrastructure/
│   │   ├── Persistence/
│   │   ├── Repositories/
│   │   ├── Identity/
│   │   ├── Email/
│   │   └── ExternalServices/
│   │
│   └── MyApplication.Api/
│       ├── Controllers/
│       ├── Middleware/
│       ├── Filters/
│       └── Configuration/
│
├── tests/
│   ├── MyApplication.UnitTests/
│   └── MyApplication.IntegrationTests/
│
├── MyApplication.sln
└── README.md
```

The exact folder structure can vary.

The important part is the **dependency direction**.

---

## 10 — Project References

A typical .NET dependency structure is:

```text
                    ┌──────────────────────┐
                    │     MyApplication.Api │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │    Application       │
                    └──────────┬───────────┘
                               │
                               ▼
                    ┌──────────────────────┐
                    │       Domain         │
                    └──────────────────────┘
                               ▲
                               │
                    ┌──────────────────────┐
                    │   Infrastructure     │
                    └──────────────────────┘
```

A common project-reference setup is:

```text
Domain
   ↑
Application
   ↑
Api
   │
   └──────→ Infrastructure
```

More specifically:

```text
Application → Domain

Infrastructure → Application
Infrastructure → Domain

Api → Application
Api → Infrastructure
```

The Domain should have no reference to:

```text
Application
Infrastructure
Api
```

The Application should not reference:

```text
Infrastructure
Api
```

This protects the inner layers from technical details.

---

## 11 — Example

Suppose we want to create a product.

### Domain Entity

```csharp
public class Product
{
    public int Id { get; private set; }

    public string Name { get; private set; }

    public decimal Price { get; private set; }

    public Product(string name, decimal price)
    {
        if (string.IsNullOrWhiteSpace(name))
            throw new ArgumentException("Name is required.");

        if (price <= 0)
            throw new ArgumentException("Price must be greater than zero.");

        Name = name;
        Price = price;
    }
}
```

The Domain contains the business rule:

```text
Price must be greater than zero.
```

There is no EF Core code here.

---

### Application Interface

```csharp
public interface IProductRepository
{
    Task AddAsync(Product product);
}
```

The Application Layer defines the contract.

---

### Application Service

```csharp
public class CreateProductService
{
    private readonly IProductRepository _repository;

    public CreateProductService(IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task ExecuteAsync(
        string name,
        decimal price)
    {
        var product = new Product(name, price);

        await _repository.AddAsync(product);
    }
}
```

The Application coordinates the operation.

---

### Infrastructure Implementation

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

Infrastructure handles the database.

---

### API Controller

```csharp
[ApiController]
[Route("api/products")]
public class ProductsController : ControllerBase
{
    private readonly CreateProductService _service;

    public ProductsController(CreateProductService service)
    {
        _service = service;
    }

    [HttpPost]
    public async Task<IActionResult> Create(
        CreateProductRequest request)
    {
        await _service.ExecuteAsync(
            request.Name,
            request.Price);

        return Ok();
    }
}
```

The complete flow is:

```text
HTTP Request
      ↓
ProductsController
      ↓
CreateProductService
      ↓
Product
      ↓
IProductRepository
      ↓
ProductRepository
      ↓
EF Core
      ↓
SQL Server
```

---

## 12 — Testing

Onion Architecture makes the inner layers easy to test because they do not depend directly on infrastructure.

For example:

```text
CreateProductService
        ↓
IProductRepository
        ↑
FakeProductRepository
```

A unit test can use a fake repository:

```csharp
public class FakeProductRepository : IProductRepository
{
    public List<Product> Products { get; } = [];

    public Task AddAsync(Product product)
    {
        Products.Add(product);

        return Task.CompletedTask;
    }
}
```

Then test the application behavior without connecting to SQL Server.

```text
Unit Test
    ↓
Application
    ↓
Domain
    ↓
Fake Repository
```

Integration tests can separately verify:

```text
API
 ↓
Application
 ↓
Infrastructure
 ↓
Real Database
```

This gives a useful separation between:

```text
Business Tests
        +
Integration Tests
```

---

## 13 — Business Logic Location

One important question in Onion Architecture is:

> Where should the business logic go?

A useful rule is:

```text
Business Rule
     ↓
Domain

Use Case / Workflow
     ↓
Application
```

For example:

```text
"Product price cannot be negative"
            ↓
          Domain
```

But:

```text
"Create product, save it, then publish ProductCreated event"
            ↓
        Application
```

The Domain focuses on **business rules and behavior**.

The Application focuses on **orchestrating use cases**.

Infrastructure focuses on **technical implementation**.

Presentation focuses on **communication with clients**.

---

## 14 — Advantages

### Business Logic Protection

The most important business rules live in the center of the application.

### Clear Dependency Direction

Dependencies are controlled and always move toward the core.

### Testability

The Domain and Application layers can be tested without requiring external infrastructure.

### Infrastructure Independence

The application core does not depend directly on:

```text
SQL Server
EF Core
Redis
RabbitMQ
External APIs
```

### Easier Changes

Infrastructure can change without forcing major changes to business logic.

For example:

```text
SQL Server
     ↓
PostgreSQL
```

or:

```text
EF Core
     ↓
Dapper
```

The core business logic can remain mostly unchanged.

---

## 15 — Common Problems

### Too Many Layers

It is easy to create unnecessary projects and folders.

For example:

```text
Domain
Application
Application.Abstractions
Application.Contracts
Application.Services
Infrastructure
Infrastructure.Abstractions
Infrastructure.Persistence
Infrastructure.Repositories
```

This can create complexity without providing real value.

The architecture should create useful boundaries, not just more folders.

---

### Anemic Domain

A common mistake is putting all business logic inside Application Services:

```csharp
public class ProductService
{
    // 500 lines of business rules
}
```

while entities contain only properties:

```csharp
public class Product
{
    public string Name { get; set; }

    public decimal Price { get; set; }
}
```

This often results in a weak domain model.

Business rules that naturally belong to the entity should stay close to the entity.

---

### Infrastructure Leakage

Avoid exposing infrastructure details through inner-layer contracts.

Bad:

```csharp
Task<DbSet<Product>> GetProducts();
```

This leaks EF Core.

Better:

```csharp
Task<IReadOnlyList<Product>> GetProductsAsync();
```

The contract should describe an application need, not a framework implementation.

---

### Abstraction Everywhere

Not every class needs an interface.

Avoid creating interfaces such as:

```text
IProductService
IProductManager
IProductHelper
IProductFactory
IProductProcessor
IProductProvider
```

without a meaningful reason.

Use abstractions where they establish a useful boundary or where dependency inversion provides real value.

---

### Treating Onion as a Folder Structure

Onion Architecture is not simply:

```text
Create 4 folders
    ↓
Call it Onion Architecture
```

The important part is the dependency rule:

```text
Outer → Inner
```

not the names of the folders.

---

## 16 — Dependency Inversion

Onion Architecture strongly relies on the **Dependency Inversion Principle**.

The idea is:

```text
High-level business logic
          ↓
      Abstraction
          ↑
Low-level implementation
```

For example:

```text
Application
     │
     │ depends on
     ▼
IProductRepository
     ▲
     │ implemented by
     │
ProductRepository
     │
     ▼
 EF Core / SQL Server
```

The Application does not depend directly on the database technology.

Instead, both are connected through an abstraction.

---

## 17 — When to Use

Onion Architecture is a good fit when:

* The application contains important business rules.
* The domain should be protected from infrastructure concerns.
* The application is expected to grow over time.
* Testability is important.
* There are meaningful boundaries between business logic and technical infrastructure.
* The team wants clear dependency rules.

It is especially useful for systems such as:

```text
E-commerce
Food Ordering
Banking
ERP
Healthcare
Booking Systems
Business Platforms
```

where business rules are more important than simple CRUD operations.

---

## 18 — When Not to Use

Onion Architecture may not be a good fit when:

* The application is extremely small.
* The application is mostly simple CRUD.
* There is very little business logic.
* The project is a short-lived prototype.
* Adding multiple layers would make the code harder to understand rather than easier.

For a very small application, something like:

```text
API
 └── Services
      └── Database
```

may be perfectly reasonable.

Architecture should solve a problem, not create one.

---

## 19 — Onion Architecture vs Traditional Layered Architecture

Traditional layered architecture often looks like:

```text
Presentation
      ↓
Business Logic
      ↓
Data Access
      ↓
Database
```

The problem can appear when the business layer becomes tightly coupled to the data-access implementation.

Onion changes the dependency direction:

```text
        Infrastructure
               ↓
        Application
               ↓
            Domain
```

The business core becomes independent from infrastructure.

The database becomes an implementation detail rather than the foundation of the design.


---

## 20 — Practical .NET Flow

For a production ASP.NET Core application, a request may look like:

```text
Client
   ↓
Controller / Endpoint
   ↓
Application Use Case
   ↓
Domain Model
   ↓
Application Abstraction
   ↓
Infrastructure Implementation
   ↓
Database / External Service
```

For example:

```text
POST /api/restaurants
          ↓
RestaurantController
          ↓
RegisterRestaurantUseCase
          ↓
Restaurant
          ↓
IRestaurantRepository
          ↓
RestaurantRepository
          ↓
EF Core
          ↓
SQL Server
```

The important part is that:

```text
Restaurant
```

does not know that SQL Server exists.

And:

```text
RegisterRestaurantUseCase
```

does not need to know how EF Core saves the entity.

---

## 21 — Summary

Onion Architecture organizes the application into **concentric layers**, with the **Domain at the center**.

The main rule is:

```text
Dependencies point inward.
```

The responsibilities are roughly:

```text
Domain
→ Business Rules and Domain Model

Application
→ Use Cases and Application Workflows

Presentation
→ HTTP / UI / Client Communication

Infrastructure
→ Database and External Technology
```

The overall idea is:

```text
┌───────────────────────────────────────┐
│           Infrastructure              │
│                                       │
│   ┌───────────────────────────────┐   │
│   │         Presentation         │   │
│   │                               │   │
│   │   ┌───────────────────────┐   │   │
│   │   │      Application      │   │   │
│   │   │                       │   │   │
│   │   │   ┌───────────────┐   │   │   │
│   │   │   │    Domain     │   │   │   │
│   │   │   │               │   │   │   │
│   │   │   └───────────────┘   │   │   │
│   │   └───────────────────────┘   │   │
│   └───────────────────────────────┘   │
└───────────────────────────────────────┘
```

The core principles are:

```text
Domain        → Business Core
Application   → Use Cases
Presentation  → Entry Point
Infrastructure → Technical Implementations
Dependencies  → Always Point Inward
```

The main goal is to keep the **business core independent from technical infrastructure**, while creating clear boundaries that make the system easier to test, understand, and change.
