# Vertical Slice Architecture — Project Tree & Structure

Vertical Slice Architecture (VSA) organizes software around **business features and capabilities** rather than technical layers. Every slice contains everything required to fulfill a specific user request from HTTP endpoint down to data storage.

---

## 01 — Architectural Concept: Horizontal Layers vs. Vertical Slices

```text
Horizontal Layers (Traditional)                      Vertical Slice Architecture
┌───────────────────────────────┐        ┌─────────────┬─────────────┬─────────────┐
│ Controllers / Presentation    │        │ CreateOrder │ CancelOrder │ ViewProduct │
├───────────────────────────────┤        │    Slice    │    Slice    │    Slice    │
│ Services / Use Cases          │   VS   ├─────────────┼─────────────┼─────────────┤
├───────────────────────────────┤        │  Endpoint   │  Endpoint   │  Endpoint   │
│ Domain Entities & Rules       │        │  Validation │  Validation │  Raw SQL    │
├───────────────────────────────┤        │  Domain Rule│  Domain Rule│  Response   │
│ Repositories / DbContext      │        │  EF Core DB │  EF Core DB │             │
└───────────────────────────────┘        └─────────────┴─────────────┴─────────────┘
```

---

## 02 — WHEN TO USE VERTICAL SLICE ARCHITECTURE (Deep Dive)

Deciding when to choose Vertical Slice Architecture instead of traditional horizontal layers (Clean or Onion) is driven by your project's development workflow and complexity distribution.

### 1. High Feature Velocity & Frequent Iteration
- **When**: New features, endpoints, and workflows are continuously added, modified, or decommissioned.
- **Why**: In Vertical Slice, each feature is completely self-contained in its own folder. Adding a feature means adding a folder; deleting a feature means deleting a folder. There is virtually zero risk of breaking unrelated features.

### 2. Heterogeneous (Asymmetric) Endpoint Complexity
- **When**: 80% of your system consists of straightforward data reading/writing (e.g., search, dropdowns, profile views), while 20% involves complex domain invariants (e.g., billing, order placement).
- **Why Clean Architecture Struggles**: Clean Architecture forces *every* endpoint through the same heavy ceremony (Controller ➔ Command ➔ Handler ➔ Domain Entity ➔ Repository ➔ DTO).
- **Why Vertical Slice Wins**: Each slice chooses its own optimal tool:
  - Simple query slices can query the database directly using Dapper or raw SQL in 25 lines of code.
  - Complex command slices can use a rich Domain Aggregate with full invariant validation and event publishing.

### 3. Multiple Parallel Developers (Zero Git Conflicts)
- **When**: Multiple engineers or cross-functional feature squads work simultaneously in the same backend codebase.
- **Why**: In layered architectures, developers constantly clash when editing shared files like `IOrderRepository.cs`, `OrderService.cs`, or `ApplicationDbContext.cs`. In Vertical Slice, each developer works exclusively inside their feature folder (`Features/Orders/CreateOrder/`), virtually eliminating merge conflicts.

### 4. Natural Path to Modular Monoliths & Microservices
- **When**: You are starting with a monolith today, but anticipate that specific modules or features might need to be extracted into independent microservices or cloud functions later.
- **Why**: Because all code, validation, mapping, and database logic for a feature are co-located in the feature slice, extraction is clean and straightforward.

### 5. High Locality of Reference
- **When**: Developers want to understand an entire use case without jumping across 5 different projects and 10 separate folders.

---

## 03 — When NOT to Use Vertical Slice Architecture

| Scenario | Why Vertical Slice May Struggle | Better Fit |
|---|---|---|
| **Highly Intertwined Domain Model** | If 50 operations all mutate the exact same core business invariants on a single aggregate, duplicating validation logic across 50 slices leads to inconsistencies. | Clean Architecture or Hybrid Model (VSA outside, Domain Model inside). |
| **Strict Framework Conventions** | Legacy platforms or frameworks that mandate strict global folders (`/controllers`, `/models`, `/views`). | Standard MVC / Layered. |
| **Teams that Cannot Tolerate Controlled Duplication** | Teams obsessed with DRY (Don't Repeat Yourself) at all costs will fight VSA by creating premature shared services, recreating horizontal layers inside feature folders. | Layered Architecture. |
| **Microservices with < 5 Endpoints** | A microservice with only 2 or 3 routes is already small enough that folder architecture provides little benefit. | Minimal APIs in a single file. |

---

## 04 — Architectural Decision Flowchart

```text
Do you have diverse features with different complexity levels?
                           │
             ┌─────────────┴─────────────┐
            YES                          NO
             │                           │
Does each feature change                 Is your domain an intricate web
independently at high velocity?          of shared invariant rules across all endpoints?
             │                           │
        ┌────┴────┐                 ┌────┴────┐
       YES        NO               YES        NO
        │          │                │          │
        ▼          ▼                ▼          ▼
   [Vertical   [Evaluate         [Clean /   [Standard
     Slice]    Team Size]         Onion]     3-Tier]
```

---

## 05 — Comprehensive Project Tree

```text
FoodDeliveryApp/
│
├── src/
│   ├── FoodDeliveryApp.Api/
│   │   ├── Features/                                   # Business Capabilities / Features
│   │   │   ├── Orders/
│   │   │   │   ├── PlaceOrder/
│   │   │   │   │   ├── PlaceOrderEndpoint.cs           # Minimal API Route / Controller
│   │   │   │   │   ├── PlaceOrderCommand.cs            # Request Contract
│   │   │   │   │   ├── PlaceOrderHandler.cs            # Use Case Execution & Persistence
│   │   │   │   │   ├── PlaceOrderValidator.cs          # Input Validation Rules
│   │   │   │   │   └── PlaceOrderResponse.cs           # Response DTO
│   │   │   │   │
│   │   │   │   ├── GetOrderById/
│   │   │   │   │   ├── GetOrderByIdEndpoint.cs
│   │   │   │   │   ├── GetOrderByIdQuery.cs
│   │   │   │   │   ├── GetOrderByIdHandler.cs          # Direct SQL / Dapper projection
│   │   │   │   │   └── OrderDetailsDto.cs
│   │   │   │   │
│   │   │   │   ├── CancelOrder/
│   │   │   │   │   ├── CancelOrderEndpoint.cs
│   │   │   │   │   ├── CancelOrderCommand.cs
│   │   │   │   │   └── CancelOrderHandler.cs
│   │   │   │   │
│   │   │   │   └── GetCustomerOrdersList/
│   │   │   │       ├── GetCustomerOrdersListEndpoint.cs
│   │   │   │       ├── GetCustomerOrdersListQuery.cs
│   │   │   │       └── CustomerOrdersListResponse.cs
│   │   │   │
│   │   │   ├── Restaurants/
│   │   │   │   ├── RegisterRestaurant/
│   │   │   │   │   ├── RegisterRestaurantEndpoint.cs
│   │   │   │   │   ├── RegisterRestaurantCommand.cs
│   │   │   │   │   ├── RegisterRestaurantHandler.cs
│   │   │   │   │   └── RegisterRestaurantValidator.cs
│   │   │   │   ├── SearchRestaurantsByLocation/
│   │   │   │   │   ├── SearchRestaurantsEndpoint.cs
│   │   │   │   │   ├── SearchRestaurantsQuery.cs
│   │   │   │   │   └── SearchRestaurantsHandler.cs
│   │   │   │   └── UpdateMenu/
│   │   │   │       ├── UpdateMenuEndpoint.cs
│   │   │   │       ├── UpdateMenuCommand.cs
│   │   │   │       └── UpdateMenuHandler.cs
│   │   │   │
│   │   │   ├── Customers/
│   │   │   │   ├── RegisterCustomer/
│   │   │   │   │   ├── RegisterCustomerEndpoint.cs
│   │   │   │   │   ├── RegisterCustomerCommand.cs
│   │   │   │   │   └── RegisterCustomerHandler.cs
│   │   │   │   └── UpdateCustomerProfile/
│   │   │   │       ├── UpdateProfileEndpoint.cs
│   │   │   │       ├── UpdateProfileCommand.cs
│   │   │   │       └── UpdateProfileHandler.cs
│   │   │   │
│   │   │   └── Payments/
│   │   │       ├── ProcessPayment/
│   │   │       │   ├── ProcessPaymentEndpoint.cs
│   │   │       │   ├── ProcessPaymentCommand.cs
│   │   │       │   └── ProcessPaymentHandler.cs
│   │   │       └── HandlePaymentWebhook/
│   │   │           ├── StripeWebhookEndpoint.cs
│   │   │           └── StripeWebhookHandler.cs
│   │   │
│   │   ├── Common/                                     # Shared Cross-Cutting Infrastructure ONLY
│   │   │   ├── Behaviors/                              # Request pipeline behaviors
│   │   │   │   ├── ValidationPipelineBehavior.cs
│   │   │   │   ├── LoggingPipelineBehavior.cs
│   │   │   │   └── PerformancePipelineBehavior.cs
│   │   │   ├── Database/                               # Central database definitions
│   │   │   │   ├── AppDbContext.cs
│   │   │   │   ├── EntityConfigurations/
│   │   │   │   └── Migrations/
│   │   │   ├── Middleware/
│   │   │   │   ├── GlobalExceptionHandlingMiddleware.cs
│   │   │   │   └── AuthenticationMiddleware.cs
│   │   │   └── Exceptions/
│   │   │       ├── NotFoundException.cs
│   │   │       └── ValidationException.cs
│   │   │
│   │   ├── appsettings.json
│   │   ├── appsettings.Development.json
│   │   └── Program.cs                                  # Service registration & endpoint discovery
│   │
│   └── tests/
│       ├── Features.Orders.Tests/                      # Tests mirror feature slices
│       │   ├── PlaceOrderTests.cs
│       │   ├── GetOrderByIdTests.cs
│       │   └── CancelOrderTests.cs
│       ├── Features.Restaurants.Tests/
│       │   ├── RegisterRestaurantTests.cs
│       │   └── SearchRestaurantsTests.cs
│       └── Features.Customers.Tests/
│           └── RegisterCustomerTests.cs
```

---

## 06 — Folder Responsibilities & Structure Rules

| Component | Responsibility | Rules |
|---|---|---|
| `Features/{Domain}/{Feature}/` | Contains the complete end-to-end implementation for one single use case. | Self-contained. Must not be referenced by neighboring feature slices. |
| `Endpoint.cs` | Receives HTTP request, handles route attributes, delegates to handler. | Focuses solely on transport and status codes. |
| `Command.cs / Query.cs` | Defines input parameters required to execute the use case. | Specific to this feature slice. |
| `Handler.cs` | Executes business logic, performs validation, talks directly to DB/storage. | Chooses whatever persistence mechanism is best for this specific slice. |
| `Validator.cs` | Validates input properties using FluentValidation. | Co-located with command. |
| `Common/` | Shared pipeline behaviors, DB context, and global middleware. | Strict rule: No business logic allowed in `Common/`. |
