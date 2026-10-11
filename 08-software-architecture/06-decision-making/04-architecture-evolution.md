# Architecture Evolution

A software architecture is never finished. As businesses grow, user traffic scales, and team structures change, architectures must evolve to support new realities.

The most catastrophic mistake in software engineering is treating architecture as a static, upfront plan:

```text
The Static Fallacy:
Design the "Final Architecture" on Day 1 ──► Suffer Over-Engineering for Years

The Evolutionary Reality:
Start Simple with Clear Boundaries ──► Measure Bottlenecks ──► Evolve Responsibly
```

An **Evolutionary Architecture** is designed to support guided, incremental change over time without requiring high-risk, expensive rewrites.

---

## 01 — The Natural Evolutionary Lifecycle

The vast majority of successful systems evolve through three distinct phases:

```text
Phase 1: Startup / MVP               Phase 2: Growth / Scale              Phase 3: Targeted Distribution
┌─────────────────────────┐          ┌─────────────────────────┐          ┌─────────────────────────┐
│     Simple Monolith     │          │    Modular Monolith     │          │   Targeted Extraction   │
│                         │          │                         │          │                         │
│  ┌───────────────────┐  │          │  ┌─────┐ ┌─────┐ ┌─────┐│          │  ┌─────┐ ┌─────┐        │
│  │  Vertical Slices  │  │ ───────► │  │ Mod │ │ Mod │ │ Mod ││ ───────► │  │ Mod │ │ Mod │ (Core) │
│  └───────────────────┘  │          │  └─────┘ └─────┘ └─────┘│          │  └─────┘ └─────┘        │
│                         │          │  Strict In-Process Bnds │          └────────────┬────────────┘
│  Single Database        │          │  Isolated DB Schemas    │                       │ Events
└─────────────────────────┘          └─────────────────────────┘          ┌────────────┴────────────┐
• Goal: Product-market fit           • Goal: Team autonomy                │   Extracted Service     │
• Zero network overhead              • High throughput, zero latency      │   (e.g., Payments / AI) │
• Instant refactoring                • Enforced boundaries                └─────────────────────────┘
```

### Why the "Monolith-First" Strategy Succeeds

Starting with microservices before domain boundaries are deeply understood creates **distributed monoliths**—the worst of all worlds:
- All the operational complexity of distributed networks.
- All the coupling of a spaghetti monolith.

By starting with a **Modular Monolith**, you discover natural business boundaries in-process where refactoring costs pennies. Once a boundary is proven and stable, extracting it into an independent service is straightforward.

---

## 02 — Reversibility: One-Way vs Two-Way Door Decisions

Architects classify decisions using the concept of reversibility:

```text
                    Architectural Decisions
                               │
       ┌───────────────────────┴───────────────────────┐
       ▼                                               ▼
Type 2: Two-Way Doors                           Type 1: One-Way Doors
(Reversible & Low Cost)                         (Irreversible & High Cost)
• Folder structure conventions                  • Primary database engine (Relational vs NoSQL)
• Internal class patterns (MediatR vs Service)  • Synchronous vs Asynchronous communication
• Caching strategies behind an interface        • Multi-tenancy isolation model (Schema vs DB)
• Library choices isolated in adapters          • Splitting a database into distributed stores
```

### The Evolutionary Rule:
> **Make Type 2 decisions quickly and boldly.**
> **Delay Type 1 decisions until empirical data forces your hand, and wrap them in abstractions that keep them as reversible as possible.**

---

## 03 — Core Migration Patterns

When an architecture must evolve, never perform a "big-bang" rewrite. Use established migration patterns that allow systems to evolve while running in production 24/7.

---

### Pattern 1: The Strangler Fig Pattern

The **Strangler Fig Pattern** replaces legacy architecture by gradually creating new capabilities around the edges until the old system has been completely superseded.

```text
Stage 1: Intercept Traffic
Client Request ──► [ Reverse Proxy / API Gateway (YARP / Nginx) ]
                          │
                          ├────────(95% traffic)────────► [ Legacy System ]
                          └────────(5% new route)───────► [ New Architecture ]

Stage 2: Expand New Services
Client Request ──► [ Reverse Proxy / API Gateway ]
                          │
                          ├────────(40% traffic)────────► [ Legacy System ]
                          └────────(60% traffic)────────► [ New Architecture ]

Stage 3: Decommission Legacy
Client Request ──► [ Reverse Proxy / API Gateway ]
                          │
                          └───────(100% traffic)────────► [ New Architecture ]
                                                          [ Legacy Deleted ]
```

#### Step-by-Step Execution:
1. Place a lightweight proxy (such as YARP, Envoy, or Cloudflare) in front of the existing application.
2. Build the new feature or refactored module in the modern architecture.
3. Update proxy routing rules to direct requests for that specific route to the new system.
4. Continue route by route until the legacy system receives zero traffic and can be safely retired.

---

### Pattern 2: Branch by Abstraction

When refactoring critical infrastructure inside an existing codebase (e.g., replacing Entity Framework with Dapper for performance, or swapping SendGrid for AWS SES), **Branch by Abstraction** enables continuous delivery without long-lived feature branches.

```text
Step 1: Introduce Abstraction
Caller ──► [ INotificationService ] ──► [ Legacy SendGrid Implementation ]

Step 2: Implement New Mechanism
Caller ──► [ INotificationService ] ──► [ Legacy SendGrid Implementation ]
                                    └─► [ New AWS SES Implementation ]

Step 3: Toggle via Feature Flag
Caller ──► [ INotificationService ]
                 │
                 ├─► If Flag == Off ──► [ Legacy SendGrid Implementation ]
                 └─► If Flag == On  ──► [ New AWS SES Implementation ]

Step 4: Cleanup
Caller ──► [ INotificationService ] ──► [ New AWS SES Implementation ]
(Legacy code deleted; flag removed)
```

This pattern avoids merge conflicts and lets you canary-test the new implementation on 1% of production traffic before rolling it out globally.

---

### Pattern 3: Database Decomposition (The Hardest Part of Evolution)

Splitting code is trivial compared to splitting databases. Relational databases with cross-table foreign keys, triggers, and joins resist evolution.

To evolve a database safely, follow this 5-step decomposition sequence:

```text
Step 1: Eliminate Database-Level Foreign Keys
        Replace physical FK constraints across module boundaries with logical ID references.
        (e.g., Order table stores CustomerId as a GUID, not a SQL FK to Customers table).

Step 2: Segregate Tables into Schemas
        Move tables into distinct PostgreSQL/SQL Server schemas:
        `ordering.orders`, `ordering.order_items`  vs  `billing.invoices`

Step 3: Eliminate Cross-Schema SQL Joins
        Rewrite queries so that no single SQL statement joins across the `ordering` and `billing` schemas.
        Perform data composition in the application layer or via read projections.

Step 4: Decouple Connection Strings
        Configure Module A and Module B with separate connection credentials so neither can query
        the other's schema even if they reside in the same database instance.

Step 5: Physical Database Separation
        Migrate the isolated schema to its own independent database server without changing
        any application code.
```

---

## 04 — Preventing Architectural Drift: Fitness Functions

Architectures naturally degrade over time if not actively protected. Under pressure, developers add quick shortcuts:
- An infrastructure repository gets injected directly into a domain entity.
- The `Catalog` module reaches into `Orders` tables.
- A circular dependency forms between two feature slices.

This gradual decay is called **Architectural Drift**.

### Enforcing Architectural Guardrails with Automated Tests

In an evolutionary architecture, **Fitness Functions** run on every pull request to guarantee that boundaries are respected:

```csharp
[Fact]
public void ModularMonolith_Modules_MustNotReferenceOtherModuleInternals()
{
    // Guarantees that Ordering module only accesses Catalog via its PublicApi
    var result = Types.InNamespace("MyApp.Modules.Ordering")
        .ShouldNot()
        .HaveDependencyOn("MyApp.Modules.Catalog.Internal")
        .GetResult();

    Assert.True(result.IsSuccessful, 
        "Ordering module cannot access internal Catalog classes!");
}

[Fact]
public void System_MustNotHave_CircularDependencies()
{
    // Prevents cyclic package dependencies that prevent modular evolution
    var result = Assemblies.InCurrentDomain()
        .Should()
        .NotHaveCircularDependencies()
        .GetResult();

    Assert.True(result.IsSuccessful, "System contains circular dependencies!");
}
```

If someone introduces an illegal dependency that compromises future evolution, the CI pipeline fails immediately.

---

## 05 — The "Full Rewrite" Trap

When an architecture accumulates technical debt, teams often demand:

```text
"This codebase is a mess. Let's pause feature development and rewrite it from scratch!"
```

In 90% of cases, the full rewrite fails. This is known as the **Second System Effect**:
1. **The moving target problem:** The old system must continue running and adding business features while the new system is being written. The new system is always playing catch-up.
2. **Hidden domain knowledge:** Thousands of subtle bug fixes and edge cases are baked into the legacy code. A rewrite strips them away, introducing regressions.
3. **Budget and morale exhaustion:** Rewrites almost always exceed initial time estimates by 2x–3x, burning engineering capital.

```text
       Strategy                 Risk                 Time to Value
──────────────────────    ─────────────────    ─────────────────────────
Full System Rewrite        Extremely High       Months/Years of zero value
Strangler Fig Evolution    Low & Predictable    Continuous incremental value
```

Always prefer **incremental evolutionary refactoring** over full rewrites.

---

## 06 — Real Triggers for Architectural Evolution

Evolve your architecture only when driven by empirical, measurable triggers:

| Genuine Trigger | What to Look For | Correct Evolution |
|---|---|---|
| **Deployment Contention** | Multiple teams are blocked waiting for release windows on the same monolith. | Split into a Modular Monolith or extract high-churn modules into independent services. |
| **Asymmetric Resource Bottlenecks** | Video encoding or AI generation spikes CPU to 100%, taking down the customer checkout API. | Extract the compute-heavy workload into an isolated background worker service. |
| **Compliance & Regulatory Isolation** | PCI-DSS payment compliance requires costly audits across the entire codebase. | Extract payment handling into an isolated microservice, drastically shrinking audit scope. |
| **Data Volume Mismatch** | Telemetry/IoT ingestion writes 50,000 events/sec, overwhelming the relational transactional DB. | Introduce Kafka/EventStore for high-throughput stream ingestion. |

### False Triggers to Ignore:
- ❌ *"Because Netflix or Google uses this architecture."* (You do not have their organizational scale).
- ❌ *"Because our monolith codebase has reached 100,000 lines of code."* (A well-structured modular monolith handles millions of lines cleanly).
- ❌ *"Because a new trendy technology framework was released."*

---

## 07 — Summary

- Architecture is an **ongoing evolutionary process**, not a one-time upfront design.
- Always prefer the **Monolith-First** strategy: refine domain boundaries in-process before distributing them across the network.
- Use the **Strangler Fig Pattern** and **Branch by Abstraction** to migrate live production systems without downtime or risky rewrites.
- Decouple **databases in stages**: remove FKs, isolate schemas, eliminate cross-schema joins, and only then split physically.
- Protect your system against architectural drift by embedding **Fitness Functions** directly into your automated CI test suite.
