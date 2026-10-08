# 04 — Presentation Layer

The Presentation Layer (often implemented as an **API**, **Web**, or **UI** project) is the **outermost boundary** and **entry point** of the application.

It is responsible for interfacing with external actors — human users, web browsers, mobile applications, external microservices, message buses, or CLI tools — and translating external requests into application operations.

> **The Presentation Layer answers the question: How does the outside world interact with the application?**

---

## 01 — What Is the Presentation Layer?

The Presentation Layer sits on the outermost perimeter of Clean Architecture. Its purpose is to adapt the application core to the outside world, and adapt the outside world to the application core.

```text
Presentation Layer
├── HTTP Endpoints (Controllers & Minimal APIs)
├── Alternative Interfaces (gRPC Services, GraphQL, SignalR Hubs, CLI)
├── Request & Response Contracts (HTTP DTOs / ViewModels)
├── Mappings (External Contracts ➔ Application Commands/Queries)
├── Global Exception Handling & ProblemDetails (RFC 7807)
├── Middleware & Pipeline Filters (Correlation IDs, Security Headers)
├── Authentication & Authorization Enforcement (JWT, Policies)
├── API Documentation (Swagger / OpenAPI)
├── API Versioning, Rate Limiting & CORS
└── Composition Root (Program.cs — DI Bootstrapping)
```

The golden rule of the Presentation Layer is:

> **It is thin. It does not think. It receives, translates, delegates, and responds.**

The Presentation Layer does not define business rules (Domain), does not coordinate business workflows (Application), and does not talk directly to databases or physical hardware (Infrastructure). It handles **transport and protocol concerns**.

---

## 02 — Where Does the Presentation Layer Sit?

In Clean Architecture, the Presentation Layer sits on the outermost circle alongside Infrastructure:

```text
┌──────────────────────────────────────┐
│ ► Presentation Layer ◄───────────────┼── You are here (Entry Point)
│                                      │
│   ┌──────────────────────────────┐   │
│   │        Infrastructure        │   │
│   │                              │   │
│   │   ┌──────────────────────┐   │   │
│   │   │     Application      │   │   │
│   │   │                      │   │   │
│   │   │   ┌──────────────┐   │   │   │
│   │   │   │    Domain    │   │   │   │
│   │   │   │              │   │   │   │
│   │   │   └──────────────┘   │   │   │
│   │   └──────────────────────┘   │   │
│   └──────────────────────────────┘   │
└──────────────────────────────────────┘
```

### Dependency Rules:

```text
Presentation → Application     ✔ Allowed (dispatches Commands, Queries, and Use Cases)
Presentation → Domain          ✔ Allowed (reads Domain types if necessary, but prefers Application DTOs)
Presentation → Infrastructure  ⚠️ ONLY as the Composition Root (Program.cs) for DI registration
```

Inside controllers, endpoints, or views, you should **never** directly inject or reference Infrastructure classes.

---

## 03 — What Belongs in the Presentation Layer?

Let's examine the essential responsibilities of the Presentation Layer.

---

### 1. Controllers & Minimal API Endpoints

The primary responsibility of an endpoint is:
1. Accept an incoming request across the transport protocol (HTTP, gRPC, etc.).
2. Bind parameters from the route, query string, headers, or body.
3. Validate format and syntactic inputs.
4. Translate the request into an Application **Command** or **Query**.
5. Dispatch to the Application Layer (e.g., via MediatR or direct handler injection).
6. Return the appropriate presentation result (HTTP status code, headers, body).

#### Approach A: Modern Minimal APIs

Minimal APIs offer high performance and clear modular endpoint grouping:

```csharp
// Presentation/Endpoints/Orders/CreateOrderEndpoint.cs
public static class CreateOrderEndpoint
{
    public static void MapCreateOrderEndpoint(this IEndpointRouteBuilder app)
    {
        app.MapPost("/api/v1/orders", async (
            CreateOrderRequest request,
            ClaimsPrincipal user,
            ISender sender,
            CancellationToken ct) =>
        {
            // 1. Extract transport details & map to Application Command
            var customerId = user.GetUserId();
            var command = new CreateOrderCommand(
                customerId,
                request.Items.Select(i => new OrderItemDto(i.ProductId, i.Quantity)).ToList()
            );

            // 2. Dispatch to Application Layer
            var result = await sender.Send(command, ct);

            // 3. Return appropriate HTTP response
            return Results.Created($"/api/v1/orders/{result.OrderId}", new OrderResponse(result.OrderId, result.TotalAmount));
        })
        .WithName("CreateOrder")
        .WithTags("Orders")
        .RequireAuthorization()
        .Produces<OrderResponse>(StatusCodes.Status201Created)
        .ProducesProblem(StatusCodes.Status400BadRequest);
    }
}
```

#### Approach B: Traditional Controllers

```csharp
// Presentation/Controllers/OrdersController.cs
[ApiController]
[Route("api/v1/[controller]")]
[Authorize]
public class OrdersController : ControllerBase
{
    private readonly ISender _sender;

    public OrdersController(ISender sender)
    {
        _sender = sender;
    }

    [HttpPost]
    [ProducesResponseType(typeof(OrderResponse), StatusCodes.Status201Created)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status400BadRequest)]
    public async Task<IActionResult> CreateOrder(
        [FromBody] CreateOrderRequest request,
        CancellationToken cancellationToken)
    {
        var command = new CreateOrderCommand(User.GetUserId(), request.Items.ToApplicationDto());
        var result = await _sender.Send(command, cancellationToken);

        return CreatedAtAction(
            nameof(GetOrderById),
            new { id = result.OrderId },
            new OrderResponse(result.OrderId, result.TotalAmount));
    }

    [HttpGet("{id:guid}")]
    [ProducesResponseType(typeof(OrderDetailsResponse), StatusCodes.Status200OK)]
    [ProducesResponseType(typeof(ProblemDetails), StatusCodes.Status404NotFound)]
    public async Task<IActionResult> GetOrderById(Guid id, CancellationToken cancellationToken)
    {
        var query = new GetOrderByIdQuery(id);
        var order = await _sender.Send(query, cancellationToken);

        return order is not null ? Ok(order) : NotFound();
    }
}
```

---

### 2. Multi-Channel Presentation (Beyond Just HTTP APIs)

The Presentation Layer is **not only REST APIs**. It handles whatever transport protocol the application needs:

```text
                        ┌────────────────────────┐
                        │   Presentation Layer   │
                        └───────────┬────────────┘
                                    │
       ┌─────────────────┬──────────┴──────────┬─────────────────┐
       ▼                 ▼                     ▼                 ▼
   REST API            gRPC                SignalR/WS           CLI
(Web & Mobile)    (Microservices)       (Real-Time Feeds)   (DevOps/Cron)
       │                 │                     │                 │
       └─────────────────┴──────────┬──────────┴─────────────────┘
                                    │ Translates into Commands/Queries
                                    ▼
                        ┌────────────────────────┐
                        │   Application Layer    │
                        └────────────────────────┘
```

For example, a **gRPC Service** belongs in the Presentation Layer:

```csharp
// Presentation/Grpc/OrderGrpcService.cs
public class OrderGrpcService : OrderProtoService.OrderProtoServiceBase
{
    private readonly ISender _sender;

    public OrderGrpcService(ISender sender)
    {
        _sender = sender;
    }

    public override async Task<OrderProtoResponse> GetOrderById(
        GetOrderProtoRequest request, 
        ServerCallContext context)
    {
        var query = new GetOrderByIdQuery(Guid.Parse(request.OrderId));
        var order = await _sender.Send(query, context.CancellationToken);

        if (order is null)
        {
            throw new RpcException(new Status(StatusCode.NotFound, "Order not found"));
        }

        return new OrderProtoResponse { Id = order.Id.ToString(), Total = (double)order.Total };
    }
}
```

Notice that both the REST controller and the gRPC service invoke the **exact same** `GetOrderByIdQuery` in the Application layer.

---

### 3. Request and Response Contracts (Contracts / DTOs)

Presentation models represent the external contracts with your consumers. They must remain isolated from internal entities and application models:

```csharp
// Presentation/Contracts/Orders/CreateOrderRequest.cs
public record CreateOrderRequest(
    List<CreateOrderItemRequest> Items,
    string? PromoCode
);

public record CreateOrderItemRequest(
    Guid ProductId,
    int Quantity
);

// Presentation/Contracts/Orders/OrderResponse.cs
public record OrderResponse(
    Guid OrderId,
    decimal TotalAmount
);
```

> **Why separate Presentation Contracts from Application DTOs?**  
> External consumers depend on API stability. If an internal application workflow modifies its parameter signature, the public HTTP API schema should not break unexpectedly.

---

### 4. Global Exception Handling & RFC 7807 ProblemDetails

Clients expect consistent, standardized error responses. The Presentation Layer catches domain, validation, and unhandled exceptions and maps them to **RFC 7807 ProblemDetails**:

```csharp
// Presentation/Middleware/GlobalExceptionHandler.cs
using Microsoft.AspNetCore.Diagnostics;
using Microsoft.AspNetCore.Mvc;

public class GlobalExceptionHandler : IExceptionHandler
{
    private readonly ILogger<GlobalExceptionHandler> _logger;

    public GlobalExceptionHandler(ILogger<GlobalExceptionHandler> logger)
    {
        _logger = logger;
    }

    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        _logger.LogError(exception, "Unhandled exception: {Message}", exception.Message);

        var (statusCode, title, detail) = exception switch
        {
            ValidationException valEx => (
                StatusCodes.Status400BadRequest,
                "Validation Error",
                string.Join("; ", valEx.Errors.Select(e => e.ErrorMessage))
            ),
            NotFoundException notFoundEx => (
                StatusCodes.Status404NotFound,
                "Resource Not Found",
                notFoundEx.Message
            ),
            DomainException domainEx => (
                StatusCodes.Status422UnprocessableEntity,
                "Business Rule Violation",
                domainEx.Message
            ),
            _ => (
                StatusCodes.Status500InternalServerError,
                "Internal Server Error",
                "An unexpected server error occurred."
            )
        };

        var problemDetails = new ProblemDetails
        {
            Status = statusCode,
            Title = title,
            Detail = detail,
            Instance = httpContext.Request.Path
        };

        httpContext.Response.StatusCode = statusCode;
        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);

        return true;
    }
}
```

---

### 5. Middleware and Pipeline Filters

Cross-cutting HTTP pipeline behaviors belong in the Presentation Layer:

* **Correlation ID Middleware:** Attaches a unique tracking ID to every request for distributed observability.
* **Security Headers:** Enforces CSP, HSTS, X-Content-Type-Options.
* **Rate Limiting:** Protects the system from abuse and denial of service.
* **CORS Policies:** Controls which browser origins can access the API.

```csharp
// Presentation/Middleware/CorrelationIdMiddleware.cs
public class CorrelationIdMiddleware
{
    private const string HeaderKey = "X-Correlation-ID";
    private readonly RequestDelegate _next;

    public CorrelationIdMiddleware(RequestDelegate next)
    {
        _next = next;
    }

    public async Task InvokeAsync(HttpContext context)
    {
        var correlationId = context.Request.Headers[HeaderKey].FirstOrDefault() 
            ?? Guid.NewGuid().ToString();

        context.Response.Headers[HeaderKey] = correlationId;

        using (LogContext.PushProperty("CorrelationId", correlationId))
        {
            await _next(context);
        }
    }
}
```

---

### 6. Authentication & Authorization Policies

Enforcing security at the gateway into your system:

```csharp
// Presentation/Extensions/SecurityExtensions.cs
public static class SecurityExtensions
{
    public static IServiceCollection AddPresentationSecurity(
        this IServiceCollection services, 
        IConfiguration configuration)
    {
        var jwtSettings = configuration.GetSection("JwtSettings").Get<JwtSettings>()!;

        services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
            .AddJwtBearer(options =>
            {
                options.TokenValidationParameters = new TokenValidationParameters
                {
                    ValidateIssuer = true,
                    ValidateAudience = true,
                    ValidateLifetime = true,
                    ValidateIssuerSigningKey = true,
                    ValidIssuer = jwtSettings.Issuer,
                    ValidAudience = jwtSettings.Audience,
                    IssuerSigningKey = new SymmetricSecurityKey(
                        Encoding.UTF8.GetBytes(jwtSettings.SecretKey))
                };
            });

        services.AddAuthorization(options =>
        {
            options.AddPolicy("AdminOnly", policy => policy.RequireRole("Admin"));
            options.AddPolicy("CanOrder", policy => policy.RequireClaim("scope", "orders:create"));
        });

        return services;
    }
}
```

---

## 04 — What Does NOT Belong in the Presentation Layer?

Because the Presentation Layer is where requests arrive, developers often succumb to the temptation of putting logic here:

```text
❌ Business Logic
   "if (order.Total > 100) applyDiscount();"
   Belongs in: Domain Layer

❌ Use Case Orchestration
   "1. Check inventory -> 2. Charge card -> 3. Save order -> 4. Send email"
   Belongs in: Application Layer

❌ Direct Database Queries
   "_dbContext.Orders.Where(o => o.CustomerId == id).ToList();"
   Belongs in: Infrastructure / Application Layer

❌ Direct Third-Party API Calls
   "new HttpClient().PostAsync("https://stripe.com/...", ...);"
   Belongs in: Infrastructure Layer
```

If your endpoint or controller contains SQL queries, database contexts, or business rules, your boundaries are compromised.

---

## 05 — The Composition Root (Program.cs)

In .NET, the Presentation Layer contains `Program.cs`. This acts as the **Composition Root** of the entire application.

The Composition Root is the single location where all layers are composed together:

```csharp
// Presentation/Program.cs
var builder = WebApplication.CreateBuilder(args);

// 1. Compose all architectural layers
builder.Services
    .AddApplication()
    .AddInfrastructure(builder.Configuration)
    .AddPresentation(builder.Configuration);

var app = builder.Build();

// 2. Configure HTTP request pipeline
app.UseMiddleware<CorrelationIdMiddleware>();
app.UseExceptionHandler();

if (app.Environment.IsDevelopment())
{
    app.UseSwagger();
    app.UseSwaggerUI();
}

app.UseHttpsRedirection();
app.UseCors("AllowFrontend");

app.UseAuthentication();
app.UseAuthorization();

// 3. Map presentation endpoints
app.MapControllers();
// app.MapOrderEndpoints();
// app.MapGrpcService<OrderGrpcService>();

app.Run();

// Accessible to integration test fixtures
public partial class Program { }
```

> **Why does Presentation reference Infrastructure?**  
> Solely so that `Program.cs` can register `builder.Services.AddInfrastructure(...)`.  
> No controller, endpoint, or middleware should ever directly instantiate or import concrete classes from the Infrastructure assembly.

---

## 06 — Presentation Layer vs Application Layer

| Feature | Presentation Layer | Application Layer |
|---|---|---|
| **Perspective** | External / Transport (HTTP, gRPC, CLI) | Internal / Business Use Cases |
| **Input** | `HttpRequest`, Query string, JSON, Form | `CreateOrderCommand`, `GetOrderByIdQuery` |
| **Output** | `200 OK`, `201 Created`, `404 NotFound`, JSON | `Result<OrderDto>`, `Result.Success()` |
| **Errors** | `RFC 7807 ProblemDetails`, HTTP Statuses | Result objects (`Error.Validation`, `NotFound`) |
| **Protocol Binding** | Bound to HTTP, gRPC, CLI, WebSockets | Completely protocol-agnostic |
| **Reusability** | Specific to transport channel | Reused across Web, CLI, background jobs |
| **Testing** | Integration Tests (`HttpClient`) | Unit Tests with mocked interfaces |

---

## 07 — The Complete Request-Response Journey

Here is how a request traverses all 4 layers from the outside world and back:

```text
1. Client sends HTTP POST /api/v1/orders
      │
      ▼
2. [PRESENTATION LAYER]
   ├── CorrelationIdMiddleware attaches tracking ID
   ├── Authentication validates Bearer JWT token
   └── CreateOrderEndpoint receives CreateOrderRequest
      │
      ▼ (Maps to CreateOrderCommand)
3. [APPLICATION LAYER]
   ├── ValidationBehavior validates command inputs (FluentValidation)
   ├── LoggingBehavior records execution
   └── CreateOrderCommandHandler executes the use case
      │
      ▼ (Loads customer & catalog via repository abstractions)
4. [DOMAIN LAYER]
   ├── Domain Entity enforces business invariant (stock availability)
   ├── Order entity instantiated and OrderPlacedDomainEvent raised
      │
      ▼ (Handler calls IOrderRepository.AddAsync and IUnitOfWork.SaveChangesAsync)
5. [INFRASTRUCTURE LAYER]
   ├── OrderRepository and DbContext execute SQL INSERT
   ├── Outbox message saved atomically
      │
      ▼ (Success Result<Guid> returns to Application Handler)
6. [PRESENTATION LAYER]
   └── Endpoint returns HTTP 201 Created with { "orderId": "..." }
```

---

## 08 — Presentation Layer Folder Structure

A production-ready folder organization:

```text
Presentation/
├── Program.cs
├── appsettings.json
├── appsettings.Development.json
├── Controllers/
│   ├── OrdersController.cs
│   └── CustomersController.cs
├── Endpoints/
│   ├── Orders/
│   │   ├── CreateOrderEndpoint.cs
│   │   └── GetOrderByIdEndpoint.cs
│   └── Customers/
│       └── RegisterCustomerEndpoint.cs
├── Contracts/
│   ├── Common/
│   │   └── PagedResponse.cs
│   ├── Orders/
│   │   ├── CreateOrderRequest.cs
│   │   └── OrderResponse.cs
│   └── Customers/
│       ├── RegisterCustomerRequest.cs
│       └── CustomerResponse.cs
├── Middleware/
│   ├── CorrelationIdMiddleware.cs
│   └── GlobalExceptionHandler.cs
├── Extensions/
│   ├── SecurityExtensions.cs
│   ├── SwaggerExtensions.cs
│   └── ClaimsPrincipalExtensions.cs
└── DependencyInjection.cs
```

---

## 09 — Testing the Presentation Layer

Presentation Layer testing is performed via **Integration Testing** using `WebApplicationFactory<Program>`.

This exercises the complete ASP.NET Core hosting pipeline: model binding, serialization, routing, middleware, authentication, and HTTP status codes:

```csharp
// Presentation.IntegrationTests/Endpoints/CreateOrderEndpointTests.cs
using System.Net;
using System.Net.Http.Json;
using Microsoft.AspNetCore.Mvc.Testing;
using Xunit;

public class CreateOrderEndpointTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;

    public CreateOrderEndpointTests(WebApplicationFactory<Program> factory)
    {
        _client = factory.CreateClient();
    }

    [Fact]
    public async Task CreateOrder_WithInvalidPayload_ShouldReturn400BadRequest()
    {
        // Arrange
        var request = new CreateOrderRequest(
            Items: new List<CreateOrderItemRequest>(), // Empty items violates validation
            PromoCode: null
        );

        // Act
        var response = await _client.PostAsJsonAsync("/api/v1/orders", request);

        // Assert
        Assert.Equal(HttpStatusCode.BadRequest, response.StatusCode);
    }

    [Fact]
    public async Task GetOrderById_WhenNotFound_ShouldReturn404NotFound()
    {
        // Arrange
        var missingId = Guid.NewGuid();

        // Act
        var response = await _client.GetAsync($"/api/v1/orders/{missingId}");

        // Assert
        Assert.Equal(HttpStatusCode.NotFound, response.StatusCode);
    }
}
```

---

## 10 — Common Mistakes

### Mistake 1: Fat Controllers / Endpoints

```text
❌ Controller connects to database, checks business rules, computes taxes, sends email
✔ Controller does 3 things: binds request, sends command to Application layer, returns HTTP status
```

---

### Mistake 2: Exposing Domain Entities directly to external clients

```text
❌ return Ok(order); // Returns Domain Entity Order
✔ return Ok(new OrderResponse(order.Id, order.Total));
```

Returning entities leaks database structures, invites cyclic reference serialization bugs, and ties your public contract to your internal domain model.

---

### Mistake 3: Bypassing the Application Layer

```text
❌ Controller directly injects DbContext or IOrderRepository
✔ Controller sends commands/queries through the Application Layer
```

Bypassing the Application Layer eliminates cross-cutting validation, logging, transaction behaviors, and use-case reuse.

---

### Mistake 4: Non-standard error formats

```text
❌ Returning raw strings or custom arbitrary error JSON payloads
✔ Using standard RFC 7807 ProblemDetails
```

---

### Mistake 5: Hardcoding status codes or discarding CancellationTokens

```text
❌ Always returning 200 OK with { success: false, error: "..." }
✔ Using semantic HTTP status codes (201, 400, 401, 403, 404, 422) and passing CancellationToken to all async calls
```

---

## 11 — Mental Model

Think of the Presentation Layer as the **reception desk, security guard, and translator of an embassy**:

```text
Visitor (External Client)  → Arrives speaking their foreign language (HTTP / JSON / gRPC)
Security Guard (Middleware) → Checks credentials, IDs, badges (Auth & Correlation IDs)
Receptionist (Endpoint)     → Accepts their paper form, translates it into the internal protocol,
                              and hands it to the internal operations department (Application Layer)
```

The receptionist does not issue diplomatic passports or make foreign policy decisions.  
The receptionist only welcomes the visitor, checks the paperwork, and hands it over to the right department.

---

## 12 — Summary

```text
Presentation Layer
├── Outermost boundary and entry point of the system
├── Multi-channel: REST, Minimal APIs, gRPC, SignalR, CLI
├── Accepts external requests and validates transport format
├── Maps HTTP contracts to Application Commands and Queries
├── Dispatches use cases and returns semantic status codes
├── Formats standardized errors via RFC 7807 ProblemDetails
├── Houses HTTP middleware (Auth, Correlation IDs, Logging, CORS)
├── Acts as the Composition Root (Program.cs) to bootstrap the DI container
├── Tested via Integration Tests using WebApplicationFactory
└── Contains ZERO business logic and ZERO direct database queries
```

> **The Presentation Layer is the interface between the outside world and your application core. Keep it thin, protocol-focused, and decoupled from internal business logic.**

---
