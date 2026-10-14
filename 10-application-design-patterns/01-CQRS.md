# CQRS (Command Query Responsibility Segregation)

## 01 — What Is CQRS?

CQRS is an architectural pattern that separates operations that change application state from operations that retrieve data.

- **Command:** Represents an intention to change application state.
- **Query:** Represents a request to retrieve data without changing application state.

## 02 — Core Principles

- Commands represent actions such as creating, updating, or deleting data.
- Queries retrieve data such as individual records, lists, or reports.
- Each command or query has a clearly defined responsibility.
- Commands may return a result, such as a resource ID or operation status.
- Queries return the requested data.
- Queries should not modify business state or produce business side effects.

## 03 — Basic Request Flow

```text
Client
  |
  v
Application Endpoint
  |
  +---- Command ----> Command Handler
  |                       |
  |                       v
  |                 Business Logic
  |                       |
  |                       v
  |                   Database
  |
  +---- Query ------> Query Handler
                          |
                          v
                       Read Data
                          |
                          v
                       Response
```

## 04 — Read and Write Models

CQRS separates the responsibilities of reading and writing data.

- **Write model:** Handles state changes and enforces business rules.
- **Read model:** Provides the data required by queries, potentially using projections optimized for specific responses.

The read and write models can share the same database. Separate databases are optional.

When separate read and write data stores are synchronized asynchronously, the read model may temporarily contain stale data. This is known as **eventual consistency**.

## 05 — Benefits

- Clear separation of responsibilities.
- Easier organization of application use cases.
- Independent optimization of read and write operations.
- Flexible query responses.
- Improved maintainability for applications with complex business logic.

## 06 — When to Use CQRS and When Not to Use It

Consider CQRS when:

- **Complex business operations:** Write operations involve meaningful business rules, validation, or workflows.
- **Different read and write requirements:** Queries require different data structures or processing from commands.
- **Complex queries:** Read operations require specialized projections, reporting, or aggregation.
- **Growing application complexity:** Separating use cases makes the codebase easier to maintain.
- **Independent performance needs:** Read and write workloads require different optimization strategies.

Avoid introducing CQRS when:

- **Simple CRUD application:** Most operations are straightforward create, read, update, and delete operations.
- **Little business logic:** Separate handlers add boilerplate without improving clarity.
- **Limited complexity:** A simpler design already meets the application's requirements.
- **Unnecessary infrastructure:** Separate databases, messaging, or synchronization would add complexity without a real need.

### Who Decides?

The developer or architect responsible for the application's design makes the decision based on business requirements, technical constraints, performance needs, maintainability, and implementation costs.

Ask yourself:

1. Will separating commands and queries make the application easier to maintain?
2. Do read and write operations have meaningfully different requirements?
3. Will the benefits justify the additional complexity?

If there is no clear benefit, start with a simpler design and introduce CQRS when the requirements justify it.

## 07 — Important Considerations

- CQRS introduces additional structure and complexity.
- Separate databases are not required.
- CQRS does not inherently require event sourcing, microservices, or message brokers.
- Separate read and write data stores may introduce eventual consistency.
- Apply CQRS when the separation provides a meaningful benefit, not simply because it is a popular pattern.

## References

- [Microsoft — CQRS Pattern](https://learn.microsoft.com/en-us/azure/architecture/patterns/cqrs)
- [Martin Fowler — CQRS](https://martinfowler.com/bliki/CQRS.html)