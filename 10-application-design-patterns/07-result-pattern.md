# Result Pattern

## 01 — What Is the Result Pattern?

The Result Pattern represents the outcome of an operation explicitly, indicating whether it succeeded or failed without relying on exceptions for expected business failures.

A Result typically contains a success or failure status, an optional value, and error information.

## 02 — Core Components

- **Success:** Indicates that the operation completed successfully.
- **Failure:** Indicates that the operation could not be completed.
- **Value:** Contains the returned data when the operation succeeds.
- **Error:** Describes the reason for failure, such as validation errors or a resource not being found.

Some implementations also include error codes, error categories, or multiple validation errors.

## 03 — Basic Request Flow

```text
API Endpoint
     |
     v
Request Handler
     |
     v
Business Logic
     |
     v
  Result<T>
   /     \
Success  Failure
   |        |
   v        v
Response   Error Response
```

The handler returns a result, and the API layer translates that result into the appropriate HTTP response.

## 04 — Common Use Cases

- Validation failures.
- Resource not found.
- Business rule violations.
- Duplicate records or other expected conflicts.
- Operations where callers need to handle success and failure explicitly.

## 05 — When to Use the Result Pattern and When Not to Use It

**Consider using it when:**
- Operations can fail for expected business reasons.
- Callers need to handle failures explicitly.
- You want consistent error handling across handlers and endpoints.
- Business failures should be returned without throwing exceptions.

**Avoid using it when:**
- It merely wraps values without providing meaningful behavior.
- It makes simple operations unnecessarily verbose.
- It is used to suppress unexpected infrastructure or programming errors instead of handling them appropriately.

### Who Decides?

The developer or architect decides based on the application's error-handling strategy and the types of failures its operations can produce.

Ask:
- Is this failure an expected outcome of the operation?
- Does the caller need to make a decision based on the failure?
- Can the API map the failure consistently to an appropriate HTTP status?
- Are unexpected errors still handled separately?

Use Results for expected failures and exception handling for unexpected errors.

## 06 — Important Considerations

- Define a consistent set of error categories and codes.
- Keep error information useful without exposing internal implementation details.
- Map failures to appropriate HTTP status codes at the API boundary; not every failure should return `400 Bad Request`.
- Avoid using exceptions for routine business failures when a Result is clearer.
- Unexpected failures, such as unhandled programming errors, should still flow through centralized exception handling.
- Decide whether operations return `Result` or `Result<T>` based on whether a value is needed.

## References

- [Microsoft — Error Handling in .NET](https://learn.microsoft.com/en-us/dotnet/standard/exceptions/)
- [Ardalis.Result](https://github.com/ardalis/Result)