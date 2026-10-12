# Architecture Evolution

## What Is It?

**Architecture Evolution** is the process of gradually changing a software system's architecture as business requirements, traffic, and team needs change.

The goal is to support change without unnecessary complexity or risky full rewrites.

## 1. Monolith-First Strategy

Start with a simple, well-structured application and evolve it when real requirements demand it.

- **Simple Monolith:** One application with a straightforward structure.
- **Modular Monolith:** One deployable application with clearly separated modules and controlled dependencies.
- **Microservices:** Independent services introduced when separate deployment, scaling, ownership, or isolation provides real value.

**Key principle:** Start simple, establish clear boundaries, and introduce distributed systems complexity only when justified.

## 2. Reversible Decisions

Architectural decisions have different costs of change.

- **Two-Way Door:** Easy or relatively inexpensive to reverse, such as changing internal class organization.
- **One-Way Door:** Expensive or difficult to reverse, such as splitting a tightly coupled database into independent databases.

**Rule:** Make reversible decisions quickly. Investigate expensive decisions carefully and keep options open when practical.

## 3. Core Migration Patterns

### Strangler Fig Pattern

Gradually replace a legacy system by routing specific features or requests to a new system until the old system can be retired.

### Branch by Abstraction

Introduce an interface or abstraction around an existing implementation, add the replacement, switch implementations safely, and remove the old one.

### Database Decomposition

Gradually reduce cross-module database dependencies before considering physically separate databases.

Typical steps include defining data ownership, separating schemas when useful, eliminating unnecessary cross-module joins, and restricting direct access between modules.

## 4. Architectural Drift & Fitness Functions

**Architectural Drift** occurs when changes gradually violate the system's intended architecture.

**Fitness Functions** are automated checks that enforce architectural rules.

Examples:

- Preventing circular dependencies.
- Preventing one module from referencing another module's internal implementation.
- Enforcing allowed project references.

These checks can run in the CI pipeline.

## 5. The Full Rewrite Trap

Rewriting an entire system can introduce regressions, lose hidden business knowledge, and delay delivering new features.

Prefer incremental refactoring when it offers a safer and more economical path.

A full rewrite may still be justified when evidence shows it is the better option.

## 6. When Should Architecture Evolve?

Evolve the architecture when real problems justify the change, such as:

- Strong coupling between modules.
- Independent deployment requirements.
- Uneven resource consumption.
- Database scalability limitations.
- Security or regulatory isolation requirements.
- Operational bottlenecks confirmed by measurement.

Do not adopt a new architecture merely because the codebase is large or another company uses it.

## Key Takeaway

**Start simple, maintain clear boundaries, measure real problems, and evolve incrementally.**

A good architecture is not designed to predict every future requirement; it is designed to accommodate change safely.