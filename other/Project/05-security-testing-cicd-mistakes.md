# Part 5 — Environments, Security, Performance, Testing, CI/CD, Git, Common Mistakes

---

## 24. Environment Configuration

- Three real environments: **Development, Staging, Production** — Staging
  should mirror Production's infrastructure shape (same Docker images,
  same Redis/DB topology conceptually) even if smaller-scale, so
  "works in Staging" is actually predictive of Production behavior.
- `ASPNETCORE_ENVIRONMENT` drives `appsettings.{Environment}.json` overlay
  — keep environment-specific files minimal (just the values that
  actually differ: connection strings, log levels, external URLs), not
  full duplicated config trees.
- **Feature flags** for anything you want to toggle without a redeploy —
  even a simple `IOptions<FeatureFlags>`-backed approach is fine at small
  scale; reach for a real feature-flag service (LaunchDarkly, or
  self-hosted like Unleash) only once you need per-user targeting or
  runtime toggling without a restart.

---

## 25. Security Best Practices

- **HTTPS enforced everywhere**, HSTS enabled in Production.
- **CORS**: explicit allowed-origins list, never `AllowAnyOrigin()` combined
  with `AllowCredentials()` (the combination the CORS spec itself
  forbids for good reason, but people still misconfigure it) — be
  explicit about exactly which origins can call the API.
- **Security headers** middleware: `X-Content-Type-Options: nosniff`,
  `X-Frame-Options: DENY`, `Content-Security-Policy` appropriate to your
  API (usually restrictive, since it's not serving HTML) — cheap to add,
  commonly skipped.
- **Input validation is not optional security** — FluentValidation
  (Part 2) plus EF Core parameterized queries (which you get "for free"
  as long as you never string-concatenate raw SQL) covers injection
  attacks. If you ever need raw SQL, use `FormattableString`-based
  `FromSqlInterpolated`, never string concatenation.
- **Least-privilege DB user** for the API's connection string — not the
  DB admin/`sa` account. Migrations can run under a more privileged
  account in CI/CD, separate from the runtime connection string.
- **Dependency scanning** in CI (`dotnet list package --vulnerable`, or
  GitHub's Dependabot) — catching a known-CVE package before it ships is
  far cheaper than after.
- **Secrets rotation plan** exists from day one even if manual initially —
  decide now how a compromised JWT signing key or DB password gets
  rotated, don't figure it out during an actual incident.

---

## 26. Performance Optimizations

- `.AsNoTracking()` on all read queries (mentioned in Part 3, worth
  repeating — it's the single highest-impact easy EF Core win).
- Projection (`.Select()` to DTOs) instead of loading full entity graphs
  when you only need a few fields — avoids over-fetching and unnecessary
  `Include()` chains.
- **Pagination on every list endpoint**, no exceptions — an unpaginated
  "get all orders" endpoint is a ticking time bomb the moment the table
  passes a few thousand rows. Decide your pagination shape (offset-based
  is fine for most admin/ERP UIs; keyset/cursor pagination only if you
  have genuinely large, high-churn datasets) once, apply consistently.
- `IHttpClientFactory` for all outbound HTTP calls (never `new HttpClient()`
  per call — the classic socket-exhaustion mistake) with Polly-based
  retry/circuit-breaker policies for calls to external/flaky services.
- Response compression (`AddResponseCompression`) enabled for API
  responses over HTTPS.
- Database indexes reviewed deliberately per query pattern — don't rely
  on "EF Core will figure it out"; review generated SQL for your hot-path
  queries (order list, search endpoints) early, not after a production
  slowdown.
- Async all the way down — no `.Result`/`.Wait()` blocking calls on async
  code anywhere (classic deadlock risk in ASP.NET Core request context).

---

## 27. Testing Strategy

Three layers, each with a distinct purpose — don't blur them:

- **Unit tests**: domain logic and handlers in isolation, dependencies
  faked/mocked (xUnit + NSubstitute or Moq + FluentAssertions). Fast,
  numerous, run on every save. Test business rules and edge cases here —
  "does applying a 10% discount to an already-discounted order throw."
- **Integration tests**: real `DbContext` against a real database (use
  **Testcontainers** to spin up an ephemeral SQL Server/PostgreSQL
  container per test run — not SQLite-in-memory, which behaves
  differently enough from your real provider to hide real bugs, and not a
  shared persistent test DB, which causes flaky/order-dependent tests).
  Test that your EF Core configuration, migrations, and queries actually
  work against the real engine.
- **Functional/End-to-end tests**: full HTTP pipeline via
  `WebApplicationFactory<Program>`, hitting real endpoints, asserting on
  real HTTP responses including status codes and headers — the closest
  simulation of an actual client calling your API, including
  auth/middleware/validation all wired together.
- `Architecture.Tests` (Part 1) is effectively a 4th category, worth
  keeping distinct — it tests structure, not behavior.
- Test naming: `MethodOrScenario_Condition_ExpectedResult` (e.g.
  `CreateOrder_WhenCustomerHasNoCreditLimit_ReturnsValidationError`) —
  consistent naming here matters because test names ARE the
  documentation of business rules in a large domain.
- Don't chase 100% coverage as a metric — prioritize coverage of business
  rules and edge cases over trivial getters/DTOs. Coverage percentage as a
  target incentivizes testing the wrong things.

---

## 28. CI/CD Recommendations

Pipeline stages, in order:
1. **Restore + Build** (fail fast on compile errors)
2. **Unit tests** (fast feedback)
3. **Static analysis** (`dotnet format --verify-no-changes`, Roslyn
   analyzers as build warnings/errors, `dotnet list package --vulnerable`)
4. **Integration tests** (Testcontainers-based — needs Docker-in-Docker
   or a Docker-capable runner)
5. **Build & push Docker image** (tag with git SHA + semantic version,
   never just `:latest` for anything deployed)
6. **Deploy to Staging** (automatic on merge to `main`/`develop`)
7. **Functional/smoke tests against Staging**
8. **Deploy to Production** (manual approval gate, or automatic with a
   feature-flag-guarded rollout — your call given team size, but a manual
   gate is the safer default for a small team without mature automated
   rollback tooling)

- GitHub Actions or Azure DevOps both work fine for a small team on
  Docker/K8s — pick based on where your code already lives (GitHub →
  GitHub Actions is the path of least friction).
- **Database migrations run as an explicit pipeline step**, separate from
  application deployment — never bundled silently into app startup (Part
  3) — so a migration can be reviewed/rolled back independently of the
  app version.

---

## 29. Git Branching Strategy & Commit Conventions

- **Trunk-based development with short-lived feature branches** —
  recommended over full GitFlow for a small team. GitFlow's
  develop/release/hotfix branch ceremony is overhead that mostly pays off
  for larger teams with scheduled release trains; a small team shipping
  continuously to Staging/Production benefits more from short branches
  merged frequently to `main`.
- Branch naming: `feature/orders-approval-workflow`,
  `fix/invoice-rounding-error`, `chore/upgrade-ef-core-9`.
- **Conventional Commits** (`feat:`, `fix:`, `refactor:`, `chore:`,
  `test:`, `docs:`) — enables automated changelog generation and makes
  `git log` genuinely useful for understanding history, not just noise.
- PRs required for `main`, even solo — forces a moment of self-review
  and keeps CI as a real gate rather than a formality.

---

## 30. Required NuGet Packages (by concern)

```
# Web / Minimal API
Asp.Versioning.Http

# CQRS / Validation
MediatR
FluentValidation
FluentValidation.DependencyInjectionExtensions

# EF Core
Microsoft.EntityFrameworkCore
Microsoft.EntityFrameworkCore.SqlServer   # or .Npgsql for PostgreSQL
Microsoft.EntityFrameworkCore.Design

# Logging
Serilog.AspNetCore
Serilog.Sinks.Console
Serilog.Sinks.Seq
Elastic.Serilog.Sinks
Serilog.Exceptions
Serilog.Enrichers.Environment

# Auth
Microsoft.AspNetCore.Authentication.JwtBearer
Microsoft.AspNetCore.Identity.EntityFrameworkCore

# Docs
Swashbuckle.AspNetCore

# Health Checks
AspNetCore.HealthChecks.SqlServer
AspNetCore.HealthChecks.Redis

# Background Jobs
Hangfire.AspNetCore
Hangfire.SqlServer

# Caching
StackExchange.Redis
Microsoft.Extensions.Caching.StackExchangeRedis

# Resilience
Microsoft.Extensions.Http.Polly

# Testing
xunit
xunit.runner.visualstudio
FluentAssertions
NSubstitute                    # or Moq
Testcontainers.MsSql           # or .PostgreSql
Microsoft.AspNetCore.Mvc.Testing
NetArchTest.Rules

# Utility
Mapster                        # or AutoMapper, for DTO projection where .Select() isn't convenient
```

---

## 31. Recommended Middleware Pipeline Order

Order matters and is a common source of subtle bugs:

```csharp
app.UseExceptionHandler();          // 1. catch everything below
app.UseHsts();                      // 2. (Production only)
app.UseHttpsRedirection();          // 3.
app.UseSerilogRequestLogging();     // 4. log after redirect, before routing
app.UseRouting();                   // 5.
app.UseCors();                      // 6. must be after UseRouting, before Auth
app.UseRateLimiter();               // 7. before auth — reject abuse before spending auth work
app.UseAuthentication();            // 8.
app.UseAuthorization();             // 9.
app.UseResponseCompression();       // can be earlier too; not order-critical relative to auth
// correlation-id middleware: register early (near top), it's a dependency
// for everything after it, including the exception handler's logging
app.MapControllers(); / endpoint mapping per module
```

---

## 32. Recommended Extension Methods & Startup Organization

`Program.cs` should read like a table of contents, not contain logic:

```csharp
var builder = WebApplication.CreateBuilder(args);

LoggingConfiguration.ConfigureBootstrapLogger();

builder
    .ConfigureSerilog()
    .AddPersistence()
    .AddAuthenticationAndAuthorization()
    .AddValidation()
    .AddApiVersioningSetup()
    .AddSwaggerDocs()
    .AddRateLimitingPolicies()
    .AddHealthChecks()
    .AddBackgroundJobs()
    .AddCachingLayer()
    .AddModules();   // discovers and registers every IModule

var app = builder.Build();

app.UseProductionPipeline();  // wraps the middleware order from #31
app.MapModuleEndpoints();     // each module maps its own endpoint group
app.MapHealthCheckEndpoints();

app.Run();
```

Each `Add*`/`Use*` extension lives in `VertexERP.Api/Configuration/`, one
file per concern, mirroring the `LoggingConfiguration` file you already
have — that file is the right template to replicate for every other
concern in this list.

---

## 33. Cross-Cutting Concerns — Where Each One Lives

| Concern | Implementation mechanism |
|---|---|
| Validation | MediatR pipeline behavior |
| Logging | Serilog + `ILogger<T>`, correlation ID middleware |
| Exception handling | `IExceptionHandler` |
| Transactions | MediatR pipeline behavior calling `SaveChangesAsync` once per command |
| Authorization | `IAuthorizationHandler` + custom attribute |
| Caching | `ICacheService` wrapper, used explicitly in query handlers |
| Audit logging | EF Core `SaveChangesInterceptor` |
| Correlation/tracing | Middleware + `LogContext` property |

The unifying principle: cross-cutting concerns belong in **pipeline
behaviors, middleware, or interceptors** — never copy-pasted into
individual handlers. If you find yourself writing the same `try/catch` or
the same `_logger.LogInformation(...)` at the start of every handler,
that's the signal it belongs in a behavior instead.

---

## 34. Common Mistakes to Avoid When Starting a New .NET Project

1. **Over-architecting for a team of one or two.** Full Clean Architecture
   with 4 projects per module, generic repositories, and full CQRS with
   separate read/write databases — for a small team, this is pure
   overhead paid on every single feature, forever. (This is the mistake
   this whole guide is steering you away from.)
2. **Under-architecting a genuinely large domain** — going full "just
   throw everything in one project with no module boundaries" because
   "we're small." Your domain is large even if your team isn't; skipping
   module boundaries here means the *codebase* becomes the bottleneck
   long before the team does.
3. **Skipping `Architecture.Tests`** — deciding on module boundaries in a
   design doc but never enforcing them in CI. Boundaries erode within
   months without automated enforcement; this is the single most common
   way "modular monolith" plans quietly fail.
4. **Generic repository + UoW over EF Core** — covered above, but worth
   repeating as its own line item because it's extremely common advice
   from outdated tutorials that predates modern EF Core's own maturity.
5. **Reading config via raw string keys** instead of the Options pattern
   — breaks silently on typos/renames.
6. **Auto-migrating the database on every app startup** in Production
   with no review gate.
7. **Using exceptions for expected business failures** instead of a
   Result type — performance and readability cost.
8. **No pagination on list endpoints** from day one — "we'll add it
   later" always means "we'll add it after an incident."
9. **Storing uploaded files on local/container disk** in a Kubernetes
   deployment — files vanish on pod reschedule.
10. **Sending email synchronously inline** in request handlers — couples
    API latency/reliability to a third-party provider's uptime.
11. **One giant shared `appsettings.json` with no options validation** —
    a missing key surfaces as a confusing null-reference deep in business
    logic instead of a clear startup failure.
12. **No correlation ID / structured logging discipline from day one** —
    retrofitting traceability into logs after the first production
    incident, when you actually need it, is exactly the wrong time to
    build it.
13. **Ignoring the "extract later" principle** — resist building anything
    (event bus, message queue, service mesh) that solves a
    microservices-scale problem you don't have yet. The modular monolith
    is specifically chosen so you CAN extract later; don't pre-pay for
    that future today.
