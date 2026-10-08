# Hexagonal Architecture

Hexagonal Architecture is a software architecture style that keeps the **application core independent from external systems**.

It organizes the application around the **business logic**, while external systems communicate with the core through defined interfaces called **Ports** and **Adapters**.

It is also known as **Ports and Adapters Architecture**.

---

## 01 — Overview

The basic idea is to put the application core in the center and connect external systems through ports and adapters.

```text
                 ┌───────────────┐
                 │   Web / API   │
                 └───────┬───────┘
                         │
                      Adapter
                         │
                         ▼
              ┌─────────────────────┐
              │                     │
              │   Application Core  │
              │                     │
              │ Business Logic      │
              │ Use Cases            │
              │                     │
              └─────────────────────┘
                         ▲
                         │
                      Adapter
                         │
                 ┌───────┴───────┐
                 │   Database    │
                 └───────────────┘
```

The application core does not depend directly on external technologies.

---

## 02 — Core Idea

The architecture has three main concepts:

```text
Application Core
      │
      ├── Ports
      │
      └── Adapters
```

### Application Core

Contains the important application logic.

Examples:

* Business Rules
* Use Cases
* Domain Models
* Application Services

The core should not depend on:

* HTTP
* Database
* UI
* External APIs
* Framework-specific infrastructure

---

### Ports

Ports define how the application communicates with the outside world.

A port is usually an **interface or contract**.

For example:

```csharp
public interface IProductRepository
{
    Task<Product?> GetByIdAsync(int id);
    Task AddAsync(Product product);
}
```

The application knows the port:

```text
Application Core
       ↓
IProductRepository
```

It does not need to know how the port is implemented.

---

### Adapters

Adapters implement or use ports to connect the application to external systems.

Examples:

```text
API Adapter
Database Adapter
Email Adapter
Payment Adapter
Message Queue Adapter
```

For example:

```text
IProductRepository
       ▲
       │
ProductRepository
       │
       ▼
   EF Core
```

The adapter translates between the external technology and the application's port.

---

## 03 — Ports and Adapters

There are two common types of ports.

### Driving Ports

Driving ports allow external systems to **use the application**.

Examples:

* REST API
* GraphQL
* CLI
* UI
* Message Consumer

```text
Client
   ↓
API Adapter
   ↓
Driving Port
   ↓
Application Core
```

---

### Driven Ports

Driven ports allow the application to **use external systems**.

Examples:

* Repository
* Email Service
* Payment Gateway
* Message Publisher
* File Storage

```text
Application Core
      ↓
Driven Port
      ↓
Database Adapter
      ↓
Database
```

---

## 04 — Architecture Flow

A typical request can flow through the architecture like this:

```text
HTTP Request
     ↓
API Adapter
     ↓
Input Port
     ↓
Application Core
     ↓
Output Port
     ↓
Database Adapter
     ↓
Database
```

The core remains independent from HTTP and the database.

---

## 05 — Example Project Structure

A Hexagonal Architecture project can be organized like:

```text
MyApplication/
│
├── Domain/
│   ├── Entities/
│   └── Rules/
│
├── Application/
│   ├── Ports/
│   │   ├── Input/
│   │   └── Output/
│   └── UseCases/
│
├── Adapters/
│   ├── In/
│   │   └── Web/
│   └── Out/
│       ├── Persistence/
│       └── ExternalServices/
│
└── Infrastructure/
```

The exact folder names can vary.

The important concept is:

```text
Core
 ↑
Ports
 ↑
Adapters
```

---

## 06 — Example

Suppose we want to create a product.

### Input Port

```csharp
public interface ICreateProduct
{
    Task ExecuteAsync(string name, decimal price);
}
```

The use case implements the port:

```csharp
public class CreateProduct : ICreateProduct
{
    private readonly IProductRepository _repository;

    public CreateProduct(IProductRepository repository)
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

---

### Output Port

```csharp
public interface IProductRepository
{
    Task AddAsync(Product product);
}
```

The application depends on the interface rather than a database implementation.

---

### Adapter

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

The repository is an adapter connecting the application to the database.

---

## 07 — Dependency Direction

The important dependency direction is toward the application core.

```text
API Adapter
     ↓
Input Port
     ↓
Application Core
     ↓
Output Port
     ↑
Database Adapter
```

The application defines the ports it needs.

External adapters implement those ports.

```text
Application
     │
     │ defines
     ▼
IProductRepository
     ▲
     │ implements
     │
ProductRepository
```

This keeps infrastructure details outside the core.

---

## 08 — External Systems

The same application core can communicate with different external systems through adapters.

For example:

```text
                 REST API
                    │
                 Adapter
                    │
                    ▼
             ┌─────────────┐
             │             │
             │ Application │
             │    Core     │
             │             │
             └─────────────┘
                    ▲
                    │
              Repository
                    │
             ┌──────┴──────┐
             │             │
          SQL Server    PostgreSQL
```

The core does not need to know which database is being used.

---

## 09 — Testing

Because the application core communicates through ports, external dependencies can be replaced during testing.

```text
Application Core
       ↓
IProductRepository
       ↑
FakeProductRepository
```

Instead of connecting to a real database, a test can use a fake implementation.

This allows business behavior to be tested independently.

---

## 10 — Advantages

### Independent Core

Business logic is isolated from external technologies.

### Replaceable Adapters

An adapter can be replaced without changing the application core.

### Testability

External dependencies can easily be replaced with test implementations.

### Clear Boundaries

Ports explicitly define how the application communicates with external systems.

### Technology Independence

The core does not need to know whether it is communicating with:

```text
SQL Server
PostgreSQL
REST API
RabbitMQ
Email Provider
```

---

## 11 — Common Problems

### Too Many Interfaces

Creating ports for every small operation can make the application unnecessarily complicated.

```text
IProductService
IProductRepository
IProductProvider
IProductManager
IProductHandler
```

Only create ports where a real boundary exists.

---

### Over-Abstraction

Not every external dependency needs a complicated abstraction.

The architecture should provide useful boundaries rather than simply increasing the number of interfaces.

---

### Incorrect Port Design

Ports should represent what the application needs, not expose unnecessary infrastructure details.

Bad:

```csharp
Task<DbSet<Product>> GetProducts();
```

This exposes Entity Framework details.

Better:

```csharp
Task<IReadOnlyList<Product>> GetProductsAsync();
```

The port should describe an application need rather than a technology.

---

## 12 — When to Use

Hexagonal Architecture is a good fit when:

* The application has **important business logic**.
* External systems may change over time.
* The application needs strong **testability**.
* There are multiple external integrations.
* Database or infrastructure independence is important.
* Clear boundaries between the core and external systems are valuable.

---

## 13 — When Not to Use

Hexagonal Architecture may not be a good fit when:

* The application is **very small and simple**.
* It mainly contains straightforward CRUD operations.
* The project is a short-lived prototype.
* There are very few external dependencies.
* The additional ports and adapters would add unnecessary complexity.

The goal is not to create ports and adapters everywhere, but to use them when they provide a **meaningful boundary**.

---

## 14 — Summary

Hexagonal Architecture places the **application core at the center** and connects it to external systems through **Ports and Adapters**.

The basic idea is:

```text
          Adapter
             ↓
          Input Port
             ↓
     ┌───────────────┐
     │ Application   │
     │     Core      │
     └───────────────┘
             ↓
         Output Port
             ↓
          Adapter
```

The core principles are:

```text
Application Core → Business Logic
Ports            → Communication Contracts
Adapters         → External Implementations
```

The main goal is to keep the **application core independent from external technologies** while providing clear and controlled communication with the outside world.
