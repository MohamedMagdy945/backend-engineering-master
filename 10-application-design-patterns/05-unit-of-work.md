# Unit of Work Pattern

## 01 — What Is the Unit of Work Pattern?

The Unit of Work Pattern coordinates multiple data changes within a single logical operation, ensuring they are committed together as one unit.

It tracks changes made during an operation and coordinates their persistence, helping maintain data consistency.

## 02 — Core Responsibilities

- **Track changes:** Keep track of entities added, modified, or deleted.
- **Coordinate persistence:** Commit multiple changes together.
- **Manage transactions:** Ensure related database operations succeed or fail together when a transaction is required.
- **Maintain consistency:** Prevent partially persisted changes when operations must be atomic.

## 03 — Basic Structure

```text
Application / Business Logic
            |
            v
       Unit of Work
            |
            v
       Repositories
            |
            v
         Database
```

Multiple repositories can participate in the same Unit of Work so their changes are committed together.

## 04 — Core Principles

- Treat related data changes as one logical operation.
- Commit changes deliberately rather than independently after every modification.
- Use transactions when multiple database operations must be atomic.
- Keep transaction and persistence responsibilities clearly defined.
- Avoid unnecessary abstractions when the data access technology already provides these capabilities.

## 05 — When to Use the Unit of Work Pattern and When Not to Use It

**Consider using it when:**
- One business operation modifies data through multiple repositories.
- Multiple changes must be committed together.
- You need explicit coordination of persistence and transaction boundaries.
- The existing data access layer does not provide the required coordination.

**Avoid adding a custom implementation when:**
- Your ORM already provides the required functionality.
- It simply wraps existing methods without adding value.
- It creates unnecessary abstractions or duplicates transaction management.

### Who Decides?

The developer or architect decides based on the persistence technology and consistency requirements.

Ask:
- Must several changes succeed or fail together?
- Does the existing data access mechanism already coordinate these changes?
- Is a custom abstraction needed, or would it duplicate existing functionality?

With Entity Framework Core, `DbContext` already provides Unit of Work behavior by tracking changes and persisting them through `SaveChangesAsync`. A separate custom Unit of Work is therefore not automatically necessary.

## 06 — Important Considerations

- A Unit of Work is not the same as a database transaction, although it may coordinate one.
- A single `SaveChangesAsync` call is transactional for supported relational database providers under normal circumstances.
- Multiple `SaveChangesAsync` calls may require an explicit transaction if they must succeed or fail together.
- Repository and Unit of Work responsibilities should not overlap unnecessarily.
- Keep the Unit of Work scoped to the intended logical operation, typically using an appropriate `DbContext` lifetime.

## References

- [Microsoft — DbContext Lifetime, Configuration, and Initialization](https://learn.microsoft.com/en-us/ef/core/dbcontext-configuration/)
- [Microsoft — Transactions](https://learn.microsoft.com/en-us/ef/core/saving/transactions/)