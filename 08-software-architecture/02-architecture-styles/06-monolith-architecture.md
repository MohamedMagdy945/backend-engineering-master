# Monolithic Architecture

Monolithic Architecture is a software architecture style where the application is built and deployed as a **single unit**.

The application's functionality, business logic, data access, authentication, background processing, and external integrations can all exist inside the same application.

The application may still be divided into layers, features, or modules, but everything belongs to the same deployable system.

The main idea is:

```text
One Application
      ↓
One Deployable Unit
```

---

## 01 — Overview

A simple monolithic application can look like:

```text
                    ┌─────────────────────────┐
                    │     Monolithic App      │
                    │                         │
                    │  ┌───────────────────┐  │
                    │  │   Presentation    │  │
                    │  ├───────────────────┤  │
                    │  │    Application    │  │
                    │  ├───────────────────┤  │
                    │  │      Domain       │  │
                    │  ├───────────────────┤  │
                    │  │  Infrastructure   │  │
                    │  └───────────────────┘  │
                    │                         │
                    └────────────┬────────────┘
                                 │
                                 ▼
                              Database
```

Everything runs as one application.

For example:

```text
Client
   ↓
Menuhat API
   ↓
SQL Server
```

The application is deployed as one unit.

---

## 02 — Core Idea

A monolithic application can contain many responsibilities:

```text
Monolithic Application
        │
        ├── Presentation
        ├── Application Logic
        ├── Domain Logic
        ├── Data Access
        ├── Authentication
        ├── Background Jobs
        └── External Integrations
```

All of these are part of the same application.

For example:

```text
Menuhat
│
├── Restaurants
├── Menus
├── Orders
├── Customers
├── Reviews
├── Authentication
└── Payments
```

They can all run inside the same application process.

The defining characteristic is not the number of classes, folders, or layers.

It is:

```text
One Deployable Application
```

---

## 03 — Monolith Does Not Mean Bad Architecture

A common misunderstanding is:

> Monolithic means everything is mixed together.

That is not true.

A monolith can have a very clean internal architecture.

For example:

```text
Menuhat
│
├── Presentation
├── Application
├── Domain
└── Infrastructure
```

or:

```text
Menuhat
│
├── Restaurants
├── Menus
├── Orders
└── Reviews
```

Both can still be monolithic applications.

So:

```text
Monolith
≠
Messy Codebase
```

A monolith describes the **deployment model**, not the quality of the internal design.

---

## 04 — Traditional Layered Monolith

A common monolithic structure is layered:

```text
Presentation
      ↓
Application
      ↓
Domain
      ↓
Infrastructure
      ↓
Database
```

For example:

```text
HTTP Request
      ↓
Controller
      ↓
Application Service
      ↓
Domain
      ↓
Repository
      ↓
Database
```

Everything remains inside one application.

---

## 05 — Project Structure

A simple .NET monolith might look like:

```text
Menuhat/
│
├── Menuhat.Api/
│   ├── Controllers/
│   ├── Middleware/
│   └── Program.cs
│
├── Menuhat.Application/
│   ├── Services/
│   ├── DTOs/
│   ├── Validators/
│   └── Interfaces/
│
├── Menuhat.Domain/
│   ├── Entities/
│   ├── ValueObjects/
│   ├── Rules/
│   └── Exceptions/
│
├── Menuhat.Infrastructure/
│   ├── Persistence/
│   ├── Repositories/
│   ├── Identity/
│   └── ExternalServices/
│
└── tests/
    ├── Menuhat.UnitTests/
    └── Menuhat.IntegrationTests/
```

The projects are separate for organization and dependency control.

But they are still deployed as one application.

---

## 06 — Single Deployment

One of the main characteristics of a monolith is that the application is deployed as one unit.

For example:

```text
Source Code
     ↓
Build
     ↓
Menuhat.Api
     ↓
Docker Image
     ↓
Server
```

A change to one part of the system normally results in rebuilding and deploying the application.

For example:

```text
Change Reviews Feature
          ↓
     Build Menuhat
          ↓
    Deploy Menuhat
```

There is no requirement for Reviews to be deployed independently from Orders or Restaurants.

---

## 07 — Request Flow

Suppose a customer places an order.

A traditional monolithic request may look like:

```text
HTTP Request
      ↓
OrdersController
      ↓
OrderService
      ↓
OrderRepository
      ↓
EF Core
      ↓
SQL Server
      ↓
HTTP Response
```

The entire request is handled inside the same application.

There is no network call between these internal components.

---

## 08 — Example

Suppose we want to create a product.

### Request Model

```csharp
namespace Menuhat.Api.Products;

public sealed record CreateProductRequest(
    string Name,
    decimal Price);
```

---

### Controller

```csharp
using Microsoft.AspNetCore.Mvc;
using Menuhat.Application.Products;

namespace Menuhat.Api.Controllers;

[ApiController]
[Route("api/products")]
public sealed class ProductsController : ControllerBase
{
    private readonly IProductService _productService;

    public ProductsController(IProductService productService)
    {
        _productService = productService;
    }

    [HttpPost]
    public async Task<IActionResult> Create(
        CreateProductRequest request,
        CancellationToken cancellationToken)
    {
        var product = await _productService.CreateAsync(
            request.Name,
            request.Price,
            cancellationToken);

        return Ok(product);
    }
}
```

The controller is responsible for HTTP concerns.

---

### Application Service

```csharp
using Menuhat.Domain.Products;

namespace Menuhat.Application.Products;

public interface IProductService
{
    Task<Product> CreateAsync(
        string name,
        decimal price,
        CancellationToken cancellationToken);
}

public sealed class ProductService : IProductService
{
    private readonly IProductRepository _repository;

    public ProductService(IProductRepository repository)
    {
        _repository = repository;
    }

    public async Task<Product> CreateAsync(
        string name,
        decimal price,
        CancellationToken cancellationToken)
    {
        var product = new Product(name, price);

        await _repository.AddAsync(
            product,
            cancellationToken);

        return product;
    }
}
```

The Application Layer coordinates the use case.

---

### Domain Entity

```csharp
namespace Menuhat.Domain.Products;

public sealed class Product
{
    public int Id { get; private set; }

    public string Name { get; private set; }

    public decimal Price { get; private set; }

    public Product(
        string name,
        decimal price)
    {
        if (string.IsNullOrWhiteSpace(name))
            throw new ArgumentException(
                "Product name is required.",
                nameof(name));

        if (price <= 0)
            throw new ArgumentOutOfRangeException(
                nameof(price),
                "Price must be greater than zero.");

        Name = name;
        Price = price;
    }
}
```

The domain contains the business rule:

```text
Price must be greater than zero.
```

The entity does not know about:

```text
ASP.NET Core
EF Core
SQL Server
HTTP
```

---

### Repository Abstraction

```csharp
using Menuhat.Domain.Products;

namespace Menuhat.Application.Products;

public interface IProductRepository
{
    Task AddAsync(
        Product product,
        CancellationToken cancellationToken);
}
```

The Application Layer defines what it needs.

---

### Repository Implementation

```csharp
using Menuhat.Application.Products;
using Menuhat.Domain.Products;
using Menuhat.Infrastructure.Persistence;
using Microsoft.EntityFrameworkCore;

namespace Menuhat.Infrastructure.Repositories;

public sealed class ProductRepository : IProductRepository
{
    private readonly AppDbContext _context;

    public ProductRepository(AppDbContext context)
    {
        _context = context;
    }

    public async Task AddAsync(
        Product product,
        CancellationToken cancellationToken)
    {
        await _context.Products.AddAsync(
            product,
            cancellationToken);

        await _context.SaveChangesAsync(
            cancellationToken);
    }
}
```

Infrastructure contains the EF Core implementation.

---

## 09 — Complete Flow

The complete operation is:

```text
HTTP Request
      ↓
ProductsController
      ↓
ProductService
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

Everything still runs inside one application.

---

## 10 — Shared Database

A traditional monolith commonly uses one database.

For example:

```text
                    SQL Server
                        │
       ┌────────────────┼────────────────┐
       │                │                │
   Restaurants         Menus           Orders
      Tables           Tables           Tables
       │                │                │
       └────────────────┼────────────────┘
                        │
                    Customers
```

Different parts of the application can access the same database.

This is convenient, especially when operations require data from multiple areas.

However, unrestricted database access can create tight coupling.

For example:

```text
Orders
  ↓
Restaurants Table
  ↓
Menus Table
  ↓
Customers Table
```

When this happens everywhere, business boundaries become harder to maintain.

---

## 11 — Advantages

### Simple Deployment

The entire application is deployed as one unit.

```text
Build
  ↓
Deploy
  ↓
One Application
```

---

### Simple Local Development

A developer can usually start the complete system locally.

```text
dotnet run
```

There is no requirement to start many independent services.

---

### Simple Communication

Internal components can communicate with normal method calls.

```text
OrderService
     ↓
RestaurantService
```

There is no need for HTTP, gRPC, or message brokers just to communicate inside the same application.

---

### Simple Debugging

A developer can follow a request inside one process:

```text
Controller
   ↓
Service
   ↓
Repository
   ↓
Database
```

This is often easier than debugging a distributed system.

---

### Transaction Simplicity

When related operations use the same database, a normal database transaction can often cover them.

For example:

```text
Create Order
     ↓
Create Order Items
     ↓
Update Inventory
     ↓
Commit Transaction
```

This is generally simpler than coordinating distributed transactions across separate services.

---

### Lower Operational Complexity

A monolith usually requires fewer infrastructure components.

For example:

```text
Application
     ↓
Database
```

instead of:

```text
Service A
Service B
Service C
Message Broker
Multiple Databases
Service Discovery
Distributed Tracing
```

---

## 12 — Common Problems

### Large Codebase

As the system grows, the codebase can become difficult to navigate.

For example:

```text
Controllers/
Services/
Repositories/
Models/
Helpers/
Utils/
```

Developers may struggle to find all of the code related to a business feature.

---

### Tight Coupling

One part of the application may directly depend on another.

For example:

```text
Orders
   ↓
Restaurants
   ↓
Menus
   ↓
Customers
```

A small change can then have unexpected effects on multiple areas.

---

### Giant Services

A service can become responsible for too many operations.

```csharp
public sealed class RestaurantService
{
    // Register Restaurant
    // Update Restaurant
    // Approve Restaurant
    // Suspend Restaurant
    // Search Restaurant
    // Review Restaurant
    // Statistics
}
```

Eventually the class becomes difficult to understand and change.

---

### Shared Database Coupling

Different parts of the application may directly depend on each other's tables.

For example:

```text
OrderService
      ↓
Restaurants Table

ReviewService
      ↓
Customers Table

MenuService
      ↓
Restaurants Table
```

Database structure can then become tightly coupled to application behavior.

---

### Large Deployments

Even a small change may require deployment of the entire application.

```text
Change Review Feature
        ↓
Build Entire App
        ↓
Deploy Entire App
```

This becomes more noticeable as the application becomes larger.

---

## 13 — When to Use

Monolithic Architecture is a good fit when:

* The application is small or medium-sized.
* The team is small.
* The product is still evolving.
* Simple deployment is important.
* The domain boundaries are not yet fully understood.
* Distributed-system complexity would not provide enough value.
* Fast development is more important than independent deployment.

A monolith is often an excellent starting point for a new product.

---

## 14 — When Not to Use

A traditional monolith may become problematic when:

* Different parts of the system require independent deployment.
* Different components need very different scaling strategies.
* The codebase has become difficult to change safely.
* Teams need strong isolation between business areas.
* The entire application becomes a deployment bottleneck.
* The business requires independently operated systems.

Even in these situations, moving directly to microservices is not always the best answer.

A **Modular Monolith** may solve the architectural problem without introducing distributed-system complexity.

---

## 15 — Monolith vs Modular Monolith

### Traditional Monolith

The application may be organized mainly by technical layers:

```text
Application
│
├── Controllers
├── Services
├── Repositories
├── Models
└── Database
```

Business boundaries can become weak.

---

### Modular Monolith

The application is organized around business modules:

```text
Application
│
├── Restaurants
│
├── Menus
│
├── Orders
│
└── Reviews
```

Each module owns a specific business capability.

For example:

```text
Restaurants
├── Register
├── Approve
└── Update
```

while:

```text
Orders
├── Place
├── Accept
└── Cancel
```

Both are still one deployable application.

The main difference is the strength of the internal boundaries.

---

## 16 — Monolith vs Vertical Slice

These describe different aspects of a system.

### Monolith

Answers:

> How is the application deployed?

```text
One Deployable Application
```

### Vertical Slice

Answers:

> How is the application organized?

```text
Feature
   ↓
Use Case
```

For example:

```text
Menuhat
│
├── Restaurants
│   ├── Register
│   └── Approve
│
├── Menus
│   └── Create
│
└── Orders
    └── Place
```

This is still a monolith.

Therefore:

```text
Monolith
   +
Vertical Slice
```

is completely possible.

---

## 17 — Monolith vs Onion Architecture

These also solve different concerns.

### Monolith

Focuses on:

```text
Deployment
```

The application is one deployable unit.

### Onion Architecture

Focuses on:

```text
Dependency Direction
```

Dependencies move toward the Domain.

For example:

```text
Infrastructure
      ↓
Application
      ↓
Domain
```

Therefore:

```text
Monolith
   +
Onion Architecture
```

is also possible.

You can have one deployable application with strict internal dependency rules.

---

## 18 — Monolith vs Hexagonal Architecture

Again, they describe different dimensions.

### Monolith

```text
One Deployable Application
```

### Hexagonal

```text
Core
 ↓
Ports
 ↓
Adapters
```

A monolithic application can still use Hexagonal Architecture:

```text
                Web API
                   ↓
                Adapter
                   ↓
                Port
                   ↓
            Application Core
                   ↓
                Port
                   ↓
                Adapter
                   ↓
               Database
```

Everything can still run inside one application.

---

## 19 — Monolith vs Microservices

### Monolith

```text
                 One Application
                       │
       ┌───────────────┼───────────────┐
       │               │               │
   Restaurants        Menus          Orders
       │               │               │
       └───────────────┼───────────────┘
                       │
                    Database
```

### Microservices

```text
Restaurant Service
       │
   Database

Menu Service
       │
   Database

Order Service
       │
   Database
```

The major difference is the deployment boundary.

Monolith:

```text
One Deployment
```

Microservices:

```text
Multiple Independent Deployments
```

Microservices also introduce distributed-system concerns such as:

```text
Network Communication
Service Failures
Distributed Observability
Message Delivery
Service Deployment
Distributed Data
```

---

## 20 — Practical .NET Example

A well-structured monolithic `menuhat` application could look like:

```text
Menuhat/
│
├── Menuhat.Api/
│
├── Menuhat.Application/
│   ├── Restaurants/
│   ├── Menus/
│   ├── Orders/
│   └── Reviews/
│
├── Menuhat.Domain/
│
├── Menuhat.Infrastructure/
│
└── tests/
```

The deployment might simply be:

```text
Menuhat.Api
      ↓
Docker Container
      ↓
Server
```

The runtime flow is:

```text
Client
   ↓
Menuhat.Api
   ↓
Application
   ↓
Domain
   ↓
Infrastructure
   ↓
SQL Server
```

---

## 21 — Healthy Monolith

A monolith can have strong internal boundaries.

For example:

```text
                  Menuhat
                     │
        ┌────────────┼────────────┐
        │            │            │
    Restaurants     Menus        Orders
        │            │            │
        └────────────┼────────────┘
                     │
                   Domain
                     │
                Infrastructure
                     │
                  Database
```

The application is still one deployment, but the code is not necessarily tightly coupled.

A healthy monolith can use:

```text
Separation of Concerns
Dependency Inversion
Domain-Driven Design
Vertical Slices
Modules
Onion Architecture
Hexagonal Architecture
```

The word **monolith** does not prevent good architecture.

---

## 22 — Evolution of a Monolith

A system can evolve gradually.

For example:

```text
Simple Monolith
       ↓
Structured Monolith
       ↓
Modular Monolith
       ↓
Microservices
```

But this is not a required path.

A product may remain a monolith for its entire lifetime.

Another possible evolution is:

```text
Monolith
    ↓
Clear Business Boundaries
    ↓
Strong Modules
    ↓
Modular Monolith
```

Only split modules into separate services when independent deployment, scaling, ownership, or other real requirements justify it.

---

## 23 — Final Mental Model

The architectures you've documented answer different questions:

```text
Monolith
→ How is the application deployed?

Modular Monolith
→ How is one application divided into business modules?

Vertical Slice
→ How is code organized around features?

Onion
→ How should dependencies point?

Hexagonal
→ How does the core communicate with external systems?
```

They can be combined.

For example:

```text
                    Menuhat
                       │
                  Monolithic
                       │
                Modular Monolith
                       │
        ┌──────────────┼──────────────┐
        │              │              │
   Restaurants        Menus          Orders
        │              │              │
   Vertical         Vertical       Vertical
    Slices           Slices         Slices
        │              │              │
        └──────────────┼──────────────┘
                       │
                  Onion Rules
                       │
                    Domain
                       │
                 Infrastructure
```

This means:

```text
Monolith
```

does not mean:

```text
One Giant Class
One Giant Project
No Architecture
No Boundaries
```

It simply means:

```text
The system is deployed as one application.
```

The quality of the system depends on how well the responsibilities, boundaries, and dependencies inside that application are designed.

---

## 24 — Summary

Monolithic Architecture builds and deploys the application as **one unit**.

The basic model is:

```text
              One Application
                     │
       ┌─────────────┼─────────────┐
       │             │             │
 Presentation    Application     Domain
       │             │             │
       └─────────────┼─────────────┘
                     │
                Infrastructure
                     │
                  Database
```

The main principles are:

```text
One Application       → One Deployable Unit
In-Process Calls      → Simple Internal Communication
Shared Runtime        → Components Run Together
Simple Operations     → Fewer Distributed-System Concerns
Internal Architecture → Still Can Be Clean and Structured
```

The most important distinction is:

```text
Monolith
→ Deployment Boundary

Modular Monolith
→ Business Module Boundaries

Vertical Slice
→ Feature Organization

Onion
→ Dependency Direction

Hexagonal
→ Ports and Adapters
```

A monolith is not automatically a bad architectural choice.

For many applications, a **well-structured monolith is the simplest and most practical architecture**, especially when the business and system boundaries are still evolving.
