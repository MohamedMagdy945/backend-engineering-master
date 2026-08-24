# VertexERP — .NET Production Setup Guide

A complete pre-implementation reference for building a large-domain,
small-team, Docker/Kubernetes-deployed .NET solution as a **pragmatic
Modular Monolith**.

Read in order the first time; use as a reference afterward.

1. `01-architecture-and-structure.md` — architecture decision, solution
   structure, naming conventions
2. `02-logging-errors-validation-auth.md` — Serilog/Seq/Elasticsearch,
   global exception handling, FluentValidation, JWT + permissions
3. `03-config-di-database-cqrs.md` — Options pattern, DI, EF Core,
   repository/UoW verdict, MediatR/CQRS, API versioning, Swagger, health
   checks, rate limiting
4. `04-caching-jobs-files-docker.md` — Redis, Hangfire, file storage,
   email, audit logging, localization, Docker/Compose
5. `05-security-testing-cicd-mistakes.md` — environments, security,
   performance, testing strategy, CI/CD, Git conventions, required NuGet
   packages, middleware order, common mistakes
6. `06-implementation-roadmap.md` — the exact build order, phase by phase

## The one-line summary of the core decision

Modular Monolith: module-per-bounded-context, light internal structure
(not full Clean Architecture per module), EF Core used directly (no
generic repository), MediatR for CQRS-as-organization (not CQRS-as-
infrastructure), enforced via `Architecture.Tests` from day one so
boundaries don't erode. Chosen specifically because your domain is large
but your team is small — this gives you separation where it matters
(between modules) without ceremony where it doesn't (inside them).
