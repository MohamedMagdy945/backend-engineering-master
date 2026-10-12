
# Clean Architecture — Layer Structures

This section documents the recommended project structure for each layer in a .NET application using Clean Architecture and Vertical Slice Architecture.

## Layer Structures

- [API Layer](01-api-layer.md) — HTTP endpoints, middleware, authentication, and API configuration.
- [Application Layer](02-application-layer.md) — Use cases, handlers, validation, and application abstractions.
- [Domain Layer](03-domain-layer.md) — Entities, value objects, domain services, and business rules.
- [Infrastructure Layer](04-infrastructure-layer.md) — Database access, external services, and technical implementations.

## Solution Structure

```text
MyProject/
│
├── MyProject.Api/
├── MyProject.Application/
├── MyProject.Domain/
└── MyProject.Infrastructure/
```

## Architectural Responsibilities

| Layer | Main Responsibility |
|---|---|
| API | Exposes application functionality through HTTP. |
| Application | Coordinates use cases and application workflows. |
| Domain | Defines core business concepts and rules. |
| Infrastructure | Implements persistence and external technical capabilities. |

## Dependency Direction

```text
API ────────────────► Application
                           │
                           ▼
                         Domain
                           ▲
                           │
Infrastructure ────────────┘
```

The API invokes application use cases. Application depends on Domain, while Infrastructure implements the technical abstractions required by Application and accesses Domain types when needed.

The composition root wires the implementations together at startup.

## Architectural Guidelines

- Organize application code by feature and use case.
- Keep business rules independent of infrastructure concerns.
- Define abstractions in the layer that requires the capability.
- Keep infrastructure implementations separate from their abstractions.
- Avoid unnecessary folders, interfaces, and abstractions.
- Adapt the folder structure to the project's actual requirements rather than treating it as a mandatory template.

## Purpose

These structures provide a practical starting point for new .NET projects. They are guidelines, not rigid rules; add or remove folders according to the responsibilities and complexity of the application.