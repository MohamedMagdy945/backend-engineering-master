# Pipeline Behaviors

## 01 — What Are Pipeline Behaviors?

Pipeline Behaviors allow you to execute shared logic before and after a request handler runs, without repeating that logic inside every handler.

They are commonly used with the Mediator Pattern to handle cross-cutting concerns consistently across application requests.

## 02 — How They Work

```text
Request
   |
   v
Pipeline Behavior 1
   |
   v
Pipeline Behavior 2
   |
   v
Request Handler
   |
   v
Response
```

Each behavior can execute logic before calling the next component, after it returns, or both.

A behavior can also stop the pipeline before the handler executes, for example, when validation fails.

## 03 — Common Use Cases

- **Validation:** Validate requests before executing business logic.
- **Logging:** Record request execution and relevant information.
- **Performance Monitoring:** Measure how long a request takes.
- **Transactions:** Define transaction boundaries when appropriate.
- **Authorization:** Enforce applicable access requirements.
- **Exception Handling:** Log or handle exceptions consistently.

## 04 — Execution Order

Pipeline behaviors execute in a defined order.

For example, if logging wraps validation and validation wraps the handler:

```text
Logging Before
    |
Validation Before
    |
Handler
    |
Validation After
    |
Logging After
```

If validation fails before calling the next component, the handler does not execute. The order in which behaviors are registered can therefore affect application behavior.

## 05 — Important Principles

- Keep shared concerns in behaviors instead of duplicating them across handlers.
- Keep business-specific rules in the appropriate handler or domain component.
- Avoid putting every application responsibility into the pipeline.
- Choose behavior ordering deliberately, especially for validation, authorization, transactions, and logging.
- Register behaviors through the mediator library's supported dependency injection configuration.

## References

- [MediatR — GitHub Repository](https://github.com/LuckyPennySoftware/MediatR)
- [Microsoft — Dependency Injection in .NET](https://learn.microsoft.com/en-us/dotnet/core/extensions/dependency-injection)