# Part 6 — Step-by-Step Implementation Roadmap

Build in this exact order. Each phase produces something runnable/testable
before moving on — don't jump ahead to business logic until Phase 4 is
genuinely solid, since everything after depends on it being right.

---

### Phase 0 — Repository & Tooling Setup
1. Create the solution and empty projects matching the structure in Part 1.
2. Add `.editorconfig`, `Directory.Build.props` (nullable enabled, implicit
   usings, warnings-as-errors for the analyzers you care about),
   `Directory.Packages.props` (central package versions).
3. Set up Git repo, branch protection on `main`, commit convention
   documented in `CONTRIBUTING.md`.
4. Write the `Architecture.Tests` project NOW, even with just one or two
   rules ("Modules must not reference each other directly") — it's much
   easier to start clean than to retrofit boundary enforcement onto code
   that's already violating it.

### Phase 1 — Cross-Cutting Foundations (before any feature)
5. `SharedKernel`: base `Entity`, `AggregateRoot`, `IDomainEvent`,
   `Result<T>`/`Result`, `Error` type, guard clauses.
6. `Contracts` project: empty for now, will hold integration events as
   modules start needing them.
7. Logging: implement `LoggingConfiguration` as you already have,
   extended with correlation ID middleware.
8. Global exception handling: `IExceptionHandler` + `ProblemDetails` wiring.
9. Options pattern conventions established (one real example, e.g.
   `JwtOptions`, wired with `ValidateOnStart()`), so later modules copy an
   existing working pattern rather than inventing their own.

### Phase 2 — Identity Module First
This module needs to exist before any other module, since everything else
depends on "who is the current user."
10. `VertexERP.Modules.Identity`: ASP.NET Core Identity setup, `DbContext`,
    initial migration.
11. JWT issuing (login/refresh endpoints), permission claims in the token.
12. `PermissionRequirement`/`IAuthorizationHandler` + `Permissions` constants class.
13. Seed initial roles/permissions/admin user (Development-only seeding, per Part 3 #10).
14. Integration test: register → login → call a `[Authorize]`-protected
    test endpoint → confirm 200; confirm 401 without token. This proves
    the whole auth chain works before any real feature depends on it.

### Phase 3 — API Composition Root Wiring
15. `Program.cs` assembled per the extension-method structure in Part 5 §32.
16. Swagger with JWT auth button working end-to-end against the Identity
    module's login endpoint.
17. API versioning scaffolded (`v1` group), even with just a `/health` and
    an Identity endpoint mapped, to prove the pattern before other modules
    copy it.
18. Health checks (`/health/live`, `/health/ready`) wired to the database.
19. Rate limiting policies added, tested against the login endpoint
    specifically (this is the endpoint that most needs it).

### Phase 4 — MediatR Pipeline & Validation Infrastructure
20. MediatR registered, `ValidationBehavior` pipeline behavior implemented
    and tested against one real command (can reuse Identity's
    "ChangePassword" as the first real example).
21. Logging pipeline behavior (logs command name + duration).
22. Transaction/`SaveChanges` pipeline behavior.
23. Unit test the pipeline behaviors in isolation (fake handler, assert
    validation actually short-circuits before reaching it).

### Phase 5 — Docker/Compose & CI, Before Writing Business Features
It's tempting to defer this until "the app does something," but doing it
now means every feature you build from here on is developed against the
real target environment, not a laptop-only setup that surprises you later.
24. `Dockerfile` (multi-stage), `docker-compose.yml` with API + DB + Redis
    + Seq + MinIO + Mailhog.
25. `docker compose up` gets a fresh clone fully running — verify this
    literally, on a clean checkout.
26. CI pipeline: restore/build/unit-test stage working on every PR.
27. Health check endpoint wired into the Dockerfile `HEALTHCHECK` and
    verified.

### Phase 6 — First Real Business Module (proves the whole pattern)
Pick your simplest real bounded context (not the most complex one) as the
template the rest will copy.
28. Scaffold the module folder structure (Domain/Application/Infrastructure/Endpoints).
29. One aggregate, one Command (Create), one Query (GetById), full vertical
    slice: entity → EF configuration → migration → command handler →
    validator → endpoint → Swagger doc → unit test → integration test →
    functional test.
30. This slice is your **reference implementation** — once it's right,
    every subsequent feature in every module is "copy this pattern,"
    which is exactly the leverage a small team needs on a large domain.

### Phase 7 — Remaining Cross-Cutting Pieces, Added When First Actually Needed
Add these when a real feature needs them, not speculatively:
31. Caching (`ICacheService` + Redis) — when the first genuinely
    expensive/frequent query shows up.
32. Background jobs (Hangfire) — when the first async/scheduled need
    shows up (e.g. sending a confirmation email after order creation).
33. Email service — same trigger point as above, likely together.
34. File upload (`IFileStorageService` + MinIO/S3) — when the first
    document/attachment feature is needed.
35. Audit logging interceptor — before the first module with genuine
    compliance/traceability needs (likely Finance or Sales) ships.

### Phase 8 — Hardening Pass Before First Production Deploy
36. Security headers middleware, CORS locked to real origins, HSTS enabled.
37. Secrets moved out of any config file into your real secret store.
38. Swagger UI disabled or auth-gated outside Development.
39. Load-test the reference module's endpoints (even lightly, k6 or similar)
    to catch obvious N+1 query issues before they're everywhere.
40. Full test suite (unit + integration + functional + architecture) green
    in CI, migrations reviewed as an explicit deploy step, staging deploy
    validated end-to-end.

---

## Why this order, specifically

Notice that **Identity comes before any business module**, and the
**MediatR pipeline + one full reference slice comes before you build out
the rest of the domain**. This is deliberate: everything from Phase 6
onward is repetition of an already-proven pattern. If you instead started
writing Sales, Inventory, and Finance modules in parallel while still
figuring out how validation/logging/transactions should work, you'd end up
with three slightly different, inconsistent implementations of the same
cross-cutting concerns — exactly the kind of drift that's expensive to
unify later and easy to avoid by sequencing it this way from the start.
