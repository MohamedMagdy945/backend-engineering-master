# Clean Architecture — Project Tree & Structure

Clean Architecture organizes software into concentric layers where **enterprise policies and use cases** remain completely independent of UI, databases, frameworks, and external tools.

---

## 01 — Architectural Concept & Dependency Direction

```text
┌────────────────────────────────────────────────────────┐
│  Frameworks & Drivers (Web, DB, UI, External Devices)  │  ◄─ Outer Layer (Volatile Details)
│  ┌──────────────────────────────────────────────────┐  │
│  │   Interface Adapters (Controllers, Repos, DTOs)   │  │
│  │  ┌────────────────────────────────────────────┐  │  │
│  │  │    Application Business Rules (Use Cases)  │  │  │
│  │  │  ┌──────────────────────────────────────┐  │  │  │
│  │  │  │  Enterprise Business Rules (Entities)│  │  │  │  ◄─ Core (Center Policy)
│  │  │  └──────────────────────────────────────┘  │  │  │
│  │  └────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘

                     Dependencies: Inward Only (──►)
```

---

## 02 — Comprehensive Project Tree

```text
EnterpriseShop/
│
├── src/
│   ├── EnterpriseShop.Domain/                          # Core Layer: Enterprise Business Rules
│   │   ├── Common/
│   │   │   ├── BaseEntity.cs
│   │   │   ├── BaseAuditableEntity.cs
│   │   │   ├── ValueObject.cs
│   │   │   ├── IDomainEvent.cs
│   │   │   └── IAggregateRoot.cs
│   │   │
│   │   ├── Entities/
│   │   │   ├── Order.cs
│   │   │   ├── OrderItem.cs
│   │   │   ├── Customer.cs
│   │   │   ├── Product.cs
│   │   │   └── Invoice.cs
│   │   │
│   │   ├── ValueObjects/
│   │   │   ├── Money.cs
│   │   │   ├── Address.cs
│   │   │   ├── Email.cs
│   │   │   └── TrackingNumber.cs
│   │   │
│   │   ├── Enums/
│   │   │   ├── OrderStatus.cs
│   │   │   ├── PaymentStatus.cs
│   │   │   └── ShippingMethod.cs
│   │   │
│   │   ├── Events/
│   │   │   ├── OrderCreatedEvent.cs
│   │   │   ├── OrderPaidEvent.cs
│   │   │   └── ProductStockDepletedEvent.cs
│   │   │
│   │   └── Exceptions/
│   │       ├── DomainException.cs
│   │       ├── OrderCannotBeCancelledException.cs
│   │       └── InvalidPriceException.cs
│   │
│   ├── EnterpriseShop.Application/                     # Middle Layer: Application Business Rules & Use Cases
│   │   ├── Common/
│   │   │   ├── Interfaces/                             # Abstractions required by use cases
│   │   │   │   ├── IApplicationDbContext.cs
│   │   │   │   ├── IOrderRepository.cs
│   │   │   │   ├── ICustomerRepository.cs
│   │   │   │   ├── ICurrentUserService.cs
│   │   │   │   ├── IDateTimeProvider.cs
│   │   │   │   ├── IEmailService.cs
│   │   │   │   └── IFileStorageService.cs
│   │   │   ├── Behaviors/                              # Pipeline cross-cutting behaviors
│   │   │   │   ├── LoggingBehavior.cs
│   │   │   │   ├── ValidationBehavior.cs
│   │   │   │   ├── PerformanceBehavior.cs
│   │   │   │   └── TransactionBehavior.cs
│   │   │   ├── Models/
│   │   │   │   ├── Result.cs
│   │   │   │   └── PaginatedList.cs
│   │   │   └── Mappings/
│   │   │       └── OrderMappingProfile.cs
│   │   │
│   │   └── Orders/                                     # CQRS Feature Folders
│   │       ├── Commands/
│   │       │   ├── CreateOrder/
│   │       │   │   ├── CreateOrderCommand.cs
│   │       │   │   ├── CreateOrderCommandHandler.cs
│   │       │   │   └── CreateOrderCommandValidator.cs
│   │       │   ├── CancelOrder/
│   │       │   │   ├── CancelOrderCommand.cs
│   │       │   │   ├── CancelOrderCommandHandler.cs
│   │       │   │   └── CancelOrderCommandValidator.cs
│   │       │   └── MarkOrderAsPaid/
│   │       │       ├── MarkOrderAsPaidCommand.cs
│   │       │       └── MarkOrderAsPaidCommandHandler.cs
│   │       │
│   │       └── Queries/
│   │           ├── GetOrderById/
│   │           │   ├── GetOrderByIdQuery.cs
│   │           │   ├── GetOrderByIdQueryHandler.cs
│   │           │   └── OrderDetailsDto.cs
│   │           └── GetOrdersWithPagination/
│   │               ├── GetOrdersWithPaginationQuery.cs
│   │               ├── GetOrdersWithPaginationQueryHandler.cs
│   │               └── OrderBriefDto.cs
│   │
│   ├── EnterpriseShop.Infrastructure/                  # Outer Layer: Technical Implementation of Abstractions
│   │   ├── Persistence/
│   │   │   ├── ApplicationDbContext.cs
│   │   │   ├── Configurations/                         # Fluent API entity mappings
│   │   │   │   ├── OrderConfiguration.cs
│   │   │   │   ├── OrderItemConfiguration.cs
│   │   │   │   └── CustomerConfiguration.cs
│   │   │   ├── Repositories/
│   │   │   │   ├── OrderRepository.cs
│   │   │   │   └── CustomerRepository.cs
│   │   │   ├── Interceptors/                           # EF Core SaveChanges audit interceptors
│   │   │   │   └── AuditableEntitySaveChangesInterceptor.cs
│   │   │   └── Migrations/
│   │   │
│   │   ├── ExternalServices/
│   │   │   ├── SendGridEmailService.cs
│   │   │   ├── StripePaymentGateway.cs
│   │   │   └── AzureBlobStorageService.cs
│   │   │
│   │   ├── Identity/
│   │   │   ├── ApplicationUser.cs
│   │   │   └── IdentityService.cs
│   │   │
│   │   └── DependencyInjection.cs                      # Registers infrastructure services
│   │
│   └── EnterpriseShop.Api/                             # Outer Layer: Presentation, HTTP & Web Host
│       ├── Controllers/
│       │   ├── OrdersController.cs
│       │   ├── ProductsController.cs
│       │   └── CustomersController.cs
│       │
│       ├── Middleware/
│       │   ├── ExceptionHandlingMiddleware.cs
│       │   └── RequestTimeLoggingMiddleware.cs
│       │
│       ├── Filters/
│       │   └── ApiExceptionFilterAttribute.cs
│       │
│       ├── Configurations/
│       │   └── SwaggerConfiguration.cs
│       │
│       ├── appsettings.json
│       ├── appsettings.Development.json
│       └── Program.cs                                  # Application entry point & Composition Root
│
└── tests/
    ├── EnterpriseShop.Domain.UnitTests/
    │   └── Entities/
    │       └── OrderTests.cs
    │
    ├── EnterpriseShop.Application.UnitTests/
    │   └── Orders/
    │       └── Commands/
    │           └── CreateOrderCommandHandlerTests.cs
    │
    ├── EnterpriseShop.Infrastructure.IntegrationTests/
    │   └── Repositories/
    │       └── OrderRepositoryTests.cs
    │
    └── EnterpriseShop.Api.FunctionalTests/
        └── Endpoints/
            └── OrdersEndpointTests.cs
```

---

## 03 — Folder Responsibilities & Project Matrix

| Project | Clean Layer | Main Responsibility | What Must NEVER Be Included |
|---|---|---|---|
| `Domain` | Enterprise Rules | Entities, Value Objects, Domain Events, pure business invariants. | EF Core, SQL, Controllers, HTTP references. |
| `Application` | Use Cases | CQRS Commands/Queries, Handlers, DTOs, interfaces for repositories. | Direct DB connections, concrete HTTP contexts. |
| `Infrastructure` | Frameworks & Drivers | EF Core DbContext, repository implementations, third-party APIs. | Core business rules and domain logic. |
| `Api` | Presentation | Exposes endpoints via HTTP, parses requests, returns HTTP status codes. | Direct database queries, business rule processing. |

---

## 04 — The Dependency Inversion Principle

- The `Application` layer defines `IOrderRepository`.
- The `Infrastructure` layer implements `OrderRepository`.
- The `Domain` layer knows nothing about either `Application` or `Infrastructure`.
- At runtime, the dependency injection container in `Api/Program.cs` resolves `IOrderRepository` to `OrderRepository`.
- Source code dependencies point inward; runtime control flows outward through abstractions.
