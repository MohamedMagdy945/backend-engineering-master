# Vertical Slice Architecture

Vertical Slice Architecture is a software architecture style that organizes an application around **features or business capabilities** rather than technical layers.

Instead of organizing the entire application into folders such as:

```text
Controllers
Services
Repositories
Entities
DTOs
```

Vertical Slice Architecture organizes the code by **what the application does**.

For example:

```text
Register Restaurant
Approve Restaurant
Create Menu
Place Order
Review Restaurant
```

Each feature contains the code needed to implement that particular use case.

The main idea is:

> **Organize code around business capabilities, not technical roles.**

---

## 01 — Overview

Traditional layered architecture often looks like:

```text
             Application
                  │
      ┌───────────┼───────────┐
      │           │           │
 Controllers   Services   Repositories
      │           │           │
      └───────────┼───────────┘
                  │
               Database
```

Vertical Slice Architecture changes the organization.

Instead of grouping similar technical components together, we group everything related to a feature together.

```text
                    Application
                         │
        ┌────────────────┼────────────────┐
        │                │                │
        ▼                ▼                ▼
  Register         Approve          Place Order
  Restaurant       Restaurant
        │                │                │
        │                │                │
   Handler          Handler          Handler
   Validator        Validator        Validator
   Repository       Repository       Repository
   DTOs             DTOs             DTOs
```

Each feature becomes a **vertical slice** through the application.

---

## 02 — Core Idea

The architecture has one main organizing principle:

```text
Feature
   │
   ├── Request
   ├── Handler
   ├── Validation
   ├── Data Access
   ├── Response
   └── Supporting Logic
```

Instead of:

```text
Controllers
Services
Repositories
Validators
DTOs
```

the code is grouped by:

```text
RegisterRestaurant
ApproveRestaurant
CreateMenu
PlaceOrder
ReviewRestaurant
```

The feature owns the code required for its behavior.

---

## 03 — Feature-Based Organization

Suppose we have a restaurant application.

A traditional structure could look like:

```text
Controllers/
    RestaurantsController.cs
    OrdersController.cs
    MenusController.cs

Services/
    RestaurantService.cs
    OrderService.cs
    MenuService.cs

Repositories/
    RestaurantRepository.cs
    OrderRepository.cs
    MenuRepository.cs

Validators/
    RestaurantValidator.cs
    OrderValidator.cs
```

As the project grows, one feature can become spread across many folders.

To understand:

```text
Approve Restaurant
```

a developer may need to jump between:

```text
Controller
Service
Validator
Repository
DTO
```

Vertical Slice Architecture keeps those pieces together.

```text
Features/
│
├── Restaurants/
│   ├── Register/
│   └── Approve/
│
├── Menus/
│   └── Create/
│
└── Orders/
    └── Place/
```

Now the code for a specific use case is concentrated in one place.

---

## 04 — A Vertical Slice

A slice represents one business capability.

For example:

```text
Approve Restaurant
```

could contain:

```text
ApproveRestaurant/
│
├── Endpoint.cs
├── Command.cs
├── Handler.cs
├── Validator.cs
└── Response.cs
```

Everything required by this use case stays close to the feature.

The flow becomes:

```text
HTTP Request
      ↓
Approve Restaurant Endpoint
      ↓
Approve Restaurant Handler
      ↓
Domain
      ↓
Database
      ↓
Response
```

The feature is effectively a vertical path through the system.

---

## 05 — Features Instead of Layers

The key difference is how the code is organized.

### Traditional Layered Architecture

```text
Controllers
Services
Repositories
Entities
DTOs
Validators
```

### Vertical Slice Architecture

```text
Features
│
├── RegisterRestaurant
│   ├── Endpoint
│   ├── Handler
│   ├── Validator
│   └── Response
│
├── ApproveRestaurant
│   ├── Endpoint
│   ├── Handler
│   └── Response
│
├── CreateMenu
│   ├── Endpoint
│   ├── Handler
│   └── Validator
│
└── PlaceOrder
    ├── Endpoint
    ├── Handler
    ├── Validator
    └── Response
```

The architecture follows the **business capabilities of the system**.

---

## 06 — Request / Command

A feature normally starts with an input model.

For example:

```csharp
public sealed record CreateProductCommand(
    string Name,
    decimal Price);
```

This represents the request needed for the use case.

The command belongs to the feature:

```text
CreateProduct/
    Command.cs
```

It does not need to become a shared DTO just because another feature might have similar properties.

The feature owns its input contract.

---

## 07 — Handler

The Handler performs the use case.

For example:

```csharp
public sealed class CreateProductHandler
{
    private readonly AppDbContext _context;

    public CreateProductHandler(AppDbContext context)
    {
        _context = context;
    }

    public async Task<int> Handle(
        CreateProductCommand command,
        CancellationToken cancellationToken)
    {
        var product = new Product(
            command.Name,
            command.Price);

        _context.Products.Add(product);

        await _context.SaveChangesAsync(
            cancellationToken);

        return product.Id;
    }
}
```

The handler contains the workflow for:

```text
Create Product
```

This is important:

> The Handler does not need to be a generic `ProductService`.

It exists because the **use case exists**.

---

## 08 — Validation

Validation can stay inside the feature.

For example:

```csharp
public sealed class CreateProductValidator
{
    public void Validate(CreateProductCommand command)
    {
        if (string.IsNullOrWhiteSpace(command.Name))
            throw new ValidationException(
                "Product name is required.");

        if (command.Price <= 0)
            throw new ValidationException(
                "Price must be greater than zero.");
    }
}
```

The feature becomes:

```text
CreateProduct/
│
├── Command.cs
├── Handler.cs
├── Validator.cs
└── Response.cs
```

The validation belongs to the use case instead of being placed in a global:

```text
Validators/
```

folder.

---

## 09 — Endpoint / Controller

A Vertical Slice can use:

```text
Controllers
Minimal APIs
Endpoints
GraphQL
gRPC
```

For example, using Minimal APIs:

```csharp
app.MapPost(
    "/api/products",
    async (
        CreateProductCommand command,
        CreateProductHandler handler,
        CancellationToken cancellationToken) =>
    {
        var productId = await handler.Handle(
            command,
            cancellationToken);

        return Results.Ok(productId);
    });
```

The endpoint is the entry point into the slice.

The request travels through one feature:

```text
HTTP
 ↓
CreateProduct
 ↓
Handler
 ↓
Domain / Data
 ↓
Response
```

---

## 10 — Complete Example

Suppose the application has a:

```text
Create Product
```

feature.

The slice could look like:

```text
CreateProduct/
│
├── Command.cs
├── Handler.cs
├── Validator.cs
├── Response.cs
└── Endpoint.cs
```

### Command

```csharp
public sealed record CreateProductCommand(
    string Name,
    decimal Price);
```

### Handler

```csharp
public sealed class CreateProductHandler
{
    private readonly AppDbContext _context;

    public CreateProductHandler(AppDbContext context)
    {
        _context = context;
    }

    public async Task<CreateProductResponse> Handle(
        CreateProductCommand command,
        CancellationToken cancellationToken)
    {
        var product = new Product(
            command.Name,
            command.Price);

        _context.Products.Add(product);

        await _context.SaveChangesAsync(
            cancellationToken);

        return new CreateProductResponse(
            product.Id);
    }
}
```

### Response

```csharp
public sealed record CreateProductResponse(
    int Id);
```

### Validator

```csharp
public sealed class CreateProductValidator
{
    public void Validate(CreateProductCommand command)
    {
        if (string.IsNullOrWhiteSpace(command.Name))
            throw new ValidationException(
                "Name is required.");

        if (command.Price <= 0)
            throw new ValidationException(
                "Price must be greater than zero.");
    }
}
```

Everything required for the use case lives together.

---

## 11 — Project Structure

A production .NET application using Vertical Slice Architecture could look like:

```text
Menuhat/
│
├── src/
│   │
│   ├── Menuhat.Api/
│   │   ├── Features/
│   │   │   ├── Restaurants/
│   │   │   │   ├── Register/
│   │   │   │   ├── Approve/
│   │   │   │   └── Get/
│   │   │   │
│   │   │   ├── Menus/
│   │   │   │   ├── Create/
│   │   │   │   └── Update/
│   │   │   │
│   │   │   └── Orders/
│   │   │       ├── Place/
│   │   │       ├── Manage/
│   │   │       └── Get/
│   │   │
│   │   ├── Infrastructure/
│   │   └── Program.cs
│   │
│   └── Menuhat.Domain/
│       ├── Restaurants/
│       ├── Menus/
│       └── Orders/
│
├── tests/
│   ├── Menuhat.UnitTests/
│   └── Menuhat.IntegrationTests/
│
├── Menuhat.sln
└── README.md
```

There is no requirement for a specific folder structure.

The important idea is:

```text
Organize around features.
```

not:

```text
Organize around technical categories.
```

---

## 12 — Feature Example for Menuhat

For a restaurant platform, the features might be:

```text
Features/
│
├── Restaurants/
│   ├── Register/
│   ├── Approve/
│   ├── Update/
│   ├── Get/
│   └── Delete/
│
├── Menus/
│   ├── Create/
│   ├── Update/
│   ├── AddItem/
│   └── RemoveItem/
│
├── Orders/
│   ├── Place/
│   ├── Cancel/
│   ├── Accept/
│   ├── Reject/
│   └── Complete/
│
└── Reviews/
    ├── Create/
    ├── Update/
    └── Delete/
```

This makes the business capabilities visible directly from the source tree.

For example:

```text
Orders/Place/
```

immediately tells you:

> This is the code responsible for placing an order.

---

## 13 — Feature Independence

One of the most useful properties of Vertical Slice Architecture is **feature independence**.

For example:

```text
Restaurants/Register
```

can have completely different validation and persistence requirements from:

```text
Restaurants/Approve
```

There is no requirement for both features to share one giant:

```text
RestaurantService
```

Instead:

```text
Register Restaurant
        ↓
Register Handler
```

and:

```text
Approve Restaurant
        ↓
Approve Handler
```

Each use case owns its own workflow.

---

## 14 — Shared Code

Vertical Slice Architecture does **not** mean:

> Never share code.

Shared code is appropriate when something is genuinely common.

For example:

```text
Shared/
├── Errors/
├── Authentication/
├── Pagination/
├── Result/
└── Behaviors/
```

But avoid creating shared abstractions simply because two features currently look similar.

For example, do not immediately create:

```text
CommonRestaurantService
```

just because both:

```text
RegisterRestaurant
ApproveRestaurant
```

work with a `Restaurant`.

The question should be:

> Is this behavior actually shared?

not:

> Can these pieces be technically reused?

---

## 15 — Database Access

A Vertical Slice does not require repositories everywhere.

A feature can directly use EF Core when that is appropriate.

For example:

```csharp
public sealed class ApproveRestaurantHandler
{
    private readonly AppDbContext _context;

    public ApproveRestaurantHandler(
        AppDbContext context)
    {
        _context = context;
    }

    public async Task Handle(
        int restaurantId,
        CancellationToken cancellationToken)
    {
        var restaurant =
            await _context.Restaurants
                .FirstOrDefaultAsync(
                    x => x.Id == restaurantId,
                    cancellationToken);

        if (restaurant is null)
            throw new NotFoundException();

        restaurant.Approve();

        await _context.SaveChangesAsync(
            cancellationToken);
    }
}
```

This can be perfectly valid.

You do not automatically need:

```text
Controller
    ↓
Service
    ↓
Repository
    ↓
DbContext
```

just because it is considered a common pattern.

The architecture should match the actual problem.

---

## 16 — Dependency Direction

Vertical Slice Architecture is primarily an **organizational and feature-oriented approach**.

It does not define one single dependency graph as strictly as Onion or Hexagonal Architecture.

For example, a slice could use:

```text
Endpoint
   ↓
Handler
   ↓
Domain
   ↓
EF Core
```

Or it could use:

```text
Endpoint
   ↓
Handler
   ↓
Application Abstraction
   ↓
Infrastructure Adapter
   ↓
Database
```

The exact dependency rules depend on the architecture you combine Vertical Slices with.

A common production design is:

```text
Vertical Slices
      +
Clean / Onion principles
```

For example:

```text
Features
   ↓
Application / Domain
   ↓
Infrastructure
```

while keeping the business logic independent from infrastructure.

---

## 17 — Testing

Testing can be organized by feature as well.

For example:

```text
Tests/
│
├── Restaurants/
│   ├── RegisterRestaurantTests.cs
│   └── ApproveRestaurantTests.cs
│
├── Menus/
│   └── CreateMenuTests.cs
│
└── Orders/
    ├── PlaceOrderTests.cs
    └── CancelOrderTests.cs
```

A test focuses on a use case:

```text
Place Order
```

rather than testing a generic:

```text
OrderService
```

This creates a close relationship between:

```text
Feature
    ↕
Tests
```

---

## 18 — Advantages

### Feature Cohesion

All code related to one business operation stays together.

### Easier Navigation

Developers can find the code for a feature without searching through many technical folders.

### Reduced Coupling

Features do not have to share large generic services.

### Easier Changes

A change to:

```text
Approve Restaurant
```

can often stay inside the:

```text
Approve/
```

slice.

### Clear Business Structure

The source code reflects what the system actually does.

For example:

```text
Register Restaurant
Approve Restaurant
Place Order
Accept Order
Review Restaurant
```

These are business concepts rather than technical concepts.

---

## 19 — Common Problems

### Giant Shared Services

One of the most common problems is creating a large service:

```csharp
public class RestaurantService
{
    // Register
    // Approve
    // Update
    // Delete
    // Search
    // Review
}
```

This can become a collection of unrelated use cases.

Vertical Slices encourage:

```text
Restaurants/
├── Register/
├── Approve/
├── Update/
├── Delete/
└── Search/
```

---

### Excessive Sharing

Developers may try to move everything into:

```text
Common/
Shared/
Utilities/
Helpers/
```

This can eventually recreate the same coupling that Vertical Slice Architecture was intended to reduce.

Shared code should represent **real shared concepts**.

---

### Duplicated Code

The opposite problem is also possible.

Two features may duplicate significant logic simply to avoid sharing anything.

For example:

```text
PlaceOrder/
    CalculateTotal()

Reorder/
    CalculateTotal()
```

If the calculation is genuinely the same business rule, it may belong in the Domain.

Vertical Slice Architecture does not mean eliminating all reuse.

---

### Confusing Feature with Entity

A slice should normally represent a **business operation or capability**, not simply an entity.

Less useful:

```text
Products/
Restaurants/
Orders/
```

More useful:

```text
Products/
    Create/
    Update/
    Archive/

Restaurants/
    Register/
    Approve/
    Suspend/

Orders/
    Place/
    Cancel/
    Accept/
```

The second structure expresses application behavior much more clearly.

---

## 20 — When to Use

Vertical Slice Architecture is a good fit when:

* The application has many business features.
* Different use cases have different requirements.
* The team wants feature-oriented code organization.
* The application is expected to grow.
* Developers frequently change individual features.
* Large shared services are becoming difficult to maintain.
* Business capabilities are more meaningful than technical layers.

It works especially well for applications such as:

```text
E-commerce
Food Ordering
Banking
Booking Systems
ERP
SaaS Platforms
Business Applications
```

where the system contains many independent workflows.

---

## 21 — When Not to Use

Vertical Slice Architecture may not be necessary when:

* The application is extremely small.
* There are only a few use cases.
* The project is a simple CRUD application.
* Feature boundaries are not meaningful.
* The additional organization would make the project harder to understand.

For example, a very small API with:

```text
Products CRUD
```

may not need an elaborate feature structure.

Do not introduce Vertical Slice Architecture simply because it is popular.

The architecture should solve an actual maintenance or organizational problem.

---

### Vertical Slice

Code is organized by business capability:

```text
CreateProduct
ApproveRestaurant
PlaceOrder
```

A request stays inside one feature:

```text
PlaceOrder
   ↓
Handler
   ↓
Domain
   ↓
Database
```

The feature owns its complete workflow.

---

## 25 — Practical .NET Flow

A production .NET application might have:

```text
Menuhat.Api
│
├── Features
│   │
│   ├── Restaurants
│   │   ├── Register
│   │   ├── Approve
│   │   └── Update
│   │
│   ├── Menus
│   │   ├── Create
│   │   └── AddItem
│   │
│   └── Orders
│       ├── Place
│       └── Cancel
│
├── Domain
│
└── Infrastructure
```

A request to approve a restaurant could flow like:

```text
POST /restaurants/{id}/approve
                 ↓
        ApproveRestaurant
                 ↓
        ApproveRestaurantHandler
                 ↓
             Restaurant
                 ↓
             Database
```

The important point is that the feature is easy to locate and understand.

---

## 26 — Summary

Vertical Slice Architecture organizes an application around **features and business capabilities** rather than technical layers.

The basic idea is:

```text
Feature
   │
   ├── Request
   ├── Handler
   ├── Validation
   ├── Business Logic
   ├── Data Access
   └── Response
```

Instead of:

```text
Controllers
Services
Repositories
DTOs
Validators
```

The main principles are:

```text
Features       → Business Capabilities
Slices         → Complete Use Cases
Handlers       → Execute the Use Case
Validation     → Feature-Specific Rules
Shared Code    → Only When Truly Shared
```

The main goal is to make the codebase reflect the **behavior of the business**, reduce unnecessary coupling between features, and make individual use cases easier to understand and change.

The most important distinction is:

```text
Onion
→ How dependencies are controlled

Hexagonal
→ How the core communicates with the outside world

Vertical Slice
→ How the application is organized around features
```

These architectures are not necessarily competitors.

They can be combined:

```text
              Vertical Slices
                     │
          ┌──────────┼──────────┐
          │          │          │
      Restaurant    Menu       Order
       Features    Features   Features
          │          │          │
          └──────────┼──────────┘
                     ↓
                  Domain
                     ↓
              Infrastructure
```

The result can be a feature-oriented application while still protecting the domain and controlling dependencies.
