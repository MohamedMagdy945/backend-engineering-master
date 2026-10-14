# Specification Pattern

## 01 — What Is the Specification Pattern?

The Specification Pattern encapsulates a business rule or query condition into a reusable object that can be evaluated against an entity.

In data access, it is often used to organize complex filtering, sorting, and query criteria without duplicating the same conditions across multiple operations.

## 02 — Core Responsibilities

- **Encapsulate criteria:** Represent a condition, such as whether a product is active or in stock.
- **Reuse business rules:** Apply the same criteria in multiple use cases.
- **Compose conditions:** Combine specifications when the implementation supports it.
- **Separate query intent:** Keep complex filtering requirements organized outside individual handlers.

## 03 — Basic Structure

```text
Application / Use Case
         |
         v
   Specification
   (Query Criteria)
         |
         v
 Repository / Query Layer
         |
         v
      Database
```

The application defines or selects the required criteria, and the data access layer applies them to retrieve matching data.

## 04 — Common Use Cases

- Filtering products by category, price, and availability.
- Retrieving active customers who meet specific eligibility rules.
- Reusing complex search criteria across multiple use cases.
- Combining optional filters into a more maintainable query.

## 05 — When to Use the Specification Pattern and When Not to Use It

**Consider using it when:**
- The same complex criteria are reused across multiple use cases.
- Query conditions are growing difficult to maintain.
- Business rules need to be represented as reusable, testable criteria.
- Combining criteria would simplify complex queries.

**Avoid adding it when:**
- Queries contain only a few straightforward conditions.
- Each query is unique and easy to understand in its handler.
- Specifications introduce more indirection than they remove.
- A generic specification framework would be more complex than the actual requirements.

### Who Decides?

The developer or architect decides based on query complexity, reuse, and maintainability.

Ask:
- Is this condition reused?
- Is the query complex enough to benefit from a named abstraction?
- Does the specification represent a meaningful business rule or merely wrap a simple LINQ expression?
- Can the database provider translate the specification into an efficient query?

Start with straightforward LINQ queries and introduce specifications when they provide clear value.

## 06 — Important Considerations

- A specification is not necessarily a repository or a persistence mechanism.
- Business-rule specifications and database-query specifications can serve different purposes; keep their responsibilities clear.
- Ensure filtering and other query operations are translated to SQL when working with Entity Framework Core, rather than unintentionally loading entire tables into memory.
- Avoid building a large specification framework before the application needs it.
- Specifications can be combined, but composition should remain readable and preserve the intended logic.

## References

- [Martin Fowler — Specification](https://martinfowler.com/apsupp/spec.pdf)
- [Microsoft — Querying Data with EF Core](https://learn.microsoft.com/en-us/ef/core/querying/)