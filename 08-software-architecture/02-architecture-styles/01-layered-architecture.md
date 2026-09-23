# Layered Architecture

Layered Architecture is a software architecture style that organizes an application into **horizontal layers**, where each layer has a specific responsibility.

Each layer focuses on a particular concern and communicates with the layers around it.

---

## 01 — Overview

The basic idea is to divide the application into layers:

```text
┌─────────────────────────────┐
│      Presentation Layer     │
├─────────────────────────────┤
│      Business Layer         │
├─────────────────────────────┤
│      Data Access Layer      │
├─────────────────────────────┤
│      Database / Storage     │
└─────────────────────────────┘
```

A request usually moves from the upper layers toward the lower layers.

```text
Client
  ↓
Presentation
  ↓
Business
  ↓
Data Access
  ↓
Database
```

---

## 02 — Main Layers

### Presentation Layer

The Presentation Layer is responsible for interacting with the outside world.

Examples:

* Controllers
* API Endpoints
* Views
* UI Components
* Request / Response Models

Its responsibilities include:

* Receiving requests
* Returning responses
* Handling presentation-related validation
* Calling the appropriate business logic

Example:

```text
HTTP Request
     ↓
Controller
     ↓
Business Layer
```

The Presentation Layer should not contain core business rules.

---

### Business Layer

The Business Layer contains the application's business logic and rules.

Examples:

* Services
* Business Rules
* Domain Operations
* Calculations
* Business Validation

For example:

```text
Create Order
     ↓
Check Customer
     ↓
Validate Products
     ↓
Calculate Total
     ↓
Create Order
```

The Business Layer answers:

> What should the application do?

---

### Data Access Layer

The Data Access Layer is responsible for communicating with the data source.

Examples:

* Repositories
* Entity Framework Core
* SQL Queries
* DbContext
* Database Mappings

Its responsibility is to handle operations such as:

```text
Get
Insert
Update
Delete
```

Example:

```text
Business Layer
      ↓
OrderRepository
      ↓
Database
```

The Business Layer should not need to know the details of how the database stores the data.

---

### Database / Storage Layer

This is where application data is physically stored.

Examples:

* SQL Server
* PostgreSQL
* MySQL
* MongoDB
* File Storage

The database is responsible for persistence rather than application business rules.

---

## 03 — Dependency Flow

A typical layered application follows a downward dependency flow:

```text
Presentation
      ↓
Business
      ↓
Data Access
      ↓
Database
```

For example:

```text
OrdersController
      ↓
OrderService
      ↓
OrderRepository
      ↓
SQL Server
```

The upper layer uses the services provided by the layer below it.

---

## 04 — Request Flow

Consider an API request for creating an order:

```text
Client
  │
  ▼
OrdersController
  │
  ▼
OrderService
  │
  ▼
OrderRepository
  │
  ▼
Database
```

The response follows the opposite direction:

```text
Database
   ▲
   │
OrderRepository
   ▲
   │
OrderService
   ▲
   │
OrdersController
   ▲
   │
Client
```

This gives the application a predictable request flow.

---

## 05 — Example Project Structure

A typical layered application can look like:

```text
MyApplication/
│
├── Presentation/
│   ├── Controllers/
│   ├── Endpoints/
│   └── Models/
│
├── Business/
│   ├── Services/
│   ├── Rules/
│   └── Models/
│
├── DataAccess/
│   ├── Repositories/
│   ├── DbContext/
│   └── Configurations/
│
└── Database/
```

Another common structure is:

```text
src/
├── API/
├── Application/
├── Domain/
└── Infrastructure/
```

The naming can change, but the main idea remains the same:

```text
Presentation
     ↓
Business
     ↓
Data Access
```

---

## 06 — Separation of Responsibilities

Each layer should have a clear responsibility.

| Layer        | Responsibility             |
| ------------ | -------------------------- |
| Presentation | Communication with clients |
| Business     | Business logic and rules   |
| Data Access  | Data persistence           |
| Database     | Data storage               |

For example:

```text
Presentation
    → "Receive the request"

Business
    → "Decide what should happen"

Data Access
    → "Read/write the data"

Database
    → "Store the data"
```

---

## 07 — Example

Suppose we have:

```text
POST /orders
```

The request could flow like this:

```text
POST /orders
     ↓
OrdersController
     ↓
OrderService
     ↓
OrderRepository
     ↓
Database
```

### Presentation

```csharp
[HttpPost]
public IActionResult Create(CreateOrderRequest request)
{
    var result = _orderService.Create(request);

    return Ok(result);
}
```

### Business

```csharp
public Order Create(CreateOrderRequest request)
{
    // Business rules

    var order = new Order();

    return order;
}
```

### Data Access

```csharp
public void Add(Order order)
{
    _dbContext.Orders.Add(order);
    _dbContext.SaveChanges();
}
```

Each part has a different responsibility.

---

## 08 — Advantages

### Separation of Concerns

Each layer has a defined responsibility.

### Simplicity

The structure is easy to understand and follow.

### Maintainability

Related technical responsibilities are grouped together.

### Testability

Business logic can be tested separately from presentation and persistence.

### Team Organization

Different developers can work on different parts of the application with clear boundaries.

### Predictable Flow

Developers can easily understand how a request moves through the application.

---

## 09 — Common Problems

Layered Architecture can become problematic when the boundaries are not respected.

### Business Logic in Controllers

Bad:

```csharp
[HttpPost]
public IActionResult Create(OrderRequest request)
{
    if (request.Quantity > 100)
    {
        // Business rule
    }

    // More business logic...
}
```

Controllers should primarily handle presentation concerns.

---

### Database Logic in Business Layer

The business layer should not become responsible for database implementation details.

Bad:

```csharp
public void CreateOrder()
{
    var connection = new SqlConnection(...);

    // SQL logic
}
```

Database access belongs to the Data Access Layer.

---

### Large Service Classes

A business service can become too large:

```text
OrderService
├── CreateOrder()
├── UpdateOrder()
├── DeleteOrder()
├── CancelOrder()
├── ApproveOrder()
├── RejectOrder()
├── CalculateTotal()
├── ApplyDiscount()
└── SendNotification()
```

When this happens, responsibilities may need to be separated.

---

## 10 — Layer Boundaries

A good layered architecture maintains clear boundaries.

```text
Presentation
     │
     ▼
Business
     │
     ▼
Data Access
     │
     ▼
Database
```

Avoid bypassing layers without a clear architectural reason.

For example:

```text
Controller ───────────────→ Database
```

This bypasses the Business and Data Access layers.

A more typical flow is:

```text
Controller
    ↓
Business
    ↓
Data Access
    ↓
Database
```

---

## 11 — Key Characteristics

Layered Architecture is characterized by:

* Horizontal layers
* Separation of concerns
* Defined responsibilities
* Layer-to-layer communication
* Predictable request flow
* Centralized technical responsibilities
* Controlled dependencies

The exact number of layers is not fixed.

An application can have:

```text
Presentation
Business
Data
```

or:

```text
Presentation
Application
Domain
Infrastructure
```

The important concept is **separating responsibilities into layers**.

---

## 12 — When to Use

Layered Architecture is commonly suitable when:

* The application has clear technical responsibilities.
* The team wants a simple and familiar structure.
* The application is small or medium-sized.
* The business logic is relatively straightforward.
* A predictable request flow is useful.

---

## 13 — Summary

Layered Architecture divides an application into horizontal layers, with each layer responsible for a specific concern.

The basic structure is:

```text
Presentation
      ↓
Business
      ↓
Data Access
      ↓
Database
```

The core idea is simple:

```text
Presentation → Communicate
Business     → Decide
Data Access  → Persist
Database     → Store
```

A well-designed layered architecture keeps these responsibilities separated and maintains clear boundaries between the layers.
