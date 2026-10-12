# Application Layer Structure

The Application Layer coordinates the application's use cases. It contains commands, queries, handlers, validation, and abstractions required to execute business operations.

It depends on the Domain Layer and defines the application workflows without depending on infrastructure implementations.

## Project Tree

```text
MyProject.Application/
│
├── Features/
│   ├── Products/
│   │   ├── GetProducts/
│   │   │   ├── GetProductsQuery.cs
│   │   │   ├── GetProductsHandler.cs
│   │   │   └── GetProductsResponse.cs
│   │   │
│   │   ├── GetProduct/
│   │   │   ├── GetProductQuery.cs
│   │   │   ├── GetProductHandler.cs
│   │   │   └── GetProductResponse.cs
│   │   │
│   │   └── CreateProduct/
│   │       ├── CreateProductCommand.cs
│   │       ├── CreateProductHandler.cs
│   │       ├── CreateProductValidator.cs
│   │       └── CreateProductResponse.cs
│   │
│   └── Orders/
│       └── CreateOrder/
│
├── Abstractions/
│   ├── Persistence/
│   └── Services/
│
├── Behaviors/
│   └── ValidationBehavior.cs
│
├── Common/
│   ├── Results/
│   └── Errors/
│
└── DependencyInjection.cs
```

## Folder Responsibilities

| Folder | Responsibility |
|---|---|
| `Features/` | Organizes application use cases by business feature and operation. |
| `Abstractions/` | Defines interfaces for infrastructure capabilities required by application workflows. |
| `Behaviors/` | Contains reusable pipeline behaviors, such as validation or logging, when using a mediator pipeline. |
| `Common/Results/` | Contains shared result types used to represent successful or failed operations. |
| `Common/Errors/` | Contains application-level errors and error definitions. |
| `DependencyInjection.cs` | Registers Application services and configures application-specific dependencies. |

## Feature Structure

Each use case has its own folder.

For example, `CreateProduct/` may contain:

- `CreateProductCommand.cs` — defines the input for the operation.
- `CreateProductHandler.cs` — coordinates the use case.
- `CreateProductValidator.cs` — validates the command's input.
- `CreateProductResponse.cs` — defines the output returned by the use case.

Queries represent operations that retrieve data. Commands represent operations that request a change.

Not every use case needs every file. Add only the files required by that operation.

## Dependency Rules

- Application can reference Domain.
- Application must not depend on API.
- Application must not depend on Infrastructure implementations.
- Application defines abstractions that Infrastructure can implement.
- Application coordinates business workflows, while Domain owns core business rules.

## Important Principles

- Organize code around use cases rather than technical categories.
- Keep each handler focused on one operation.
- Keep business rules in the Domain Layer when they govern domain state and behavior.
- Keep HTTP-specific concerns, such as status codes and endpoint routing, in the API Layer.
- Avoid creating generic abstractions until there is a real need for them.
- Add mediator behaviors only when the application uses a suitable pipeline mechanism.

## Key Takeaway

The Application Layer defines **what the application does**, coordinating domain behavior and required capabilities to execute each use case.