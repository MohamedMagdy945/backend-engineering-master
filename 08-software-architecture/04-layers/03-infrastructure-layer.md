# 03 — Infrastructure Layer

The Infrastructure Layer is the layer that provides **concrete technical implementations** for the application.

It deals with external systems, databases, networks, file systems, third-party services, and frameworks.

> **The Infrastructure Layer answers the question: How are technical operations implemented?**

---

## 01 — What Is the Infrastructure Layer?

The Infrastructure Layer contains all the code that interacts with things **outside** the application's memory and process.

It translates the abstract needs of the Domain and Application layers into concrete interactions with technology:

```text
Infrastructure Layer
├── Database Persistence (EF Core, Dapper, SQL Server, PostgreSQL)
├── Repository Implementations
├── Unit of Work Implementation
├── External APIs & Clients (Stripe, Twilio, SendGrid)
├── Authentication & Identity Services (ASP.NET Core Identity, JWT)
├── Message Brokers & Event Buses (RabbitMQ, Azure Service Bus, Kafka)
├── Caching Implementations (Redis, Distributed Cache)
├── File Storage (AWS S3, Azure Blob, Local File System)
├── Background Jobs & Schedulers (Hangfire, Quartz, Hosted Services)
└── System Clock & Date Providers (IDateTimeProvider)
```

The core principle of the Infrastructure Layer is:

> **It depends on the inside; the inside does NOT depend on it.**

The Infrastructure layer implements interfaces that were defined by the **Domain** or **Application** layers.

---

## 02 — Where Does the Infrastructure Layer Sit?

In the Clean Architecture concentric circle diagram, the Infrastructure Layer sits on the outer edge alongside Presentation:

```text
┌──────────────────────────────────────┐
│          Presentation                │
│                                      │
│   ┌──────────────────────────────┐   │
│   │ ► Infrastructure ◄───────────┼───┼── You are here
│   │                              │   │
│   │   ┌──────────────────────┐   │   │
│   │   │     Application      │   │   │
│   │   │                      │   │   │
│   │   │   ┌──────────────┐   │   │   │
│   │   │   │    Domain    │   │   │   │
│   │   │   └──────────────┘   │   │   │
│   │   └──────────────────────┘   │   │
│   └──────────────────────────────┘   │
└──────────────────────────────────────┘
```

Notice the direction of dependencies:

```text
Infrastructure → Application   ✔ Allowed (implements Application abstractions)
Infrastructure → Domain        ✔ Allowed (implements Domain abstractions)
Application    → Infrastructure ❌ NOT allowed (Application depends on abstractions)
Domain         → Infrastructure ❌ NOT allowed (Domain has ZERO dependencies)
```

Through **Dependency Inversion (DIP)**:
1. The **Application layer** defines an interface (e.g., `IEmailService`, `IOrderRepository`).
2. The **Infrastructure layer** implements that interface (e.g., `SmtpEmailService`, `OrderRepository`).
3. At runtime, the Dependency Injection container wires them together.

---

## 03 — What Belongs in the Infrastructure Layer?

Let's examine each major component that belongs in the Infrastructure Layer.

---

### 1. Database Context & ORM Configuration (EF Core)

The database context and its mappings belong in Infrastructure. The Domain layer must **never** be polluted with EF Core attributes or database column names.

Instead, use EF Core's **Fluent API** with separate configuration classes:

```csharp
// Infrastructure/Persistence/Configurations/OrderConfiguration.cs
using Microsoft.EntityFrameworkCore;
using Microsoft.EntityFrameworkCore.Metadata.Builders;

public class OrderConfiguration : IEntityTypeConfiguration<Order>
{
    public void Configure(EntityTypeBuilder<Order> builder)
    {
        builder.ToTable("Orders");

        builder.HasKey(o => o.Id);

        builder.Property(o => o.CustomerId)
            .IsRequired();

        builder.Property(o => o.Status)
            .HasConversion<string>()
            .HasMaxLength(50)
            .IsRequired();

        builder.Property(o => o.CreatedAt)
            .IsRequired();

        // Configure private collection navigation
        builder.HasMany(o => o.Items)
            .WithOne()
            .HasForeignKey("OrderId")
            .OnDelete(DeleteBehavior.Cascade);

        // Value Object mapping (OwnsOne)
        builder.OwnsOne(o => o.ShippingAddress, address =>
        {
            address.Property(a => a.Street).HasColumnName("Street").HasMaxLength(150);
            address.Property(a => a.City).HasColumnName("City").HasMaxLength(100);
            address.Property(a => a.PostalCode).HasColumnName("PostalCode").HasMaxLength(20);
        });
    }
}
```

And the `ApplicationDbContext`:

```csharp
// Infrastructure/Persistence/ApplicationDbContext.cs
using Microsoft.EntityFrameworkCore;

public class ApplicationDbContext : DbContext, IUnitOfWork
{
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<Customer> Customers => Set<Customer>();

    public ApplicationDbContext(DbContextOptions<ApplicationDbContext> options)
        : base(options)
    {
    }

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        // Automatically applies all configurations in this assembly
        modelBuilder.ApplyConfigurationsFromAssembly(typeof(ApplicationDbContext).Assembly);
        base.OnModelCreating(modelBuilder);
    }

    public async Task<int> CommitChangesAsync(CancellationToken cancellationToken = default)
    {
        return await base.SaveChangesAsync(cancellationToken);
    }
}
```

---

### 2. Repository Implementations

The interface was declared in the **Domain** or **Application** layer:

```csharp
// Domain/Repositories/IOrderRepository.cs  (or Application)
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(Guid id, CancellationToken cancellationToken = default);
    Task AddAsync(Order order, CancellationToken cancellationToken = default);
    void Update(Order order);
    void Remove(Order order);
}
```

The concrete implementation lives in **Infrastructure**:

```csharp
// Infrastructure/Persistence/Repositories/OrderRepository.cs
using Microsoft.EntityFrameworkCore;

public class OrderRepository : IOrderRepository
{
    private readonly ApplicationDbContext _context;

    public OrderRepository(ApplicationDbContext context)
    {
        _context = context;
    }

    public async Task<Order?> GetByIdAsync(Guid id, CancellationToken cancellationToken = default)
    {
        return await _context.Orders
            .Include(o => o.Items)
            .FirstOrDefaultAsync(o => o.Id == id, cancellationToken);
    }

    public async Task AddAsync(Order order, CancellationToken cancellationToken = default)
    {
        await _context.Orders.AddAsync(order, cancellationToken);
    }

    public void Update(Order order)
    {
        _context.Orders.Update(order);
    }

    public void Remove(Order order)
    {
        _context.Orders.Remove(order);
    }
}
```

---

### 3. External API Integrations (Third-Party Clients)

When communicating with external HTTP APIs, payment processors, or SMS gateways, the concrete HTTP calls belong here.

Interface in Application:

```csharp
// Application/Common/Interfaces/IPaymentGateway.cs
public interface IPaymentGateway
{
    Task<PaymentResult> ChargeAsync(decimal amount, string currency, string paymentMethodId, CancellationToken cancellationToken = default);
}
```

Implementation in Infrastructure:

```csharp
// Infrastructure/Services/Payments/StripePaymentGateway.cs
using Stripe;

public class StripePaymentGateway : IPaymentGateway
{
    private readonly StripeSettings _settings;
    private readonly ILogger<StripePaymentGateway> _logger;

    public StripePaymentGateway(IOptions<StripeSettings> settings, ILogger<StripePaymentGateway> logger)
    {
        _settings = settings.Value;
        _logger = logger;
    }

    public async Task<PaymentResult> ChargeAsync(decimal amount, string currency, string paymentMethodId, CancellationToken cancellationToken = default)
    {
        try
        {
            StripeConfiguration.ApiKey = _settings.SecretKey;

            var service = new PaymentIntentService();
            var options = new PaymentIntentCreateOptions
            {
                Amount = (long)(amount * 100), // Stripe expects cents
                Currency = currency.ToLowerInvariant(),
                PaymentMethod = paymentMethodId,
                Confirm = true
            };

            var intent = await service.CreateAsync(options, cancellationToken: cancellationToken);

            return intent.Status == "succeeded" 
                ? PaymentResult.Success(intent.Id)
                : PaymentResult.Failed(intent.LastPaymentError?.Message ?? "Charge incomplete");
        }
        catch (StripeException ex)
        {
            _logger.LogError(ex, "Stripe charge failed for payment method {PaymentMethodId}", paymentMethodId);
            return PaymentResult.Failed(ex.Message);
        }
    }
}
```

---

### 4. Email and Notification Services

Interface in Application:

```csharp
// Application/Common/Interfaces/IEmailService.cs
public interface IEmailService
{
    Task SendEmailAsync(string recipientEmail, string subject, string body, CancellationToken cancellationToken = default);
}
```

Implementation in Infrastructure:

```csharp
// Infrastructure/Services/Email/SendGridEmailService.cs
using SendGrid;
using SendGrid.Helpers.Mail;

public class SendGridEmailService : IEmailService
{
    private readonly ISendGridClient _client;
    private readonly EmailSettings _settings;

    public SendGridEmailService(ISendGridClient client, IOptions<EmailSettings> settings)
    {
        _client = client;
        _settings = settings.Value;
    }

    public async Task SendEmailAsync(string recipientEmail, string subject, string body, CancellationToken cancellationToken = default)
    {
        var msg = new SendGridMessage
        {
            From = new EmailAddress(_settings.SenderEmail, _settings.SenderName),
            Subject = subject,
            HtmlContent = body
        };
        msg.AddTo(new EmailAddress(recipientEmail));

        var response = await _client.SendEmailAsync(msg, cancellationToken);
        if (!response.IsSuccessStatusCode)
        {
            throw new InvalidOperationException($"Failed to send email. Status code: {response.StatusCode}");
        }
    }
}
```

---

### 5. Authentication & Token Generation (JWT)

Interface in Application:

```csharp
// Application/Common/Interfaces/IJwtTokenGenerator.cs
public interface IJwtTokenGenerator
{
    string GenerateToken(User user, IEnumerable<string> roles);
}
```

Implementation in Infrastructure:

```csharp
// Infrastructure/Authentication/JwtTokenGenerator.cs
using System.IdentityModel.Tokens.Jwt;
using System.Security.Claims;
using System.Text;
using Microsoft.IdentityModel.Tokens;

public class JwtTokenGenerator : IJwtTokenGenerator
{
    private readonly JwtSettings _jwtSettings;

    public JwtTokenGenerator(IOptions<JwtSettings> jwtSettings)
    {
        _jwtSettings = jwtSettings.Value;
    }

    public string GenerateToken(User user, IEnumerable<string> roles)
    {
        var claims = new List<Claim>
        {
            new(JwtRegisteredClaimNames.Sub, user.Id.ToString()),
            new(JwtRegisteredClaimNames.Email, user.Email),
            new(JwtRegisteredClaimNames.Jti, Guid.NewGuid().ToString())
        };

        foreach (var role in roles)
        {
            claims.Add(new Claim(ClaimTypes.Role, role));
        }

        var key = new SymmetricSecurityKey(Encoding.UTF8.GetBytes(_jwtSettings.SecretKey));
        var creds = new SigningCredentials(key, SecurityAlgorithms.HmacSha256);

        var token = new JwtSecurityToken(
            issuer: _jwtSettings.Issuer,
            audience: _jwtSettings.Audience,
            claims: claims,
            expires: DateTime.UtcNow.AddMinutes(_jwtSettings.ExpiryMinutes),
            signingCredentials: creds
        );

        return new JwtSecurityTokenHandler().WriteToken(token);
    }
}
```

---

### 6. Caching Implementation (Redis / Distributed Cache)

Interface in Application:

```csharp
// Application/Common/Interfaces/ICacheService.cs
public interface ICacheService
{
    Task<T?> GetAsync<T>(string key, CancellationToken cancellationToken = default);
    Task SetAsync<T>(string key, T value, TimeSpan? expiration = null, CancellationToken cancellationToken = default);
    Task RemoveAsync(string key, CancellationToken cancellationToken = default);
}
```

Implementation in Infrastructure:

```csharp
// Infrastructure/Caching/RedisCacheService.cs
using System.Text.Json;
using Microsoft.Extensions.Caching.Distributed;

public class RedisCacheService : ICacheService
{
    private readonly IDistributedCache _cache;

    public RedisCacheService(IDistributedCache cache)
    {
        _cache = cache;
    }

    public async Task<T?> GetAsync<T>(string key, CancellationToken cancellationToken = default)
    {
        var bytes = await _cache.GetAsync(key, cancellationToken);
        if (bytes == null || bytes.Length == 0) return default;

        return JsonSerializer.Deserialize<T>(bytes);
    }

    public async Task SetAsync<T>(string key, T value, TimeSpan? expiration = null, CancellationToken cancellationToken = default)
    {
        var bytes = JsonSerializer.SerializeToUtf8Bytes(value);
        var options = new DistributedCacheEntryOptions
        {
            AbsoluteExpirationRelativeToNow = expiration ?? TimeSpan.FromMinutes(10)
        };

        await _cache.SetAsync(key, bytes, options, cancellationToken);
    }

    public async Task RemoveAsync(string key, CancellationToken cancellationToken = default)
    {
        await _cache.RemoveAsync(key, cancellationToken);
    }
}
```

---

### 7. Clock & Time Provider

Always abstract `DateTime.UtcNow` so that time-dependent application logic can be deterministically tested.

Interface in Application:

```csharp
// Application/Common/Interfaces/IDateTimeProvider.cs
public interface IDateTimeProvider
{
    DateTime UtcNow { get; }
}
```

Implementation in Infrastructure:

```csharp
// Infrastructure/Services/SystemDateTimeProvider.cs
public class SystemDateTimeProvider : IDateTimeProvider
{
    public DateTime UtcNow => DateTime.UtcNow;
}
```

---

### 8. Message Broker & Event Publishing (Outbox Pattern)

Publishing integration events or domain events to message brokers like RabbitMQ or Kafka belongs in Infrastructure.

```csharp
// Infrastructure/Messaging/RabbitMqEventPublisher.cs
public class RabbitMqEventPublisher : IEventPublisher
{
    private readonly IBus _bus; // e.g. MassTransit

    public RabbitMqEventPublisher(IBus bus)
    {
        _bus = bus;
    }

    public async Task PublishAsync<TEvent>(TEvent @event, CancellationToken cancellationToken = default) 
        where TEvent : class
    {
        await _bus.Publish(@event, cancellationToken);
    }
}
```

To prevent data inconsistencies between database writes and message broker publishes, Infrastructure implements the **Transactional Outbox Pattern**:

```text
Application Handler
       │
       ▼
1. Save Entity to Database
2. Save OutboxMessage to same DB Transaction
       │
       ▼
Commit Transaction (Atomic)
       │
       ▼
Background Job in Infrastructure (e.g., Quartz / Hosted Service)
       │
       ├── Reads unhandled OutboxMessage records
       ├── Publishes message to RabbitMQ / Kafka
       └── Marks record as Processed
```

---

## 04 — What Does NOT Belong in the Infrastructure Layer?

The Infrastructure Layer is strictly for technical details and external integrations. It must not contain:

```text
❌ Business Rules
   → Belongs in Domain (e.g., "Discount cannot exceed 30%")

❌ Domain Invariants
   → Belongs in Domain entities

❌ Use Case Orchestration
   → Belongs in Application (e.g., "Load customer, validate, charge, save, notify")

❌ HTTP Controllers & Route Definitions
   → Belongs in API / Presentation

❌ Web-Specific ViewModels or HTTP Status Codes
   → Belongs in API / Presentation
```

---

## 05 — Dependency Inversion in Action

The central design principle enabling Clean Architecture is the **Dependency Inversion Principle (DIP)**:

> High-level modules should not depend on low-level modules. Both should depend on abstractions.
> Abstractions should not depend on details. Details should depend on abstractions.

### Without Dependency Inversion (Tightly Coupled)

```text
Application Layer
       │
       │ (depends on concrete class)
       ▼
Infrastructure Layer (SqlServerOrderRepository)
```

If we change from SQL Server to MongoDB, or want to write unit tests for the Application layer, we are trapped.

### With Dependency Inversion

```text
┌──────────────────────────────────────────────┐
│ Application Layer                            │
│                                              │
│   CreateOrderCommandHandler                  │
│             │                                │
│             ▼                                │
│      IOrderRepository  (Interface)           │
│             ▲                                │
└─────────────┼────────────────────────────────┘
              │ Implements
┌─────────────┼────────────────────────────────┐
│ Infrastructure Layer                         │
│                                              │
│      SqlOrderRepository (Implementation)     │
│             │                                │
│             ▼                                │
│      EF Core / SQL Server                    │
└──────────────────────────────────────────────┘
```

The Application layer owns the abstraction `IOrderRepository`.
The Infrastructure layer conforms to that abstraction.
The arrow of source code dependency points **inward**.

---

## 06 — Infrastructure Layer vs Other Layers

| Concern | Domain Layer | Application Layer | Infrastructure Layer | API / Presentation |
|---|---|---|---|---|
| **Primary Question** | *What are the rules?* | *What does the app do?* | *How is it implemented technically?* | *How do clients communicate?* |
| **Contains** | Entities, Value Objects, Domain Events | Commands, Queries, Handlers, DTOs | EF Core, Repositories, Redis, Stripe, JWT | Controllers, Minimal APIs, Middleware |
| **Dependencies** | None | Domain only | Domain, Application | Application, Infrastructure (DI only) |
| **Database knowledge?** | No | No | **Yes** | No |
| **HTTP knowledge?** | No | No | Yes (for 3rd-party HTTP clients) | **Yes** (incoming requests/responses) |
| **Stability** | Very High (rarely changes) | High | Low (technologies change) | Low (APIs/UI evolve) |

---

## 07 — The Anti-Corruption Layer (ACL) Pattern

When the Infrastructure Layer calls external APIs (e.g., Stripe, Shopify, Salesforce), their data models, naming conventions, and concepts rarely match our clean Domain model.

If we let external data models leak into our Application layer, our system becomes corrupted:

```text
❌ BAD: External models leak into our core
External API DTO ────────► Application Handler ────────► Domain Model
```

Instead, use an **Anti-Corruption Layer (ACL)** inside Infrastructure:

```text
✔ GOOD: Anti-Corruption Layer inside Infrastructure
External API DTO ──► Infrastructure Adapter ──► Maps to Application/Domain Model
```

Example Adapter:

```csharp
// Infrastructure/ExternalServices/Shipping/FedExShippingAdapter.cs
public class FedExShippingAdapter : IShippingRateCalculator
{
    private readonly FedExApiClient _client;

    public FedExShippingAdapter(FedExApiClient client)
    {
        _client = client;
    }

    public async Task<ShippingEstimate> CalculateRateAsync(PostalCode origin, PostalCode destination, Weight weight)
    {
        // 1. Translate internal domain request to FedEx request format
        var fedExRequest = new FedExRateRequestDto
        {
            OriginZip = origin.Value,
            DestZip = destination.Value,
            PackageWeightLbs = weight.ToPounds()
        };

        // 2. Call external system
        var fedExResponse = await _client.GetRateAsync(fedExRequest);

        // 3. Translate FedEx response into clean Domain/Application model
        return new ShippingEstimate(
            Carrier: "FedEx",
            Cost: Money.FromDecimal(fedExResponse.TotalNetCharge.Amount, fedExResponse.TotalNetCharge.Currency),
            EstimatedDeliveryDays: fedExResponse.TransitDays
        );
    }
}
```

The Application and Domain layers remain 100% untouched by FedEx's internal API changes.

---

## 08 — Dependency Injection Registration

How does the rest of the application get access to these concrete implementations without referencing them directly in use cases?

Through **Service Registration Extension Methods**:

```csharp
// Infrastructure/DependencyInjection.cs
using Microsoft.EntityFrameworkCore;
using Microsoft.Extensions.Configuration;
using Microsoft.Extensions.DependencyInjection;

public static class DependencyInjection
{
    public static IServiceCollection AddInfrastructure(this IServiceCollection services, IConfiguration configuration)
    {
        // 1. Database
        services.AddDbContext<ApplicationDbContext>(options =>
            options.UseSqlServer(
                configuration.GetConnectionString("DefaultConnection"),
                b => b.MigrationsAssembly(typeof(ApplicationDbContext).Assembly.FullName)));

        services.AddScoped<IUnitOfWork>(sp => sp.GetRequiredService<ApplicationDbContext>());

        // 2. Repositories
        services.AddScoped<IOrderRepository, OrderRepository>();
        services.AddScoped<ICustomerRepository, CustomerRepository>();

        // 3. External Services
        services.Configure<EmailSettings>(configuration.GetSection("EmailSettings"));
        services.AddScoped<IEmailService, SendGridEmailService>();

        services.Configure<StripeSettings>(configuration.GetSection("StripeSettings"));
        services.AddScoped<IPaymentGateway, StripePaymentGateway>();

        // 4. Caching
        services.AddStackExchangeRedisCache(options =>
        {
            options.Configuration = configuration.GetConnectionString("Redis");
        });
        services.AddSingleton<ICacheService, RedisCacheService>();

        // 5. Cross-cutting
        services.AddSingleton<IDateTimeProvider, SystemDateTimeProvider>();

        // 6. Authentication
        services.Configure<JwtSettings>(configuration.GetSection("JwtSettings"));
        services.AddSingleton<IJwtTokenGenerator, JwtTokenGenerator>();

        return services;
    }
}
```

In the API Layer's `Program.cs`, we simply call:

```csharp
builder.Services.AddApplication();
builder.Services.AddInfrastructure(builder.Configuration);
```

---

## 09 — Infrastructure Layer Folder Structure

Here is a standard, battle-tested folder organization for the Infrastructure Layer:

```text
Infrastructure/
├── Persistence/
│   ├── ApplicationDbContext.cs
│   ├── Configurations/
│   │   ├── OrderConfiguration.cs
│   │   └── CustomerConfiguration.cs
│   ├── Repositories/
│   │   ├── OrderRepository.cs
│   │   └── CustomerRepository.cs
│   └── Migrations/
│       ├── 20261001_InitialCreate.cs
│       └── ApplicationDbContextModelSnapshot.cs
├── Services/
│   ├── Email/
│   │   ├── SendGridEmailService.cs
│   │   └── EmailSettings.cs
│   ├── Payments/
│   │   ├── StripePaymentGateway.cs
│   │   └── StripeSettings.cs
│   └── SystemDateTimeProvider.cs
├── Authentication/
│   ├── JwtTokenGenerator.cs
│   └── JwtSettings.cs
├── Caching/
│   ├── RedisCacheService.cs
│   └── CacheOptions.cs
├── Messaging/
│   ├── Outbox/
│   │   ├── OutboxMessage.cs
│   │   └── ProcessOutboxMessagesJob.cs
│   └── RabbitMqEventPublisher.cs
├── FileStorage/
│   ├── S3FileStorageService.cs
│   └── S3Settings.cs
└── DependencyInjection.cs
```

Each technical concern has its own folder. Adding, changing, or removing a technical provider is isolated.

---

## 10 — Testing the Infrastructure Layer

Because the Infrastructure Layer interacts with databases, network sockets, and file systems, testing it requires a different approach than Domain and Application layers.

```text
Domain Layer       → Unit Tests (Pure logic, fastest)
Application Layer  → Unit Tests (Mocked interfaces)
Infrastructure     → Integration Tests (Real containers/databases)
```

### Best Practice: Testcontainers for Persistence Tests

Do **not** use the EF Core InMemory database provider for testing Infrastructure repositories. In-memory databases do not support transactions, relational constraints, or database-specific SQL features.

Instead, use **Testcontainers** to spin up a lightweight real database instance in Docker:

```csharp
// Infrastructure.IntegrationTests/Repositories/OrderRepositoryTests.cs
using Testcontainers.MsSql;
using Xunit;

public class OrderRepositoryTests : IAsyncLifetime
{
    private readonly MsSqlContainer _dbContainer = new MsSqlBuilder()
        .WithImage("mcr.microsoft.com/mssql/server:2022-latest")
        .Build();

    private ApplicationDbContext _context = null!;
    private OrderRepository _repository = null!;

    public async Task InitializeAsync()
    {
        await _dbContainer.StartAsync();

        var options = new DbContextOptionsBuilder<ApplicationDbContext>()
            .UseSqlServer(_dbContainer.GetConnectionString())
            .Options;

        _context = new ApplicationDbContext(options);
        await _context.Database.EnsureCreatedAsync();

        _repository = new OrderRepository(_context);
    }

    [Fact]
    public async Task AddAsync_ShouldPersistOrderInDatabase()
    {
        // Arrange
        var customerId = Guid.NewGuid();
        var order = new Order(customerId);

        // Act
        await _repository.AddAsync(order);
        await _context.SaveChangesAsync();

        // Assert
        var persistedOrder = await _repository.GetByIdAsync(order.Id);
        Assert.NotNull(persistedOrder);
        Assert.Equal(customerId, persistedOrder.CustomerId);
    }

    public async Task DisposeAsync()
    {
        await _context.DisposeAsync();
        await _dbContainer.DisposeAsync();
    }
}
```

---

## 11 — Common Mistakes

### Mistake 1: Defining interfaces in Infrastructure

```text
❌ Defining IOrderRepository inside Infrastructure
✔ Defining IOrderRepository inside Domain or Application
```

If the interface is defined in Infrastructure, then Application would have to reference Infrastructure to use it, violating the Dependency Rule.

---

### Mistake 2: Leaking ORM models or database attributes into Domain

```text
❌ Putting [Table("Orders")], [Key], or [ForeignKey] on Domain entities
✔ Keeping Domain entities clean POCOs and using EF Core Fluent API configurations in Infrastructure
```

---

### Mistake 3: Business rules sneaking into Repositories

```text
❌ Repository method:
   if (order.Total > 1000) applyDiscount();
✔ Repositories only perform queries and state persistence; rules live in the Domain entity or domain service
```

---

### Mistake 4: Using EF Core directly in Application Handlers

```text
❌ Command Handler injects ApplicationDbContext directly
✔ Command Handler injects IOrderRepository or IUnitOfWork abstractions
```

Injecting `DbContext` directly into handlers ties the use case to EF Core and makes unit testing significantly harder.

---

### Mistake 5: Hardcoding connection strings and API keys

```text
❌ Hardcoding "Server=localhost;..." or "sk_test_..." in code
✔ Using the Options pattern (IOptions<T>) with strongly typed settings bound to appsettings.json or environment variables
```

---

## 12 — Mental Model

Think of the Infrastructure Layer as the **drivers, cables, and plumbing**:

```text
Application Core  → The computer's operating system (processes logic and intent)
Infrastructure    → The hardware drivers, graphics cards, network cables, and power supply
```

The operating system doesn't care whether your monitor is an LG or a Dell; it speaks through a generic display driver interface. 

Similarly:
```text
Application wants: "Save this order"
Infrastructure executes: "INSERT INTO Orders (Id, CustomerId...) VALUES (...)"

Application wants: "Notify customer"
Infrastructure executes: "POST https://api.sendgrid.com/v3/mail/send HTTP/1.1"

Application wants: "Charge $50"
Infrastructure executes: "POST https://api.stripe.com/v1/payment_intents HTTP/1.1"
```

The core is insulated from the external world.

---

## 13 — Summary

```text
Infrastructure Layer
├── Provides concrete technical implementations
├── Handles persistence, databases, and ORMs (EF Core, Dapper)
├── Integrates with external APIs (Stripe, Twilio, SendGrid)
├── Implements caching, messaging, and authentication
├── Implements abstractions defined by Domain and Application layers
├── Protects core with the Anti-Corruption Layer (ACL)
├── Registers its dependencies via service collection extension methods
├── Tested via Integration Tests (Testcontainers)
└── Contains ZERO business logic and ZERO use case orchestration
```

> **The Infrastructure Layer is the servant of the application core. It makes things happen in the physical world without dictating the rules of the domain.**

---
