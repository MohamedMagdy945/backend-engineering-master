# Modular Monolith Architecture

A Modular Monolith is an application that is deployed as **one application**, but internally it is divided into **well-defined business modules**.

Each module represents a meaningful business capability and has clear boundaries.

For example, a restaurant platform might contain:

```text
Restaurants
Menus
Orders
Customers
Reviews
Payments
```

All of these modules run inside the same application and are usually deployed together.

The important idea is:

> **One deployable application, multiple well-separated business modules.**

A Modular Monolith is not simply a large application with many folders.

The modules should have **clear responsibilities, controlled dependencies, and limited knowledge of each other**.

---

## 01 — Overview

A traditional monolith may look like:

```text
                  Monolithic Application
                         │
       ┌─────────────────┼─────────────────┐
       │                 │                 │
   Restaurant          Menu              Order
      Code              Code               Code
       │                 │                 │
       └─────────────────┼─────────────────┘
                         │
                      Database
```

Everything exists in one application, but the boundaries between business areas may be weak.

A Modular Monolith creates explicit modules:

```text
                 ┌───────────────────────────┐
                 │      Modular Monolith     │
                 │                           │
                 │  ┌─────────┐ ┌─────────┐  │
                 │  │Restaurant│ │  Menu   │  │
                 │  └─────────┘ └─────────┘  │
                 │                           │
                 │  ┌─────────┐ ┌─────────┐  │
                 │  │  Order  │ │ Review  │  │
                 │  └─────────┘ └─────────┘  │
                 │                           │
                 └────────────┬──────────────┘
                              │
                          Database
```

The entire system is still one application.

But internally:

```text
Restaurant Module
       ≠
Menu Module
       ≠
Order Module
```

Each module owns its own business responsibilities.

---

## 02 — Core Idea

The architecture has three important concepts:

```text
Application
    │
    ├── Modules
    │
    ├── Module Boundaries
    │
    └── Controlled Communication
```

A module should have:

* Its own business logic
* Its own use cases
* Its own data access
* Its own domain concepts
* A clear public API
* Internal implementation details hidden from other modules

For example:

```text
Restaurants
│
├── Domain
├── Application
├── Infrastructure
└── Public API
```

Another module:

```text
Orders
│
├── Domain
├── Application
├── Infrastructure
└── Public API
```

The modules live in the same application, but they should behave as separate business components.

---

## 03 — What Is a Module?

A module represents a **business capability**.

For example:

```text
Restaurant Module
```

could be responsible for:

```text
Register Restaurant
Approve Restaurant
Update Restaurant
Suspend Restaurant
```

The:

```text
Order Module
```

could be responsible for:

```text
Place Order
Cancel Order
Accept Order
Reject Order
Complete Order
```

The important point is that a module is organized around a **business boundary**, not simply an entity.

Less useful:

```text
Entities/
    Restaurant
    Order
    Product
```

More meaningful:

```text
Restaurants/
    Register
    Approve
    Update

Orders/
    Place
    Cancel
    Accept
```

---

## 04 — Module Boundaries

A good module owns its internal implementation.

For example:

```text
Restaurant Module
│
├── Restaurant
├── RestaurantRepository
├── RestaurantValidator
├── RestaurantService
└── RestaurantDbContext
```

Another module should not directly access those internals.

For example, this is problematic:

```csharp
OrdersModule
    ↓
RestaurantsModule
    ↓
RestaurantRepository
```

because the Orders module is now aware of the Restaurant module's implementation.

Instead, the Restaurant module should expose something intentional.

For example:

```text
Orders Module
      ↓
Restaurant Module API
      ↓
Restaurant Module
```

This preserves the boundary.

---

## 05 — Public API of a Module

A module can expose a small public contract.

For example:

```csharp
public interface IRestaurantModule
{
    Task<bool> IsActiveAsync(
        int restaurantId,
        CancellationToken cancellationToken);
}
```

The Order module can use this contract:

```text
Order Module
     ↓
IRestaurantModule
```

without knowing:

```text
RestaurantRepository
EF Core
DbContext
Restaurant tables
```

This creates a controlled dependency.

---

## 06 — Internal vs Public Code

A useful mental model is:

```text
Restaurant Module
│
├── Public
│   └── IRestaurantModule
│
└── Internal
    ├── Restaurant
    ├── Repository
    ├── Handlers
    ├── Validators
    └── Database
```

Other modules should only use what the module intentionally exposes.

Conceptually:

```text
             ┌──────────────────────┐
             │ Restaurant Module    │
             │                      │
Other Module │  Public API          │
─────────────┼──────►               │
             │                      │
             │  Internal Code       │
             │  Hidden              │
             └──────────────────────┘
```

This is similar to **encapsulation**, but applied at the module level.

---

## 07 — Example Project Structure

A .NET Modular Monolith can be organized like:

```text
Menuhat/
│
├── src/
│   │
│   ├── Menuhat.Api/
│   │
│   ├── Modules/
│   │   │
│   │   ├── Restaurants/
│   │   │   ├── Domain/
│   │   │   ├── Application/
│   │   │   ├── Infrastructure/
│   │   │   └── RestaurantsModule.cs
│   │   │
│   │   ├── Menus/
│   │   │   ├── Domain/
│   │   │   ├── Application/
│   │   │   ├── Infrastructure/
│   │   │   └── MenusModule.cs
│   │   │
│   │   ├── Orders/
│   │   │   ├── Domain/
│   │   │   ├── Application/
│   │   │   ├── Infrastructure/
│   │   │   └── OrdersModule.cs
│   │   │
│   │   └── Reviews/
│   │       ├── Domain/
│   │       ├── Application/
│   │       ├── Infrastructure/
│   │       └── ReviewsModule.cs
│   │
│   └── Shared/
│
├── tests/
│   ├── Restaurants.Tests/
│   ├── Menus.Tests/
│   ├── Orders.Tests/
│   └── Reviews.Tests/
│
└── Menuhat.sln
```

The exact structure can vary.

The important part is that business capabilities have clear boundaries.

---

## 08 — Modular Monolith Request Flow

Suppose a customer places an order.

The request might flow like:

```text
HTTP Request
      ↓
API
      ↓
Order Module
      ↓
Place Order Use Case
      ↓
Restaurant Module
      ↓
Menu Module
      ↓
Order Module
      ↓
Database
```

For example:

```text
POST /api/orders
       ↓
Orders Module
       ↓
PlaceOrder
       ↓
Check Restaurant
       ↓
Check Menu Items
       ↓
Create Order
       ↓
Save Order
```

The important point is that the workflow crosses **business module boundaries**, but does so through controlled communication.

---

## 09 — Example: Restaurant Module

Suppose `Restaurants` owns restaurant availability.

```csharp
public interface IRestaurantModule
{
    Task<bool> IsActiveAsync(
        int restaurantId,
        CancellationToken cancellationToken);
}
```

The implementation remains internal:

```csharp
internal sealed class RestaurantModule
    : IRestaurantModule
{
    private readonly RestaurantDbContext _context;

    public RestaurantModule(
        RestaurantDbContext context)
    {
        _context = context;
    }

    public async Task<bool> IsActiveAsync(
        int restaurantId,
        CancellationToken cancellationToken)
    {
        return await _context.Restaurants
            .AnyAsync(
                x => x.Id == restaurantId &&
                     x.IsActive,
                cancellationToken);
    }
}
```

The Order module does not need to know how the Restaurant module determines whether a restaurant is active.

---

## 10 — Example: Order Module

The Order module can depend on the public contract:

```csharp
public sealed class PlaceOrderHandler
{
    private readonly IRestaurantModule _restaurants;

    public PlaceOrderHandler(
        IRestaurantModule restaurants)
    {
        _restaurants = restaurants;
    }

    public async Task Handle(
        int restaurantId,
        CancellationToken cancellationToken)
    {
        var isActive =
            await _restaurants.IsActiveAsync(
                restaurantId,
                cancellationToken);

        if (!isActive)
            throw new InvalidOperationException(
                "Restaurant is not active.");

        // Create order...
    }
}
```

The dependency is:

```text
Orders
  ↓
IRestaurantModule
  ↓
Restaurants
```

not:

```text
Orders
  ↓
RestaurantDbContext
  ↓
Restaurants table
```

The second approach destroys the module boundary.

---

## 11 — Module Communication

Modules can communicate in different ways.

### Direct Module API

One module calls another through an explicit contract.

```text
Orders
  ↓
Restaurants Public API
```

This is appropriate for operations that need an immediate result.

---

### Domain Events

A module can publish an event.

For example:

```text
Restaurant Approved
```

Then another module reacts to it:

```text
Restaurants
    ↓
RestaurantApproved
    ↓
Orders
```

or:

```text
Restaurants
    ↓
RestaurantApproved
    ↓
Notifications
```

This reduces direct coupling.

---

### Integration Events

For more decoupled communication, modules can communicate using messages.

```text
Restaurant Module
       ↓
Event
       ↓
Message Bus
       ↓
Notification Module
```

Inside a monolith, the messaging mechanism can still be in-process.

The important idea is the **boundary**, not whether RabbitMQ is physically involved.

---

## 12 — Shared Database

A Modular Monolith may use one physical database.

For example:

```text
                 SQL Server
                     │
       ┌─────────────┼─────────────┐
       │             │             │
 Restaurants       Menus         Orders
    Tables          Tables         Tables
```

However, sharing a database does **not** mean modules should freely query each other's tables.

Bad:

```text
Orders Module
     ↓
SELECT ...
FROM Restaurants
```

Better:

```text
Orders Module
     ↓
Restaurant Module API
     ↓
Restaurants Data
```

The database can be shared physically while ownership remains logically separated.

A stronger design can even use separate schemas:

```text
restaurants.Restaurants
menus.Items
orders.Orders
reviews.Reviews
```

This can make ownership clearer.

---

## 13 — Module Data Ownership

Every important piece of data should have a clear owner.

For example:

```text
Restaurant
     ↓
Restaurants Module

Menu
     ↓
Menus Module

Order
     ↓
Orders Module

Review
     ↓
Reviews Module
```

This gives a simple rule:

> **A module owns its data and the business rules around that data.**

Another module should not directly modify that data.

For example:

```text
Orders Module
```

should not directly change:

```text
Restaurant.IsActive
```

Instead:

```text
Orders
   ↓
Restaurants Module
   ↓
Restaurant.Approve()
```

or another explicitly exposed operation.

---

## 14 — Example for Menuhat

A Modular Monolith for `menuhat` could contain:

```text
Menuhat
│
├── Restaurants
│   ├── Register
│   ├── Approve
│   ├── Update
│   └── Suspend
│
├── Menus
│   ├── Create
│   ├── AddItem
│   ├── UpdateItem
│   └── RemoveItem
│
├── Orders
│   ├── Place
│   ├── Accept
│   ├── Reject
│   ├── Cancel
│   └── Complete
│
├── Reviews
│   ├── Create
│   ├── Update
│   └── Delete
│
└── Identity
    ├── Register
    ├── Login
    └── Authorization
```

This structure represents the actual business areas of the system.

---

## 18 — Advantages

### Single Deployment

The whole system can be deployed as one application.

```text
Menuhat
   ↓
One Deployment
```

This is much simpler than managing many microservices.

---

### Strong Business Boundaries

Modules make business responsibilities explicit.

```text
Restaurants
Orders
Menus
Reviews
```

Each area has a clear owner.

---

### Easier Development

Developers can work within one module without understanding the entire system.

```text
Orders Module
```

can be understood largely independently.

---

### Easier Testing

Modules can have focused tests:

```text
Restaurants.Tests
Orders.Tests
Menus.Tests
```

---

### Easier Future Extraction

A well-designed module can potentially become a separate service later.

For example:

```text
Modular Monolith

       Orders
          │
          ▼
      Eventually
          │
          ▼
Orders Microservice
```

This is often called an evolutionary approach.

However, a module is **not automatically a future microservice**.

It should first provide a meaningful business boundary.

---

### Simpler Operations

Compared with microservices, a Modular Monolith usually has:

```text
One Application
One Deployment
Simpler Local Development
Simpler Debugging
Simpler Monitoring
```

while still providing internal structure.

---

## 19 — Common Problems

### Fake Modules

A project may create folders:

```text
Restaurants/
Menus/
Orders/
```

but allow everything to access everything.

For example:

```text
Orders
   ↓
RestaurantDbContext
   ↓
Restaurants Tables
```

This creates the appearance of modularity without actual boundaries.

---

### Shared Database Access Everywhere

If every module can directly modify every table, the modules are tightly coupled.

For example:

```text
Orders
   ├── reads Restaurant tables
   ├── modifies Menu tables
   └── modifies Customer tables
```

This makes future changes difficult.

---

### Giant Shared Library

A large:

```text
Shared/
```

project can become a dependency dumping ground.

For example:

```text
Shared/
    RestaurantRepository
    OrderService
    MenuService
    CustomerService
    DatabaseHelpers
```

Eventually every module depends on Shared.

The result is effectively a hidden monolith inside the monolith.

Shared code should remain small and intentional.

---

### Circular Dependencies

For example:

```text
Orders
  ↓
Restaurants
  ↓
Orders
```

This makes module boundaries unclear.

Try to design communication so dependencies remain understandable.

Events can sometimes help break direct dependency cycles.

---

### Modules Based Only on Technical Concerns

This is not a useful modular design:

```text
Database Module
Logging Module
Validation Module
Controller Module
```

Business modules are usually more valuable:

```text
Restaurants
Menus
Orders
Reviews
```

The module should represent a meaningful business capability.

---

## 20 — When to Use

A Modular Monolith is a good fit when:

* The application is becoming large enough to need strong boundaries.
* The domain contains multiple business capabilities.
* You want simpler deployment than microservices.
* You want clear ownership of business logic and data.
* The team is not ready for the operational complexity of microservices.
* You want the possibility of extracting modules later.
* Different parts of the system have different business responsibilities.

It is especially suitable for:

```text
E-commerce
Food Platforms
ERP
SaaS
Booking Systems
Business Platforms
Financial Applications
```

---

## 21 — When Not to Use

A Modular Monolith may be unnecessary when:

* The application is very small.
* There is almost no business complexity.
* There is only one obvious business area.
* The project is a temporary prototype.
* Module boundaries would be artificial.

For example:

```text
Simple Todo API
```

probably does not need:

```text
Tasks Module
Users Module
Notifications Module
Analytics Module
```

unless those boundaries actually provide value.

Do not create modules just to make the architecture look sophisticated.

---

## 22 — Modular Monolith vs Traditional Monolith

### Traditional Monolith

```text
Application
│
├── Controllers
├── Services
├── Repositories
├── Entities
└── Database
```

The business boundaries can become unclear.

---

### Modular Monolith

```text
Application
│
├── Restaurants
│   ├── Domain
│   ├── Application
│   └── Infrastructure
│
├── Menus
│   ├── Domain
│   ├── Application
│   └── Infrastructure
│
└── Orders
    ├── Domain
    ├── Application
    └── Infrastructure
```

The application remains one deployable unit while business boundaries become explicit.

---

A Modular Monolith can avoid much of this complexity while still maintaining strong internal boundaries.

---

## 23 — Dependency Model

The goal is controlled dependencies between modules.

For example:

```text
Orders
   ↓
Restaurants Public API
```

and:

```text
Orders
   ↓
Menus Public API
```

but not:

```text
Orders
   ↓
Restaurant Internal Repository
```

or:

```text
Orders
   ↓
Restaurant DbContext
```

The module boundary should be respected.

Conceptually:

```text
┌───────────────────────┐
│      Restaurants      │
│                       │
│  Public API           │
│  ─────────────────    │
│  Internal Details     │
└──────────▲────────────┘
           │
           │ controlled
           │ communication
           │
┌──────────┴────────────┐
│        Orders         │
│                       │
│  Public API           │
│  Internal Details     │
└───────────────────────┘
```

---

## 24 — Practical .NET Flow

A production .NET Modular Monolith might look like:

```text
                    Menuhat.Api
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
   Restaurants         Menus            Orders
      Module           Module           Module
        │                │                │
        └────────────────┼────────────────┘
                         │
                  Shared Infrastructure
                         │
                      Database
```

A request such as:

```text
POST /api/orders
```

can enter the:

```text
Orders Module
```

which may interact with:

```text
Restaurants Module
Menus Module
```

through controlled contracts.

The application remains:

```text
One Process
One Deployment
Multiple Business Modules
```

---

## 25 — Summary

Modular Monolith Architecture organizes one deployable application into **independent business modules with clear boundaries**.

The basic idea is:

```text
One Application
      │
      ├── Restaurants
      ├── Menus
      ├── Orders
      └── Reviews
```

Each module owns:

```text
Business Logic
Use Cases
Data
Rules
Internal Implementation
```

Communication between modules should happen through:

```text
Public Contracts
Module APIs
Domain Events
Application Events
```

rather than direct access to another module's internal implementation.

The main principles are:

```text
Modules            → Business Boundaries
Module Ownership   → Clear Responsibility
Public API         → Controlled Communication
Internal Code      → Hidden Implementation
Data Ownership     → Module Responsibility
Deployment         → One Application
```

The main goal is to achieve the benefits of **modularity and clear business boundaries** without immediately taking on the operational complexity of microservices.

---

## 26 — Final Mental Model

Think of the architectures this way:

```text
Onion
→ How dependencies move

Hexagonal
→ How the core communicates with the outside world

Vertical Slice
→ How features are organized

Modular Monolith
→ How the whole application is divided into business modules
```

They can be combined.

For example, a strong `menuhat` architecture could look like:

```text
                    Menuhat
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
                 Onion / Hexagonal
                  Dependency Rules
                       │
                    Domain
                       │
                 Infrastructure
```

This is why these architectures should not be viewed simply as:

```text
Architecture A vs Architecture B
```

They often address **different dimensions of the same system**.
