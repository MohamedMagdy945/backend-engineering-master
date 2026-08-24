# Part 3 — Configuration, DI, Database, Repository/UoW, CQRS, API Concerns

---

## 8. Configuration Management & Options Pattern

- Every cross-cutting concern gets its own `{Concern}Options` class bound
  via `IOptions<T>` (or `IOptionsSnapshot<T>` if it needs to reload without
  restart, `IOptionsMonitor<T>` if you need change notifications) —
  never read raw `IConfiguration["Key:SubKey"]` strings scattered through
  the codebase. This is one of the most common early mistakes: it works
  today, then someone renames a config key and three unrelated classes
  silently break at runtime with no compile-time signal.

```csharp
public class JwtOptions
{
    public const string SectionName = "Jwt";
    [Required] public string Issuer { get; init; } = default!;
    [Required] public string Audience { get; init; } = default!;
    [Required, MinLength(32)] public string SigningKey { get; init; } = default!;
    public int AccessTokenMinutes { get; init; } = 15;
}
```

- **Validate options at startup**, not at first use. Use
  `.AddOptions<JwtOptions>().Bind(...).ValidateDataAnnotations().ValidateOnStart();`
  so a missing/invalid config value fails the app at boot with a clear
  error — not three hours later when the first login attempt happens in
  production.
- **Secrets never in `appsettings.json`.** Local dev: .NET User Secrets.
  Docker/K8s: environment variables or mounted secrets, ideally via a real
  secret store (Azure Key Vault, HashiCorp Vault, or K8s Secrets +
  external-secrets-operator) rather than plain env vars for anything
  truly sensitive (DB passwords, JWT signing keys, API keys).
- `appsettings.{Environment}.json` for non-secret environment differences
  (log levels, feature flags, external endpoint URLs).

---

## 9. Dependency Injection Best Practices

- **Default to `Scoped`** for anything touching `DbContext` or per-request
  state. `Singleton` only for genuinely stateless or thread-safe services
  (HTTP clients via `IHttpClientFactory`, caches, options). `Transient`
  rarely needed explicitly — MediatR handlers are transient by default and
  that's correct.
- **One `AddXyz()` extension method per concern**, called from
  `Program.cs` in a clear, readable sequence — not 200 lines of raw
  `builder.Services.AddX()` calls inline. This is exactly the pattern your
  original `LoggingConfiguration` file already demonstrates — replicate it
  for every concern (`AddPersistence`, `AddAuthentication`, `AddSwaggerDocs`,
  `AddRateLimiting`, etc.), all living in `VertexERP.Api/Configuration/`.
- **Each module registers itself** via a small `IModule` interface
  (`RegisterServices(IServiceCollection)`, `MapEndpoints(WebApplication)`),
  discovered and invoked from `Program.cs` in a loop. This keeps
  `Program.cs` from needing to know the internals of every module — it
  just asks each module to wire itself up. This is the actual mechanism
  that makes "modular" real, not just a folder name.
- **Never use the Service Locator anti-pattern** (`app.Services.GetService<T>()`
  scattered through business logic) — inject dependencies through
  constructors always. The one legitimate exception is inside
  `Program.cs`/startup code itself.
- Avoid constructor over-injection (>5-6 dependencies) as a code smell —
  usually means a handler is doing too much and should be split.

---

## 10. Database Setup (EF Core, Migrations, Seeding)

- **One `DbContext` per module**, each with its own schema
  (`modelBuilder.HasDefaultSchema("sales")`) inside a *shared physical
  database* (simplest operationally for a small team) — this gives you
  logical separation matching your module boundaries without the
  operational cost of separate databases per module. You can split into
  separate databases later per module if/when you extract a service; the
  schema separation makes that migration much less painful.
- **Migrations per module**, each module's DbContext has its own
  `Migrations/` folder — apply them independently in the startup pipeline
  (loop over modules, call `.Database.MigrateAsync()` on each), so modules
  stay deployable/migratable independently even inside one monolith.
- **Never run `Database.Migrate()` automatically in Production startup**
  as a silent default — this is a common but risky shortcut. Prefer
  migrations applied via a controlled CI/CD step (or explicitly gated
  behind a startup flag/environment check) so a bad migration can't
  auto-apply against production data with no review gate.
- **Seeding**: separate "reference data" seeding (lookup tables,
  permissions list — safe to run every startup, idempotent) from "sample/
  dev data" seeding (only in Development environment, behind an explicit
  check). Conflating these is a common mistake that leads to test data
  leaking into staging/production.
- Use `IEntityTypeConfiguration<T>` classes (in `Infrastructure/Persistence/Configurations/`),
  never inline `OnModelCreating` fluent config for every entity in one
  giant method — doesn't scale past ~5 entities.

---

## 11. Repository Pattern & Unit of Work — should you use it?

**Recommendation: don't add a generic `IRepository<T>` on top of EF Core.
EF Core's `DbContext` + `DbSet<T>` already *is* a Repository + Unit of Work
implementation.** Wrapping it in your own generic repository interface is
one of the most common unnecessary-abstraction mistakes in .NET projects —
it:
- Adds a layer that just forwards to EF Core anyway (`Add`, `Update`, `Remove`, `FirstOrDefaultAsync`...).
- Actively hides genuinely useful EF Core features (`.Include()`,
  compiled queries, `.AsNoTracking()`, projection via `.Select()`) behind
  a leaky abstraction that eventually grows `IncludeCustomer(bool)`-style
  parameters to compensate.
- Doesn't actually make testing easier in practice — you still need to
  mock/fake it, and EF Core's in-memory or SQLite provider already gives
  you a realistic testable substitute for the real thing.

**What to do instead:**
- Inject `DbContext` (or better, a narrow interface exposing only what a
  given module needs — `ISalesDbContext` with just the `DbSet<T>`
  properties that module uses) directly into command/query handlers.
- If you want a *specific*, purposeful abstraction (not generic), write
  one interface per real use case — e.g. `IOrderRepository` with exactly
  the methods Orders actually needs (`GetByIdWithLinesAsync`,
  `GetOverdueOrdersAsync`) — not a generic `Repository<T>`. This is fine
  and sometimes worthwhile for aggregates with complex loading needs, but
  it's the exception, not the default for every entity.
- "Unit of Work" is just `DbContext.SaveChangesAsync()` — call it once at
  the end of the command handler. Don't build a custom `IUnitOfWork`
  wrapper around a class that already fills that role.

This isn't a purity argument — it's specifically about avoiding
maintenance burden for a small team. Big teams sometimes justify the
repository layer for stricter enforcement of persistence-ignorance; that
tradeoff doesn't pay for itself here.

---

## 12. CQRS and MediatR

- Yes, use MediatR — it's the right tool for the "vertical slice within a
  module" folder structure from Part 1, and it gives you pipeline behaviors
  (validation, logging, transactions) for free, applied consistently.
- **Don't split into separate read/write databases** ("full CQRS") unless
  you have an actual, measured need (heavy read load, complex reporting
  needs separate from OLTP). For most modules, Commands and Queries both
  hit the same `DbContext` — that's "CQRS as a code organization pattern,"
  which is what you want here, not "CQRS as an infrastructure pattern."
- Queries should use `.AsNoTracking()` always (change-tracking is pure
  overhead for read-only operations) and should return DTOs projected
  directly via `.Select()`, not full entities mapped after loading —
  avoids loading columns/relations you don't need.
- Standard pipeline behavior order:
  `Logging → Validation → Authorization (if not handled by [Authorize]) → Transaction/UnitOfWork → Handler`

---

## 13. API Versioning

- Use `Asp.Versioning.Http` (the actively maintained successor to the
  archived `Microsoft.AspNetCore.Mvc.Versioning`).
- URL segment versioning (`/api/v1/orders`) — most explicit and cache-
  friendly, easiest for API consumers to understand, and works cleanly
  with Minimal APIs' route groups. Header-based versioning is more
  "correct" by some REST purists' standards but meaningfully harder to
  document/test/debug in practice — not worth it for most teams.
- Version at the endpoint group level per module
  (`app.MapGroup("/api/v{version:apiVersion}/orders").HasApiVersion(1.0)`),
  not globally — different modules will need to version independently as
  the ERP evolves.

---

## 14. Swagger / OpenAPI

- Use `Swashbuckle.AspNetCore` (still the most mature option for full
  Swagger UI + versioning + JWT auth button integration), though .NET 9's
  built-in `Microsoft.AspNetCore.OpenApi` is worth watching as it matures.
- Configure **JWT bearer auth in Swagger UI** (`AddSecurityDefinition` +
  `AddSecurityRequirement`) so you can actually test authenticated
  endpoints from the docs page — trivial to add, frequently forgotten.
- Group by module/tag, one `SwaggerDoc` per API version.
- **Disable Swagger UI in Production** (or gate it behind auth) — exposing
  full API documentation publicly is a minor but real information-
  disclosure risk many teams forget to close off before going live.
- Add XML doc comments (`<summary>`) on endpoints and enable
  `GenerateDocumentationFile` in the API project — Swagger picks these up
  automatically for real endpoint descriptions instead of just type
  names.

---

## 15. Health Checks

- `Microsoft.Extensions.Diagnostics.HealthChecks` + provider packages:
  `AspNetCore.HealthChecks.SqlServer` (or Npgsql), `AspNetCore.HealthChecks.Redis`,
  `AspNetCore.HealthChecks.Seq`/Elasticsearch if you want to verify those too.
- Expose **two separate endpoints**, not one:
  - `/health/live` — liveness: is the process running at all? No
    dependency checks. Kubernetes uses this to decide whether to restart
    the pod.
  - `/health/ready` — readiness: are dependencies (DB, cache, message
    broker) actually reachable? Kubernetes uses this to decide whether to
    route traffic to the pod.
  Conflating these into one `/health` endpoint is a common mistake — it
  means a temporary DB blip causes Kubernetes to kill and restart a
  perfectly healthy process instead of just pulling it from the load
  balancer until the DB recovers.
- Tag each check (`"live"` vs `"ready"`) and filter which run per endpoint
  via `HealthCheckOptions.Predicate`.

---

## 16. Rate Limiting

- Use the built-in `Microsoft.AspNetCore.RateLimiting` middleware (.NET
  7+) — no third-party package needed for standard cases.
- Apply a **global fallback limiter** (fixed or sliding window) as a
  baseline, then **stricter, named policies** on sensitive endpoints
  (login, register, password reset — these are the actual
  brute-force/enumeration attack surface, and deserve tighter limits than
  general API traffic).
- Return `429 Too Many Requests` with a `Retry-After` header — the
  middleware does this by default, don't override it away.
- For multi-instance deployment (which Kubernetes implies), be aware
  in-memory rate limiting is **per-pod**, not global — if you truly need a
  cluster-wide limit, back it with Redis (e.g. via a distributed rate
  limiting library) rather than assuming the built-in in-memory limiter
  coordinates across pods, because it doesn't.
