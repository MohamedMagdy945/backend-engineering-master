# Domain Layer Structure

The Domain Layer contains the core business concepts and rules of the application. It represents the business independently of databases, frameworks, and external services.

## Project Tree

```text
MyProject.Domain/
│
├── Entities/
│   └── Product.cs
│
├── ValueObjects/
│   └── Money.cs
│
├── Enums/
│   └── OrderStatus.cs
│
├── DomainServices/
│   └── PricingService.cs
│
├── Events/
│   └── ProductCreatedEvent.cs
│
└── Exceptions/
    └── DomainException.cs
```

## Folder Responsibilities

| Folder | Responsibility |
|---|---|
| `Entities/` | Objects with identity and lifecycle, such as `Product` and `Order`. |
| `ValueObjects/` | Objects defined by their values rather than identity, such as `Money` and `Address`. |
| `Enums/` | Fixed sets of domain-related values, such as `OrderStatus`. |
| `DomainServices/` | Business rules that do not naturally belong to a single entity or value object. |
| `Events/` | Events representing business-significant occurrences within the domain. |
| `Exceptions/` | Domain-specific exceptions when exceptional conditions require them. |

## Dependency Rules

- Domain must not depend on Application, Infrastructure, or API.
- Domain should not depend on EF Core, ASP.NET Core, or other infrastructure frameworks.
- Domain logic should remain independent of persistence and external services.
- Other layers can reference Domain.

## Important Principles

- Keep business rules close to the entities and value objects they govern.
- Use entities when identity matters; use value objects when values define equality.
- Do not create a folder or abstraction unless the domain needs it.
- Prefer explicit business operations over public setters that allow invalid state changes.
- Use domain services only when a business operation does not naturally belong to one entity or value object.

## Example

A `Product` entity can enforce business rules such as preventing a negative price or invalid stock quantity. The Domain Layer defines these rules without knowing how the product is stored in the database.

## Key Takeaway

The Domain Layer models **what the business is and what rules it must follow**, independently of how the application exposes, processes, or persists that business.