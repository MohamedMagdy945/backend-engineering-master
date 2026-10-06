src/
├── MyApp.Domain/                      # Enterprise business rules (no dependencies)
│   ├── Common/
│   │   ├── Entity.cs
│   │   ├── AggregateRoot.cs
│   │   ├── ValueObject.cs
│   │   └── IDomainEvent.cs
│   ├── Orders/
│   │   ├── Order.cs                   # Aggregate root
│   │   ├── OrderItem.cs
│   │   ├── OrderStatus.cs
│   │   ├── Events/OrderPlacedEvent.cs
│   │   └── Errors/OrderErrors.cs
│   ├── Customers/
│   │   ├── Customer.cs
│   │   └── Email.cs                   # Value object
│   └── Exceptions/DomainException.cs
│
├── MyApp.Application/                 # Use cases, organized as VERTICAL SLICES
│   ├── Common/
│   │   ├── Abstractions/
│   │   │   ├── IApplicationDbContext.cs
│   │   │   ├── IDateTimeProvider.cs
│   │   │   ├── ICurrentUser.cs
│   │   │   └── IEmailSender.cs
│   │   ├── Behaviors/                 # MediatR pipeline
│   │   │   ├── ValidationBehavior.cs
│   │   │   ├── LoggingBehavior.cs
│   │   │   ├── TransactionBehavior.cs
│   │   │   └── CachingBehavior.cs
│   │   ├── Results/Result.cs
│   │   ├── Exceptions/
│   │   └── Mappings/
│   ├── Features/
│   │   ├── Orders/
│   │   │   ├── CreateOrder/
│   │   │   │   ├── CreateOrderCommand.cs
│   │   │   │   ├── CreateOrderHandler.cs
│   │   │   │   ├── CreateOrderValidator.cs
│   │   │   │   └── CreateOrderResponse.cs
│   │   │   ├── GetOrderById/
│   │   │   │   ├── GetOrderByIdQuery.cs
│   │   │   │   ├── GetOrderByIdHandler.cs
│   │   │   │   └── OrderDto.cs
│   │   │   ├── CancelOrder/
│   │   │   └── ListOrders/
│   │   └── Customers/
│   │       ├── RegisterCustomer/
│   │       └── GetCustomerProfile/
│   └── DependencyInjection.cs
│
├── MyApp.Infrastructure/              # External concerns
│   ├── Persistence/
│   │   ├── ApplicationDbContext.cs
│   │   ├── Configurations/            # EF entity configs
│   │   │   └── OrderConfiguration.cs
│   │   ├── Migrations/
│   │   ├── Interceptors/
│   │   │   └── DomainEventDispatcherInterceptor.cs
│   │   └── Repositories/              # Only if you need them
│   ├── Identity/
│   ├── Services/
│   │   ├── DateTimeProvider.cs
│   │   └── SmtpEmailSender.cs
│   ├── Caching/
│   ├── Messaging/                     # RabbitMQ / Service Bus
│   └── DependencyInjection.cs
│
└── MyApp.Api/                         # Presentation (endpoints per slice)
    ├── Endpoints/
    │   ├── Orders/
    │   │   ├── CreateOrderEndpoint.cs
    │   │   ├── GetOrderByIdEndpoint.cs
    │   │   └── OrdersModule.cs        # MapGroup("/orders")
    │   └── Customers/
    ├── Middleware/
    │   └── ExceptionHandlingMiddleware.cs
    ├── Extensions/
    ├── Program.cs
    └── appsettings.json

tests/
├── MyApp.Domain.UnitTests/
├── MyApp.Application.UnitTests/       # Handler tests per slice
├── MyApp.Api.IntegrationTests/        # WebApplicationFactory + Testcontainers
└── MyApp.ArchitectureTests/           # NetArchTest: enforce dependency rules