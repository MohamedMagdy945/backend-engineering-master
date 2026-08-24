# .NET Production-Ready Project Setup Guide
### Part 1 — Architecture, Solution Structure, Naming Conventions

Context this guide assumes: large domain (ERP/CRM-like, multiple bounded contexts),
small team, Docker/Kubernetes deployment. Every recommendation below is chosen
*for that context* — not "best practice" in the abstract.

---

## 1. Architecture Decision: Modular Monolith (not Clean Architecture, not Microservices)

### The three options, honestly compared

**Full Clean Architecture (Domain/Application/Infrastructure/API as separate
class libraries, repeated per bounded context)**
Gives you strict dependency inversion and testability. Cost: for a solo/small
team, you pay a constant tax — every feature touches 4 projects, every
interface has exactly one implementation (so the abstraction buys you
nothing but ceremony), and onboarding requires understanding the whole
layering philosophy before writing a single endpoint. This is the classic
over-engineering trap for small teams on large domains. It shines with
large teams who need enforced boundaries between layers — not your case.

**Pure Vertical Slice (every feature is 100% self-contained, no shared
layers at all)**
Excellent for CRUD-heavy, low-domain-complexity apps. But an ERP/CRM has
*real* cross-cutting domain logic — a "Customer" entity is referenced by
Orders, Invoicing, and Support modules. Pure vertical slices with zero
shared domain model leads to duplicated entities and sync problems between
slices. Not ideal for a genuinely large, interconnected domain.

**Modular Monolith (recommended)**
You get bounded-context separation (each module owns its own domain model,
its own DbContext/schema, its own public contracts) *without* the
deployment complexity of microservices and *without* the layering tax of
per-module Clean Architecture. Inside each module, use light internal
structure (not 4 separate class libraries — just folders). Between modules,
communicate only through public interfaces or MediatR — never reach into
another module's internals or database tables directly.

### Why this is the correct call for you specifically

- **Large domain** → you need module boundaries, so bounded contexts matter.
  A single flat "Vertical Slice, no structure" approach would turn into a
  ball of mud as Sales/Inventory/HR/Finance modules all start touching each
  other's tables.
- **Small team** → you cannot afford the coordination overhead of real
  microservices (separate deploys, distributed transactions, network
  boundaries, service discovery) nor the ceremony of per-module Clean
  Architecture (4x the projects, 4x the DI wiring).
- **Docker/K8s deployment** → a modular monolith deploys as *one* container
  initially. If/when a specific module (say, Reporting) needs to scale
  independently later, its clean module boundary means it CAN be extracted
  into its own service without a rewrite — you're not backed into a corner.

This is deliberately the "boring, safe, extractable-later" choice. You are
not betting the project on getting DDD layering perfectly right on day one.

---

## 2. Solution Structure

```
VertexERP/
│
├── src/
│   ├── VertexERP.Api/                     # ASP.NET Core host — composition root only
│   │   ├── Program.cs
│   │   ├── Configuration/                 # extension methods: AddSerilog, AddSwagger, etc.
│   │   ├── Middleware/                    # global exception handler, correlation id, etc.
│   │   └── appsettings*.json
│   │
│   ├── VertexERP.Modules.Sales/           # one project per bounded context
│   │   ├── Domain/                        # entities, value objects, domain events
│   │   ├── Application/                   # CQRS handlers, DTOs, validators
│   │   ├── Infrastructure/                # EF Core config, external service impls
│   │   ├── Endpoints/                     # minimal API endpoint definitions
│   │   └── SalesModule.cs                 # public registration entry point (IModule)
│   │
│   ├── VertexERP.Modules.Inventory/       # same internal shape as above
│   ├── VertexERP.Modules.Identity/        # auth/users/roles module
│   │
│   ├── VertexERP.SharedKernel/            # truly shared: base entity, result type,
│   │                                       # domain event interfaces, guard clauses
│   │                                       # NOTE: contains NO business logic
│   │
│   └── VertexERP.Contracts/               # cross-module integration contracts
│                                           # (events/DTOs modules use to talk to
│                                           # each other WITHOUT direct references)
│
├── tests/
│   ├── VertexERP.Modules.Sales.UnitTests/
│   ├── VertexERP.Modules.Sales.IntegrationTests/
│   ├── VertexERP.Api.FunctionalTests/     # full pipeline, WebApplicationFactory
│   └── VertexERP.Architecture.Tests/      # NetArchTest — enforce module boundaries!
│
├── docker/
│   ├── Dockerfile
│   ├── docker-compose.yml
│   └── docker-compose.override.yml
│
├── .github/workflows/                     # or /azure-pipelines.yml
├── docs/
│   └── adr/                               # Architecture Decision Records
├── .editorconfig
├── Directory.Build.props                  # shared MSBuild settings across all projects
├── Directory.Packages.props               # central package version management
└── VertexERP.sln
```

### Why each piece is shaped this way

**One project per module, not per layer.** `VertexERP.Modules.Sales` is a
single class library containing its own Domain/Application/Infrastructure
*folders* — not separate projects. This is the "light internal structure"
compromise: you get organizational clarity without 4x the `.csproj` files
and project references to manage. If Sales ever needs to become a real
microservice, extracting one project is far easier than untangling folders
inside a shared project — but you don't pay that tax until you actually
need it.

**`SharedKernel` vs `Contracts` — an important, easy-to-blur distinction.**
- `SharedKernel`: technical building blocks with zero business meaning
  (base `Entity`, `Result<T>`, `IDomainEvent`, guard clause helpers). Every
  module can depend on this freely.
- `Contracts`: the *business* messages modules use to talk to each other
  (e.g. `OrderPlacedIntegrationEvent`, `CustomerCreditCheckRequest`). This
  is how Sales tells Inventory "reserve this stock" without Sales
  referencing Inventory's project directly. Keeping these separate stops
  `SharedKernel` from silently becoming a dumping ground for "shared"
  domain concepts that actually belong to one module.

**`VertexERP.Api` is a thin host, not where logic lives.** Its only jobs:
wire up DI/middleware/config (composition root) and register each module's
endpoints. If you ever need to split a module out, the Api project barely
changes.

**`Architecture.Tests` project — do this from day one, not "later."** Using
`NetArchTest.Rules` (NuGet), write tests like "Sales module must never
reference Inventory module's internals directly" and run them in CI. This
is the actual enforcement mechanism that keeps a modular monolith modular
— without it, module boundaries erode within 3 months as deadlines hit.
This single practice is what makes "modular monolith with a small team"
survivable; skip it and you'll silently degrade into a big ball of mud
that *happens* to have folders named after modules.

**`Directory.Packages.props` (Central Package Management).** One file
pins every NuGet package version across the whole solution. Prevents the
extremely common bug of two projects referencing different versions of
the same package and getting binary-runtime surprises. Standard in every
serious .NET 8+ repo now.

---

## 3. Naming Conventions

### Projects
`{SolutionName}.{Area}[.{SubArea}]`
- `VertexERP.Api`
- `VertexERP.Modules.Sales`
- `VertexERP.Modules.Sales.UnitTests` (test project mirrors the project it tests, suffix indicates test type)
- `VertexERP.SharedKernel`
- `VertexERP.Contracts`

### Namespaces
Mirror folder structure exactly (enable
`<ImplicitUsings>` but do NOT enable file-scoped-namespace auto-collapsing
incorrectly) — e.g. file at
`VertexERP.Modules.Sales/Application/Orders/Commands/CreateOrder.cs`
has namespace `VertexERP.Modules.Sales.Application.Orders.Commands`.
No exceptions, no "just dump it in the root namespace" shortcuts — this is
what makes a large solution navigable without a mental map.

### Folders (inside each module)
```
Domain/
  Entities/
  ValueObjects/
  Events/            # domain events (in-process, same module)
  Exceptions/         # domain-specific exceptions
Application/
  {AggregateName}/     # e.g. Orders/, Customers/ — one folder per aggregate
    Commands/
      CreateOrder/
        CreateOrderCommand.cs
        CreateOrderHandler.cs
        CreateOrderValidator.cs
    Queries/
      GetOrderById/
        GetOrderByIdQuery.cs
        GetOrderByIdHandler.cs
    DTOs/
Infrastructure/
  Persistence/
    Configurations/    # IEntityTypeConfiguration<T> per entity
    Migrations/
  ExternalServices/
Endpoints/
  OrderEndpoints.cs
```
This "vertical slice within the module" pattern (folder per
command/query, containing the command + handler + validator together) is
the modern MediatR convention — colocate everything related to one
operation instead of spreading it across Commands/, Handlers/, Validators/
top-level folders where related files live far apart.

### Classes & Files
- File name == public type name, always (`CreateOrderHandler.cs` contains
  `class CreateOrderHandler`). No multi-class files except tiny private
  nested helpers.
- Commands: verb + noun + `Command` → `CreateOrderCommand`,
  `CancelOrderCommand`
- Queries: `Get`/`List`/`Search` + noun + `Query` → `GetOrderByIdQuery`,
  `ListOrdersQuery`
- Handlers: `{CommandOrQueryName}Handler`
- Validators: `{CommandOrQueryName}Validator`
- DTOs / read models: `{Noun}Dto` or `{Noun}Response` — pick ONE
  convention and enforce it (recommendation: `Dto` for internal
  cross-layer shapes, `Response` for what actually serializes over the
  wire in the API contract, since they can diverge).
- Interfaces: `I{Noun}` — standard, no debate needed here.
- Async methods: always suffix `Async` (`GetByIdAsync`), no exceptions.
- Extension method classes: `{Target}Extensions` (e.g.
  `WebApplicationBuilderExtensions`, `ClaimsPrincipalExtensions`).
- Configuration/Options classes: `{Concern}Options` or `{Concern}Settings`
  — pick one (recommendation: `Options`, matching `IOptions<T>` naming so
  the type name and DI pattern agree — e.g. `JwtOptions`, not
  `JwtSettings` fighting `IOptions<JwtSettings>`).

### Why strict naming matters more here than in a small app
In a modular monolith with multiple bounded contexts, you WILL have
`Customer` conceptually appear in both Sales and Identity modules with
different shapes. Consistent, predictable naming + namespace-per-folder is
what lets you and future contributors instantly know which `Customer` a
given `using` statement refers to, without opening the file.
