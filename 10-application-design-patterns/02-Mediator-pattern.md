# Mediator Pattern

## 01 — What Is the Mediator Pattern?

The Mediator Pattern is a behavioral design pattern that centralizes communication between components through a mediator instead of allowing them to communicate directly with one another.

In backend applications, a mediator receives a request and dispatches it to the appropriate handler.

## 02 — Core Components

- **Mediator:** Receives requests and dispatches them to their handlers.
- **Request:** Represents an operation to be executed.
- **Handler:** Contains the logic for processing a specific request.
- **Pipeline Behavior:** Executes shared logic around request handling, such as validation, logging, or performance monitoring.

## 03 — Basic Request Flow

```text
Client
  |
  v
API Endpoint
  |
  v
Mediator.Send(request)
  |
  v
Request Handler
  |
  v
Business Logic
  |
  v
Response
```

The mediator handles request dispatching. The handler is responsible for executing the operation.

## 04 — Benefits

- **Reduced coupling:** The sender does not need a direct dependency on the concrete handler.
- **Clear responsibilities:** Each handler focuses on a specific request.
- **Centralized request dispatching:** Requests follow a consistent execution mechanism.
- **Extensibility:** Pipeline behaviors can add shared processing without duplicating code.
- **Maintainability:** Request handling becomes easier to organize in larger applications.

## 05 — When to Use the Mediator Pattern and When Not to Use It

Consider using the Mediator Pattern when:

- The application has many commands and queries that need consistent dispatching.
- You want endpoints to delegate operations without depending directly on handler implementations.
- You need pipeline behaviors for validation, logging, transactions, or performance monitoring.
- Request handling is becoming difficult to organize as the application grows.
- The pattern fits your existing architecture and improves maintainability.

Avoid introducing it when:

- The application has only a few straightforward operations.
- Directly calling a service or handler is already clear and maintainable.
- The mediator adds unnecessary abstractions and boilerplate.
- You are introducing it solely because a framework or tutorial uses it.

### Who Decides?

The developer or architect responsible for the application's design decides whether the mediator provides enough value to justify its additional abstraction.

Ask yourself:

1. Will it reduce unnecessary dependencies between request senders and handlers?
2. Will centralized dispatching simplify request handling?
3. Will pipeline behaviors provide value across multiple requests?
4. Is the benefit worth the additional abstraction and library dependency?

For a small application, direct calls may be sufficient. For a larger application with many independent use cases and shared pipeline requirements, a mediator may be beneficial.

## 06 — Important Considerations

- The Mediator Pattern does not replace business logic or dependency injection.
- It does not automatically implement CQRS; it can dispatch both commands and queries, but CQRS is a separate architectural decision.
- Pipeline behaviors are optional and depend on the mediator implementation.
- A mediator should dispatch requests, not become a place for business logic.
- Libraries such as MediatR provide a ready-made implementation, but the pattern itself does not require a specific library.

## References

- [MediatR — GitHub Repository](https://github.com/LuckyPennySoftware/MediatR)
- [Microsoft — Dependency Injection in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)