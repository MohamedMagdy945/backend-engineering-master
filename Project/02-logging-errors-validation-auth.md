# Part 2 — Logging, Exception Handling, Validation, Auth

---

## 4. Logging (Serilog + Seq + Elasticsearch)

You already have a strong starting point (reviewed earlier). Production
checklist on top of that:

- **Bootstrap logger → real logger swap** — keep this pattern, it's correct.
- **Always wrap `app.Run()`** in try/catch/finally with `Log.CloseAndFlush()`
  in `Program.cs`, so buffered Seq/Elasticsearch batches flush on shutdown
  (including on `SIGTERM` from Kubernetes — this matters more than people
  realize, since K8s gives pods a limited grace period to shut down).
- **Correlation ID middleware, explicitly wired**, pushing to
  `LogContext.PushProperty("CorrelationId", id)` per request — either
  accept an incoming `X-Correlation-Id` header or generate a GUID. Return
  it in the response header too, so client + server logs can be joined.
- **Environment/version enrichers**: `Enrich.WithProperty("Environment", ...)`
  and `Enrich.WithProperty("Version", assemblyVersion)` — critical once you
  have Staging + Production both writing to the same Elasticsearch cluster.
- **Never log secrets.** Add a Serilog destructuring policy or explicit
  `.Destructure.ByExcludingObsoleteProperties()` + manual care in request
  logging to ensure `Authorization` headers, passwords, tokens never land
  in logs. This is a real, common breach vector — treat it as a hard rule,
  not a nice-to-have.
- **Log levels discipline**: Information = business-relevant events only
  (order created, user logged in). Debug = technical detail, off by default
  in Production. Warning = handled-but-notable (validation failure,
  external API retry). Error = unhandled exception or failed operation
  requiring investigation. Overusing Information for everything makes logs
  useless at scale — this is the #1 logging mistake in real projects.
- **Per-module logging categories**: use `ILogger<T>` (typed per class),
  never a shared untyped `ILogger`. Lets you filter/configure log levels
  per-module in `appsettings.json` (`"Override": { "VertexERP.Modules.Sales": "Debug" }`).

---

## 5. Exception Handling & Global Error Handling

### Use the built-in .NET 8+ `IExceptionHandler`, not custom middleware

.NET 8 introduced `IExceptionHandler` as the standard mechanism —
prefer it over hand-rolled exception middleware (which was the pre-.NET-8
pattern and is now legacy style).

```csharp
public sealed class GlobalExceptionHandler(
    ILogger<GlobalExceptionHandler> logger) : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(
        HttpContext httpContext,
        Exception exception,
        CancellationToken cancellationToken)
    {
        var (statusCode, title) = exception switch
        {
            ValidationException => (StatusCodes.Status400BadRequest, "Validation failed"),
            NotFoundException   => (StatusCodes.Status404NotFound, "Resource not found"),
            ForbiddenException  => (StatusCodes.Status403Forbidden, "Forbidden"),
            _                   => (StatusCodes.Status500InternalServerError, "An unexpected error occurred")
        };

        logger.LogError(exception, "Unhandled exception: {Message}", exception.Message);

        httpContext.Response.StatusCode = statusCode;

        var problemDetails = new ProblemDetails
        {
            Status = statusCode,
            Title = title,
            Detail = statusCode == 500 ? null : exception.Message, // never leak internals on 500
            Instance = httpContext.Request.Path,
            Extensions = { ["correlationId"] = httpContext.TraceIdentifier }
        };

        if (exception is ValidationException validationEx)
        {
            problemDetails.Extensions["errors"] = validationEx.Errors;
        }

        await httpContext.Response.WriteAsJsonAsync(problemDetails, cancellationToken);
        return true;
    }
}
```

Register with `builder.Services.AddExceptionHandler<GlobalExceptionHandler>();
builder.Services.AddProblemDetails();` and `app.UseExceptionHandler();`.

### Why RFC 7807 ProblemDetails, not a custom error shape
`ProblemDetails` is the actual HTTP standard for API error responses (RFC
7807), natively supported by ASP.NET Core, understood by API tooling and
client generators. Inventing your own `{ success: false, error: "..." }`
shape is a common beginner mistake — it works, but throws away
interoperability for no benefit.

### A small hierarchy of domain exceptions (in SharedKernel)
`NotFoundException`, `ValidationException` (or reuse FluentValidation's),
`ForbiddenException`, `ConflictException` — map each to its correct HTTP
status in the handler above. Resist the urge to create 20 exception types;
5-6 well-named ones covering real HTTP semantics is enough for most APIs.

### Result pattern for expected failures, exceptions for unexpected ones
This is an important distinction many small teams get wrong: **don't use
exceptions for expected business-rule failures** ("insufficient stock",
"email already registered") — those are normal control flow, not
exceptional. Use a `Result<T>` / `Result` type (from SharedKernel) returned
by command/query handlers instead, and only throw real exceptions for
genuinely unexpected conditions. Exceptions are expensive (stack trace
capture) and using them for expected business outcomes is a real
performance and readability anti-pattern seen constantly in .NET codebases.

```csharp
public class Result<T>
{
    public bool IsSuccess { get; }
    public T? Value { get; }
    public Error? Error { get; }
    // factory methods: Result<T>.Success(value), Result<T>.Failure(error)
}
```

Your minimal API endpoint then maps `Result` → HTTP response explicitly
(200/201 on success, 400/404/409 based on `Error.Type` on failure) —
explicit, testable, no magic.

---

## 6. Validation (FluentValidation)

- One validator per command/query, colocated in the same folder (see Part 1
  folder structure) — not a separate top-level `Validators/` folder far
  from the command it validates.
- Register all validators in one line via assembly scanning:
  `services.AddValidatorsFromAssembly(typeof(SalesModule).Assembly);` per
  module — don't manually register each validator one by one.
- **Run validation via a MediatR pipeline behavior**, not manually inside
  each handler. This is the single highest-leverage FluentValidation setup
  decision: write it once, every command/query gets validated automatically,
  and handlers never contain `if (!ModelState.IsValid)`-style boilerplate.

```csharp
public class ValidationBehavior<TRequest, TResponse>(
    IEnumerable<IValidator<TRequest>> validators) : IPipelineBehavior<TRequest, TResponse>
    where TRequest : IRequest<TResponse>
{
    public async Task<TResponse> Handle(
        TRequest request, RequestHandlerDelegate<TResponse> next, CancellationToken ct)
    {
        var failures = (await Task.WhenAll(
                validators.Select(v => v.ValidateAsync(request, ct))))
            .SelectMany(r => r.Errors)
            .Where(f => f is not null)
            .ToList();

        if (failures.Count != 0)
            throw new Common.ValidationException(failures);

        return await next();
    }
}
```

- Keep validators about **shape and format** ("email is required and
  valid", "amount must be positive"). Business-rule validation that
  requires database lookups ("email must be unique", "customer must have
  active subscription") belongs in the handler itself or a domain
  service — don't inject `DbContext` into validators; it blurs the line
  between input validation and business logic and makes validators hard
  to unit test in isolation.

---

## 7. Authentication & Authorization (JWT, Identity, Roles, Permissions)

### Identity module as its own bounded context
Auth is genuinely its own domain in an ERP — put it in
`VertexERP.Modules.Identity`, using ASP.NET Core Identity for user/role
storage (battle-tested password hashing, lockout, token providers — don't
hand-roll this).

### JWT setup essentials
- Short-lived access tokens (15 min) + refresh tokens (stored hashed in DB,
  rotated on use, revocable). Never use long-lived access tokens "for
  convenience" — this is a common shortcut that becomes a real security
  liability.
- Sign with **asymmetric keys (RS256)**, not symmetric (HS256), once you
  have more than one module/service that needs to *validate* tokens
  without being able to *issue* them. For a monolith you can start with
  HS256 and a strong secret from a proper secret store — just don't
  hardcode it in `appsettings.json`.
- Validate `Issuer`, `Audience`, `Lifetime`, and `SigningKey` — all four,
  every time. Skipping audience/issuer validation is a common
  misconfiguration that quietly weakens the whole scheme.

### Roles vs Permissions — use claims-based permissions, not just roles
Roles alone (`[Authorize(Roles = "Admin")]`) don't scale for an ERP where
you'll need fine-grained checks like `orders.approve` or `invoices.void`.
Recommended shape:
- Roles are just named *groups of permissions* (e.g. "Sales Manager" =
  `orders.create, orders.approve, customers.view`).
- Permissions are baked into the JWT as claims at login time.
- Use a custom `[HasPermission("orders.approve")]` authorization attribute
  backed by an `IAuthorizationHandler`, not scattered `if (user.IsInRole(...))`
  checks inside handlers. Centralizing this means permission logic is
  testable and consistent, and adding a new permission never means hunting
  through business logic for role checks.

```csharp
public class PermissionRequirement(string permission) : IAuthorizationRequirement
{
    public string Permission { get; } = permission;
}

public class PermissionHandler : AuthorizationHandler<PermissionRequirement>
{
    protected override Task HandleRequirementAsync(
        AuthorizationHandlerContext context, PermissionRequirement requirement)
    {
        if (context.User.HasClaim("permission", requirement.Permission))
            context.Succeed(requirement);
        return Task.CompletedTask;
    }
}
```

- Store the canonical permission list as constants (`Permissions.Orders.Approve`),
  not magic strings scattered across attributes — one source of truth,
  refactor-safe.
