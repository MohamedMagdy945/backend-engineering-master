# Part 4 — Caching, Background Jobs, File Upload, Email, Audit, Localization, Docker

---

## 17. Caching (Memory Cache / Redis)

- **In-memory (`IMemoryCache`)**: fine for single-instance, low-churn
  reference data (permission lists, lookup tables) — but remember it's
  per-pod, so with multiple Kubernetes replicas, each pod has its own
  potentially-stale copy. Fine for data that changes rarely; wrong choice
  for anything that needs cross-instance consistency.
- **Redis (`IDistributedCache` or `StackExchange.Redis` directly)**: use
  once you have more than one pod replica and need shared cache state —
  session-like data, computed aggregates, rate-limit counters if you go
  that route. Given your Docker/K8s target, plan for Redis from the start
  rather than retrofitting it later — it's a small addition now, an
  annoying refactor later once code has assumed in-memory semantics.
- **Cache invalidation strategy, decided upfront, not improvised per
  feature**: prefer short TTLs + cache-aside pattern over trying to
  precisely invalidate on every write. Precise invalidation across a
  modular monolith (module A caches data that module B's write
  invalidates) creates hidden coupling between modules — a cross-cutting
  concern that's genuinely hard to get right. Short TTL + accept slight
  staleness is usually the pragmatic choice for a small team.
- Wrap caching behind a small `ICacheService` interface in SharedKernel
  (`GetOrCreateAsync<T>(key, factory, ttl)`), not raw `IMemoryCache`/
  `IDistributedCache` calls scattered through handlers — lets you swap
  Memory→Redis later without touching business logic, and gives you one
  place to add cache-stampede protection if needed.

---

## 18. Background Jobs (Hangfire vs Quartz)

**Recommendation: Hangfire**, for your context specifically.
- Hangfire gives you a built-in dashboard (huge for a small team — you can
  *see* job history/failures without building your own tooling) and
  simpler persistence-based job storage (SQL Server/PostgreSQL — reuses
  infrastructure you already have, no new moving part).
- Quartz.NET is the better choice if you need complex cron-like scheduling
  semantics or are already in a pure "no extra DB table" philosophy — but
  for most ERP-style background work (nightly reports, invoice reminders,
  data cleanup), Hangfire's simplicity wins for small teams.
- Use **Hangfire's own storage schema** in your existing database
  (separate schema, e.g. `hangfire`) rather than a separate database — one
  less thing to provision/back up/monitor.
- Secure the Hangfire dashboard behind authorization (`IDashboardAuthorizationFilter`)
  — an unsecured `/hangfire` dashboard exposed publicly is a real, common
  misconfiguration that leaks internal job details and lets anyone trigger jobs.
- Each module can enqueue jobs (`IBackgroundJobClient.Enqueue<ISalesJobs>(...)`)
  without needing to know Hangfire specifics — wrap job scheduling behind
  an interface per module if you want to keep Hangfire itself swappable.

---

## 19. File Upload Strategy

- **Never store files on local container disk** as the primary store —
  Kubernetes pods are ephemeral; a file saved to local disk vanishes on
  pod restart/rescheduling. This is one of the most common file-upload
  mistakes when moving from "works on my machine" to containers.
- Use **object storage** (S3-compatible — MinIO for on-prem/self-hosted
  Docker/K8s setups, or actual S3/Azure Blob if you have cloud access) from
  day one, even in local dev (run MinIO as a docker-compose service —
  cheap, and keeps dev/prod parity).
- Abstract behind `IFileStorageService` (`UploadAsync`, `GetDownloadUrlAsync`
  returning a pre-signed URL, `DeleteAsync`) in SharedKernel — swappable
  implementation, and handlers never talk to S3 SDK directly.
- **Validate on the server, not just client-side**: file size limits,
  allowed content-types (check actual magic bytes, not just the extension
  or client-reported `Content-Type` — trivially spoofable), and scan for
  malware if handling user-uploaded documents in a business context
  (ClamAV via a sidecar or API is a reasonable option).
- Stream uploads (don't buffer the whole file in memory) for anything
  beyond small files — use `Request.Form.Files` with `MultipartReader` for
  large uploads to avoid memory pressure under load.

---

## 20. Email Service

- Abstract behind `IEmailService` (`SendAsync(EmailMessage message)`) —
  never call a provider SDK directly from business logic.
- Use a real transactional email provider (SendGrid, Postmark, Amazon SES,
  or SMTP relay like Mailgun) — not raw `SmtpClient` against a generic SMTP
  server for anything production-facing (deliverability, bounce handling,
  and tracking are genuinely hard problems these providers already solved).
- **Send emails via a background job (Hangfire), never inline in the
  request/response cycle.** An email provider outage or slowness should
  never make an API request hang or fail. Queue `SendWelcomeEmailJob`
  after the command handler commits — decouples email delivery from
  request latency entirely.
- Templates: use a proper templating approach (Razor-based email
  templates via `RazorLight`, or the provider's own template system) —
  not string concatenation/interpolation building HTML inline in C#.
- Local dev: use a fake SMTP catcher (Mailhog/Papercut, run via
  docker-compose) so developers never accidentally send real emails
  during testing.

---

## 21. Audit Logging

Distinct from Serilog request/error logging — audit logging is a
**business record** ("who changed this Customer's credit limit and
when"), not an operational log, and often has compliance/retention
requirements in ERP contexts.

- Implement via an EF Core `SaveChangesAsync` interceptor
  (`ISaveChangesInterceptor`) that inspects `ChangeTracker.Entries()` for
  entities implementing an `IAuditable` marker interface, capturing
  old/new values, the acting user (from `IHttpContextAccessor`/current
  user service), and timestamp — automatic, not something each handler
  remembers to call manually. Manual per-handler audit calls are how audit
  trails end up with gaps.
- Store audit records in their **own table/schema**, decoupled from the
  Serilog sink — audit data has different query patterns (by entity, by
  user, by date range) and different retention rules than operational logs,
  and shouldn't live in Elasticsearch mixed with debug-level noise.
- Decide upfront what "changed" means for sensitive fields — for things
  like salary or credit limit, log old/new *values*; for things like
  passwords, log *that* a change happened without the values, obviously.

---

## 22. Localization

- Only build this in from day one if you genuinely need multi-language
  now — retrofitting later is mostly mechanical (wrapping strings) and
  not worth the upfront complexity tax if you have a single-language user
  base today. If you DO need it:
- `IStringLocalizer<T>` + resource files (`.resx`) per module, not one
  giant shared resource file — keeps translations colocated with the
  feature they belong to, same reasoning as the rest of this structure.
- For validation messages (FluentValidation), use
  `.WithMessage(x => localizer["OrderAmountMustBePositive"])` rather than
  hardcoded English strings baked into validators.
- Set culture from the `Accept-Language` header via
  `RequestLocalizationOptions`, with a sane default fallback culture
  always configured explicitly.

---

## 23. Docker & Docker Compose

- **Multi-stage Dockerfile**: SDK image for build/publish stage, `aspnet`
  runtime image (not `sdk`) for the final stage — keeps final image small
  and reduces attack surface (no build tooling in the production image).
- Run as **non-root user** in the final image (`USER app` — the official
  ASP.NET Core images since .NET 8 include a non-root `app` user by
  default, use it explicitly).
- `docker-compose.yml` for local dev should stand up the **whole
  dependency graph**: API, SQL Server/PostgreSQL, Redis, Seq, MinIO,
  Mailhog — one `docker compose up` gets a new team member fully running
  with zero manual setup. This is worth investing real time in; it's the
  difference between a 10-minute and a 2-day onboarding for a new
  developer.
- Use `docker-compose.override.yml` for local-only tweaks (volume mounts
  for hot reload, exposed ports for debugging) so the base compose file
  stays deployment-representative.
- Health check directives in Dockerfile/compose (`HEALTHCHECK`) pointing
  at your `/health/live` endpoint — lets `docker compose up` and
  Kubernetes both know when the container is actually ready, not just
  "process started."

```dockerfile
# --- build stage ---
FROM mcr.microsoft.com/dotnet/sdk:9.0 AS build
WORKDIR /src
COPY Directory.Packages.props Directory.Build.props ./
COPY src/VertexERP.Api/VertexERP.Api.csproj src/VertexERP.Api/
# ... copy other .csproj files for layer caching
RUN dotnet restore src/VertexERP.Api/VertexERP.Api.csproj
COPY src/ src/
RUN dotnet publish src/VertexERP.Api/VertexERP.Api.csproj -c Release -o /app --no-restore

# --- runtime stage ---
FROM mcr.microsoft.com/dotnet/aspnet:9.0 AS runtime
WORKDIR /app
COPY --from=build /app .
USER app
HEALTHCHECK --interval=30s --timeout=5s CMD curl -f http://localhost:8080/health/live || exit 1
ENTRYPOINT ["dotnet", "VertexERP.Api.dll"]
```
