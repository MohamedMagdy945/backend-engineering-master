# Repository Pattern

## 01 — What Is the Repository Pattern?

The Repository Pattern provides an abstraction for accessing and persisting data, separating data access logic from business logic.

It allows application code to work with a repository interface without depending directly on the underlying data access implementation.

## 02 — Core Responsibilities

- **Retrieve data:** Fetch entities or records.
- **Persist changes:** Add, update, or remove entities.
- **Encapsulate queries:** Keep reusable data access logic in one place.
- **Hide persistence details:** Separate application logic from the underlying database technology.

## 03 — Basic Structure

```text
Application / Business Logic
            |
            v
    Repository Interface
            |
            v
 Repository Implementation
            |
            v
       Data Storage
```

The application depends on the repository abstraction, while the infrastructure layer implements it using a database or another persistence mechanism.

## 04 — Core Principles

- Keep business rules outside repositories.
- Define repository abstractions around actual application needs.
- Keep each repository focused on a specific responsibility.
- Use Dependency Injection to provide implementations.
- Avoid unnecessary abstractions and excessive repository methods.

## 05 — When to Use the Repository Pattern and When Not to Use It

**Consider using it when:**
- Data access logic needs a clear boundary.
- Queries or persistence operations are complex or reusable.
- You want to isolate application code from persistence details.
- The abstraction improves maintainability or testability.

**Avoid adding it automatically when:**
- It only wraps simple Entity Framework Core operations without adding meaningful value.
- A generic repository duplicates functionality already provided by `DbContext` and `DbSet`.
- It introduces unnecessary layers or restricts useful EF Core features.

### Who Decides?

The developer or architect decides based on the application's requirements and the value of the abstraction.

Ask:
- Does it make data access easier to maintain?
- Does it provide a useful boundary between application and infrastructure?
- Does it simplify testing or encapsulate meaningful query logic?
- Is the benefit worth the additional code?

For many applications using EF Core, direct use of `DbContext` through an appropriate application or infrastructure boundary can be sufficient. A custom repository is optional, not mandatory.

## 06 — Important Considerations

- A repository is not a replacement for business logic or the database context.
- Avoid repositories that combine unrelated responsibilities.
- Be careful about exposing `IQueryable` across boundaries, as this can leak persistence details.
- Decide deliberately where `SaveChangesAsync` and transaction management belong.
- EF Core already provides repository-like and unit-of-work capabilities through `DbSet` and `DbContext`; additional abstractions should provide meaningful value.

## References

- [Microsoft — Persistence Layer Design](https://learn.microsoft.com/en-us/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)
- [Microsoft — DbContext Lifetime, Configuration, and Initialization](https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/)