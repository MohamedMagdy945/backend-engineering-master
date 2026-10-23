# Onion Architecture — Project Tree & Structure

Onion Architecture places the **Domain Model at the absolute center**, surrounded by concentric rings of business rules, application services, and external adapters.

---

## 01 — Architectural Concept & Dependency Flow

```text
┌────────────────────────────────────────────────────────┐
│                   Infrastructure / UI                  │  ◄─ External Ring (Volatile)
│  ┌──────────────────────────────────────────────────┐  │   (Database, HTTP, External APIs)
│  │               Application Services               │  │  ◄─ Layer 3
│  │  ┌────────────────────────────────────────────┐  │  │   (Use Cases, Workflows, Interfaces)
│  │  │              Domain Services               │  │  │  ◄─ Layer 2
│  │  │  ┌──────────────────────────────────────┐  │  │  │   (Cross-Entity Business Rules)
│  │  │  │             Domain Model             │  │  │  │  ◄─ Core (Center)
│  │  │  │       (Entities, Value Objects)      │  │  │  │   (Enterprise Concepts)
│  │  │  └──────────────────────────────────────┘  │  │  │
│  │  └────────────────────────────────────────────┘  │  │
│  └──────────────────────────────────────────────────┘  │
└────────────────────────────────────────────────────────┘

                     Dependencies: Inward Only (──►)
```

---

## 02 — Comprehensive Project Tree

```text
ECommerceSystem/
│
├── src/
│   ├── ECommerceSystem.Domain/                         # Core Ring 1 & Ring 2: Pure Business Logic
│   │   ├── Common/
│   │   │   ├── BaseEntity.cs
│   │   │   ├── BaseAuditableEntity.cs
│   │   │   ├── IAggregateRoot.cs
│   │   │   ├── ValueObject.cs
│   │   │   └── IDomainEvent.cs
│   │   │
│   │   ├── Entities/
│   │   │   ├── Customer.cs
│   │   │   ├── Order.cs
│   │   │   ├── OrderItem.cs
│   │   │   ├── Product.cs
│   │   │   ├── Category.cs
│   │   │   └── Payment.cs
│   │   │
│   │   ├── ValueObjects/
│   │   │   ├── Address.cs
│   │   │   ├── Money.cs
│   │   │   ├── EmailAddress.cs
│   │   │   └── PhoneNumber.cs
│   │   │
│   │   ├── Enums/
│   │   │   ├── OrderStatus.cs
│   │   │   ├── PaymentMethod.cs
│   │   │   └── Currency.cs
│   │   │
│   │   ├── DomainServices/
│   │   │   ├── IPricingDomainService.cs
│   │   │   ├── PricingDomainService.cs
│   │   │   ├── IInventoryAllocationService.cs
│   │   │   └── InventoryAllocationService.cs
│   │   │
│   │   ├── Events/
│   │   │   ├── OrderPlacedDomainEvent.cs
│   │   │   ├── OrderCancelledDomainEvent.cs
│   │   │   └── PaymentReceivedDomainEvent.cs
│   │   │
│   │   └── Exceptions/
│   │       ├── DomainException.cs
│   │       ├── InsufficientStockException.cs
│   │       └── InvalidOrderStateException.cs
│   │
│   ├── ECommerceSystem.Application/                    # Ring 3: Orchestration & Core Interfaces
│   │   ├── Common/
│   │   │   ├── Interfaces/
│   │   │   │   ├── IApplicationDbContext.cs
│   │   │   │   ├── IUnitOfWork.cs
│   │   │   │   ├── IDateTimeProvider.cs
│   │   │   │   └── ICurrentUserService.cs
│   │   │   ├── Models/
│   │   │   │   ├── PaginatedList.cs
│   │   │   │   └── Result.cs
│   │   │   └── Exceptions/
│   │   │       ├── NotFoundException.cs
│   │   │       └── ValidationException.cs
│   │   │
│   │   ├── Repositories/                               # Core interfaces implemented by Infrastructure
│   │   │   ├── ICustomerRepository.cs
│   │   │   ├── IOrderRepository.cs
│   │   │   ├── IProductRepository.cs
│   │   │   └── IPaymentRepository.cs
│   │   │
│   │   ├── ExternalServices/                           # Contracts for third-party systems
│   │   │   ├── IEmailNotificationService.cs
│   │   │   ├── IPaymentGatewayService.cs
│   │   │   └── ISmsNotificationService.cs
│   │   │
│   │   ├── Services/                                   # Application Services (Workflows)
│   │   │   ├── CustomerApplicationService.cs
│   │   │   ├── OrderApplicationService.cs
│   │   │   ├── ProductCatalogApplicationService.cs
│   │   │   └── CheckoutApplicationService.cs
│   │   │
│   │   ├── DTOs/
│   │   │   ├── Orders/
│   │   │   │   ├── CreateOrderRequestDto.cs
│   │   │   │   ├── OrderDetailsResponseDto.cs
│   │   │   │   ├── OrderSummaryDto.cs
│   │   │   │   └── OrderItemDto.cs
│   │   │   ├── Customers/
│   │   │   │   ├── CustomerProfileDto.cs
│   │   │   │   └── RegisterCustomerDto.cs
│   │   │   └── Products/
│   │   │       ├── ProductListDto.cs
│   │   │       └── ProductFilterDto.cs
│   │   │
│   │   └── Validators/
│   │       ├── CreateOrderRequestValidator.cs
│   │       └── RegisterCustomerValidator.cs
│   │
│   ├── ECommerceSystem.Infrastructure/                 # Ring 4: External Implementation Details
│   │   ├── Persistence/
│   │   │   ├── ApplicationDbContext.cs
│   │   │   ├── UnitOfWork.cs
│   │   │   ├── Configurations/                         # EF Core Entity Type Configurations
│   │   │   │   ├── CustomerConfiguration.cs
│   │   │   │   ├── OrderConfiguration.cs
│   │   │   │   ├── OrderItemConfiguration.cs
│   │   │   │   └── ProductConfiguration.cs
│   │   │   ├── Repositories/                           # Concrete implementations of Application interfaces
│   │   │   │   ├── CustomerRepository.cs
│   │   │   │   ├── OrderRepository.cs
│   │   │   │   ├── ProductRepository.cs
│   │   │   │   └── PaymentRepository.cs
│   │   │   └── Migrations/
│   │   │       └── 20261023_InitialCreate.cs
│   │   │
│   │   ├── ExternalServices/
│   │   │   ├── SendGridEmailNotificationService.cs
│   │   │   ├── StripePaymentGatewayService.cs
│   │   │   └── TwilioSmsNotificationService.cs
│   │   │
│   │   ├── Services/
│   │   │   ├── SystemDateTimeProvider.cs
│   │   │   └── RedisCacheService.cs
│   │   │
│   │   └── DependencyInjection.cs                      # Service registration for Infrastructure
│   │
│   └── ECommerceSystem.Api/                            # Ring 4: UI & HTTP Presentation
│       ├── Controllers/
│       │   ├── CustomersController.cs
│       │   ├── OrdersController.cs
│       │   └── ProductsController.cs
│       │
│       ├── Middleware/
│       │   ├── ExceptionHandlingMiddleware.cs
│       │   └── PerformanceLoggingMiddleware.cs
│       │
│       ├── Filters/
│       │   └── ApiKeyAuthFilter.cs
│       │
│       ├── Extensions/
│       │   └── SwaggerServiceExtensions.cs
│       │
│       ├── appsettings.json
│       ├── appsettings.Development.json
│       └── Program.cs                                  # Composition Root
│
└── tests/
    ├── ECommerceSystem.Domain.UnitTests/
    │   ├── Entities/
    │   │   └── OrderTests.cs
    │   └── DomainServices/
    │       └── PricingDomainServiceTests.cs
    │
    ├── ECommerceSystem.Application.UnitTests/
    │   └── Services/
    │       └── OrderApplicationServiceTests.cs
    │
    └── ECommerceSystem.Api.IntegrationTests/
        └── Controllers/
            └── OrdersControllerIntegrationTests.cs
```

---

## 03 — Folder Responsibilities & Project Matrix

| Project / Folder | Onion Ring Level | Primary Responsibility | Dependencies Allowed |
|---|---|---|---|
| `Domain/Entities/` | Ring 1 (Center) | Enterprise objects with unique identity and lifecycle. | None (Pure C# / .NET) |
| `Domain/ValueObjects/` | Ring 1 (Center) | Immutable objects defined solely by their property values. | None |
| `Domain/DomainServices/` | Ring 2 | Complex business rules spanning multiple domain entities. | `Domain` types only |
| `Application/Services/` | Ring 3 | Coordinates workflows and use cases; manages transactions. | `Domain` |
| `Application/Repositories/` | Ring 3 | Defines persistence contracts (interfaces) needed by use cases. | `Domain` |
| `Infrastructure/Persistence/` | Ring 4 (Outer) | Concrete database implementation, EF Core mapping, migrations. | `Application`, `Domain` |
| `Infrastructure/ExternalServices/` | Ring 4 (Outer) | Third-party SDK integrations (Stripe, SendGrid, Redis). | `Application`, `Domain` |
| `Api/Controllers/` | Ring 4 (Outer) | HTTP routing, request parsing, status codes, authentication. | `Application`, `Domain` |

---

## 04 — Dependency Rules

1. `Domain` references **nothing** outside itself.
2. `Application` references **Domain**.
3. `Infrastructure` references **Application** and **Domain**.
4. `Api` references **Application** and **Infrastructure** (strictly for DI wiring at startup).
5. Outer rings adapt to inner rings; inner rings are completely unaware of outer rings.
