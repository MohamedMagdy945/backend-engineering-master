# API Layer Structure

The API Layer is the entry point to the application. It exposes application functionality through HTTP endpoints and handles HTTP-specific concerns such as routing, authentication, authorization, middleware, and HTTP response mapping.

In a Minimal API project using Vertical Slice Architecture, endpoints are organized by feature and use case.

## Project Tree

```text
MyProject.Api/
│
├── Features/
│   ├── Products/
│   │   ├── GetProducts/
│   │   │   └── GetProductsEndpoint.cs
│   │   ├── GetProduct/
│   │   │   └── GetProductEndpoint.cs
│   │   └── CreateProduct/
│   │       └── CreateProductEndpoint.cs
│   │
│   └── Authentication/
│       ├── Login/
│       │   └── LoginEndpoint.cs
│       └── RefreshToken/
│           └── RefreshTokenEndpoint.cs
│
├── Common/
│   ├── Endpoints/
│   │   ├── IEndpoint.cs
│   │   └── EndpointExtensions.cs
│   ├── Middleware/
│   ├── Filters/
│   └── OpenApi/
│
├── Extensions/
│   ├── AuthenticationExtensions.cs
│   └── AuthorizationExtensions.cs
│
├── Configuration/
│   └── JwtOptions.cs
│
├── DependencyInjection.cs
└── Program.cs
```

## Folder Responsibilities

| Folder | Responsibility |
|---|---|
| `Features/` | Organizes HTTP endpoints by business feature and use case. |
| `Common/Endpoints/` | Contains shared endpoint contracts and endpoint registration or discovery logic. |
| `Common/Middleware/` | Contains custom middleware for cross-cutting HTTP concerns. |
| `Common/Filters/` | Contains reusable endpoint filters when needed. |
| `Common/OpenApi/` | Contains reusable OpenAPI configuration and documentation helpers. |
| `Extensions/` | Contains extension methods for configuring authentication, authorization, and the HTTP pipeline. |
| `Configuration/` | Contains API-specific configuration classes and options. |
| `DependencyInjection.cs` | Registers API-specific services and configures endpoint dependencies where appropriate. |
| `Program.cs` | Composes application startup, service registration, and the HTTP request pipeline. |

## Feature Structure

Each endpoint belongs to the feature and operation it exposes.

For example:

```text
Features/
└── Products/
    └── CreateProduct/
        └── CreateProductEndpoint.cs
```

The endpoint should:

1. Define the HTTP route and method.
2. Bind and validate HTTP input where appropriate.
3. Invoke the corresponding Application use case.
4. Map the application result to an HTTP status code and response.

The business workflow belongs in the Application Layer, not in the endpoint.

## Dependency Rules

- API can reference Application.
- API should not reference Infrastructure directly for business operations; infrastructure registration can be composed at the application startup boundary.
- API must not contain core business rules.
- API should not access the database directly.
- HTTP-specific concerns, such as status codes and route definitions, remain in API.

## Important Principles

- Use Minimal APIs when controllers are not required.
- Organize endpoints by feature rather than placing all endpoints in one global folder.
- Keep endpoint classes focused on HTTP handling.
- Centralize exception handling when consistent error responses are needed.
- Add middleware, filters, and configuration folders only when the project requires them.
- Avoid creating a separate file for every possible abstraction without a real need.

## Key Takeaway

The API Layer defines **how clients communicate with the application over HTTP**, while Application coordinates the use cases, Domain owns business rules, and Infrastructure implements technical capabilities.