# 02 — Application Layer

The Application Layer is the layer that **orchestrates** the application's use cases.

It sits between the Domain Layer and the outer layers (Infrastructure, Presentation).

> **The Application Layer answers the question: What does the application do?**

---

## 01 — What Is the Application Layer?

The Application Layer contains the **use cases** of the application.

A use case is a specific operation that the application performs.

For example:

```text
Create Order
Cancel Order
Get Order By Id
Register Customer
Send Invoice
```

Each use case represents one thing the application can do.

The Application Layer is **not** about business rules — that belongs to the Domain.

The Application Layer is about **coordinating** the steps needed to complete an operation.

```text
Domain Layer
→ What are the rules?

Application Layer
→ What steps do we follow to complete this operation?
```

---

## 02 — Where Does the Application Layer Sit?

The Application Layer sits directly around the Domain Layer:

```text
┌──────────────────────────────────────┐
│          Presentation                │
│                                      │
│   ┌──────────────────────────────┐   │
│   │        Infrastructure        │   │
│   │                              │   │
│   │   ┌──────────────────────┐   │   │
│   │   │ ► Application ◄─────┼───┼── You are here
│   │   │                      │   │   │
│   │   │   ┌──────────────┐   │   │   │
│   │   │   │    Domain    │   │   │   │
│   │   │   └──────────────┘   │   │   │
│   │   └──────────────────────┘   │   │
│   └──────────────────────────────┘   │
└──────────────────────────────────────┘
```

It depends on the Domain Layer.
It does **not** depend on Infrastructure or Presentation.

```text
Application → Domain       ✔ Allowed
Application → Infrastructure  ❌ Not allowed
Application → Presentation    ❌ Not allowed
```

---

## 03 — What Belongs in the Application Layer?

```text
Application Layer
├── Use Cases / Command Handlers / Query Handlers
├── Application Services
├── Interfaces (Abstractions)
├── DTOs (Data Transfer Objects)
├── Validators
├── Mappers
└── Application Exceptions
```

Each of these has a specific purpose.

---

### Use Cases (Commands and Queries)

A use case is the primary unit of work in the Application Layer.

Using the CQRS pattern, use cases are often split into **Commands** and **Queries**:

```text
Command → Changes state (Create, Update, Delete)
Query   → Reads state (Get, List, Search)
```

Example Command:

```csharp
public class CreateOrderCommand
{
    public Guid CustomerId { get; set; }
    public List<OrderItemDto> Items { get; set; }
}
```

Example Query:

```csharp
public class GetOrderByIdQuery
{
    public Guid OrderId { get; set; }
}
```

---

### Command Handlers

A Command Handler executes a command (use case):

```csharp
public class CreateOrderCommandHandler
{
    private readonly IOrderRepository _orderRepository;
    private readonly IProductRepository _productRepository;

    public CreateOrderCommandHandler(
        IOrderRepository orderRepository,
        IProductRepository productRepository)
    {
        _orderRepository = orderRepository;
        _productRepository = productRepository;
    }

    public async Task<Guid> Handle(CreateOrderCommand command)
    {
        // 1. Create the domain entity
        var order = new Order(command.CustomerId);

        // 2. Add items (domain logic)
        foreach (var item in command.Items)
        {
            var product = await _productRepository.GetByIdAsync(item.ProductId);

            if (product is null)
                throw new NotFoundException($"Product {item.ProductId} not found.");

            order.AddItem(product, item.Quantity);
        }

        // 3. Persist through abstraction
        await _orderRepository.AddAsync(order);
        await _orderRepository.SaveChangesAsync();

        // 4. Return the result
        return order.Id;
    }
}
```

Notice what the handler does:

```text
1. Receives input (Command)
2. Calls Domain logic (order.AddItem)
3. Uses abstractions (IOrderRepository)
4. Returns a result

It does NOT:
❌ Access the database directly
❌ Know about HTTP
❌ Know about EF Core
❌ Implement business rules
```

---

### Query Handlers

A Query Handler retrieves data:

```csharp
public class GetOrderByIdQueryHandler
{
    private readonly IOrderRepository _orderRepository;

    public GetOrderByIdQueryHandler(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }

    public async Task<OrderDto?> Handle(GetOrderByIdQuery query)
    {
        var order = await _orderRepository.GetByIdAsync(query.OrderId);

        if (order is null)
            return null;

        return new OrderDto
        {
            Id = order.Id,
            CustomerId = order.CustomerId,
            Status = order.Status.ToString(),
            Items = order.Items.Select(i => new OrderItemDto
            {
                ProductId = i.ProductId,
                ProductName = i.ProductName,
                Price = i.Price.Amount,
                Quantity = i.Quantity
            }).ToList(),
            Total = order.GetTotal().Amount
        };
    }
}
```

---

### DTOs (Data Transfer Objects)

DTOs are simple objects used to transfer data between layers.

```csharp
public class OrderDto
{
    public Guid Id { get; set; }
    public Guid CustomerId { get; set; }
    public string Status { get; set; }
    public List<OrderItemDto> Items { get; set; }
    public decimal Total { get; set; }
}
```

```csharp
public class OrderItemDto
{
    public Guid ProductId { get; set; }
    public string ProductName { get; set; }
    public decimal Price { get; set; }
    public int Quantity { get; set; }
}
```

DTOs are **not** Entities. They have no behavior and no business rules.

```text
Entity (Domain)
├── Has identity
├── Has behavior
├── Enforces rules
└── Private setters

DTO (Application)
├── No identity
├── No behavior
├── No rules
└── Public setters
```

---

### Interfaces (Abstractions)

The Application Layer defines **interfaces** for capabilities it needs but does not implement.

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(Guid id);
    Task<List<Order>> GetByCustomerIdAsync(Guid customerId);
    Task AddAsync(Order order);
    Task SaveChangesAsync();
}
```

```csharp
public interface IEmailService
{
    Task SendAsync(string to, string subject, string body);
}
```

```csharp
public interface IPaymentGateway
{
    Task<PaymentResult> ChargeAsync(Guid orderId, Money amount);
}
```

These interfaces are implemented in the **Infrastructure Layer**:

```text
Application Layer          Infrastructure Layer
─────────────────          ────────────────────
IOrderRepository     ───►  OrderRepository (EF Core)
IEmailService        ───►  SmtpEmailService
IPaymentGateway      ───►  StripePaymentGateway
```

> **The Application Layer defines WHAT it needs. Infrastructure provides HOW.**

---

### Validators

Validators ensure the input to a use case is valid **before** executing it.

```csharp
public class CreateOrderCommandValidator
{
    public List<string> Validate(CreateOrderCommand command)
    {
        var errors = new List<string>();

        if (command.CustomerId == Guid.Empty)
            errors.Add("CustomerId is required.");

        if (command.Items is null || !command.Items.Any())
            errors.Add("At least one item is required.");

        foreach (var item in command.Items ?? Enumerable.Empty<OrderItemDto>())
        {
            if (item.ProductId == Guid.Empty)
                errors.Add("ProductId is required for each item.");

            if (item.Quantity <= 0)
                errors.Add("Quantity must be greater than zero.");
        }

        return errors;
    }
}
```

Important distinction:

```text
Application Validation (Application Layer)
→ Is the input well-formed?
→ Are required fields present?
→ Are values in valid format?

Business Validation (Domain Layer)
→ Can this order be confirmed?
→ Is the customer allowed to place an order?
→ Does the product have enough stock?
```

> **Application validation checks input format. Domain validation checks business rules.**

---

### Application Exceptions

Application-specific exceptions for cases like "not found" or "unauthorized":

```csharp
public class NotFoundException : Exception
{
    public NotFoundException(string message)
        : base(message)
    {
    }
}
```

```csharp
public class ValidationException : Exception
{
    public List<string> Errors { get; }

    public ValidationException(List<string> errors)
        : base("One or more validation errors occurred.")
    {
        Errors = errors;
    }
}
```

```csharp
public class ForbiddenException : Exception
{
    public ForbiddenException(string message)
        : base(message)
    {
    }
}
```

---

## 04 — The Application Layer Orchestrates

The key word for the Application Layer is **orchestration**.

It coordinates between:

```text
Input (Command/Query)
     ↓
Domain Logic (Entities, Value Objects)
     ↓
Abstractions (Repositories, Services)
     ↓
Output (DTO, Result)
```

A use case handler is like a **conductor**:

```text
The conductor does not play instruments.
The conductor tells the musicians what to do and when.

The Application Layer does not implement business rules.
The Application Layer tells the Domain what to do and when.
```

Example flow:

```csharp
public async Task<Guid> Handle(CreateOrderCommand command)
{
    // Step 1: Validate input
    var errors = _validator.Validate(command);
    if (errors.Any())
        throw new ValidationException(errors);

    // Step 2: Load required data
    var customer = await _customerRepository.GetByIdAsync(command.CustomerId);
    if (customer is null)
        throw new NotFoundException("Customer not found.");

    // Step 3: Execute domain logic
    var order = new Order(customer.Id);
    foreach (var item in command.Items)
    {
        var product = await _productRepository.GetByIdAsync(item.ProductId);
        if (product is null)
            throw new NotFoundException($"Product {item.ProductId} not found.");

        order.AddItem(product, item.Quantity);
    }

    // Step 4: Persist
    await _orderRepository.AddAsync(order);
    await _orderRepository.SaveChangesAsync();

    // Step 5: Side effects
    await _emailService.SendAsync(
        customer.Email,
        "Order Created",
        $"Your order {order.Id} has been created.");

    // Step 6: Return result
    return order.Id;
}
```

Each step has a clear responsibility:

```text
Step 1: Application concern (validation)
Step 2: Application concern (data loading)
Step 3: Domain concern (business logic)
Step 4: Infrastructure concern (persistence via abstraction)
Step 5: Infrastructure concern (email via abstraction)
Step 6: Application concern (return result)
```

---

## 05 — Application Layer Dependencies

The Application Layer depends on:

```text
✔ Domain Layer (Entities, Value Objects, Domain Events)
✔ Interfaces it defines (IOrderRepository, IEmailService)
```

The Application Layer does **not** depend on:

```text
❌ Infrastructure Layer
❌ Presentation Layer
❌ EF Core
❌ ASP.NET
❌ Specific database technology
❌ Specific email provider
```

Project reference structure:

```text
Application.csproj
├── References Domain.csproj    ✔
├── No reference to Infrastructure  ❌
└── No reference to Presentation    ❌
```

The dependency direction:

```text
Presentation → Application → Domain
                    ↑
              Infrastructure
```

Infrastructure implements the abstractions defined in the Application Layer.

---

## 06 — Application Layer vs Domain Layer

A common source of confusion:

```text
Where does this logic belong?
     ↓
Is it a business rule?
     ↓
Yes → Domain Layer
No  → Application Layer
```

Examples:

```text
"An order cannot have zero items"
→ Business Rule → Domain Layer

"Load the customer, create the order, save it, send an email"
→ Orchestration → Application Layer

"The total price includes a 10% tax"
→ Business Rule → Domain Layer

"Validate that CustomerId is not empty"
→ Input Validation → Application Layer

"A cancelled order cannot be confirmed"
→ Business Rule → Domain Layer

"Return a DTO with the order details"
→ Data Mapping → Application Layer
```

Simple guideline:

```text
Domain Layer
→ Rules that exist even without the application

Application Layer
→ Operations that the application performs
```

---

## 07 — Application Services vs Use Case Handlers

There are two common patterns for organizing the Application Layer:

### Application Services

Group related operations into a single service:

```csharp
public class OrderService
{
    private readonly IOrderRepository _orderRepository;

    public OrderService(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }

    public async Task<Guid> CreateOrder(CreateOrderCommand command) { /* ... */ }
    public async Task CancelOrder(Guid orderId) { /* ... */ }
    public async Task<OrderDto?> GetOrderById(Guid orderId) { /* ... */ }
}
```

### Use Case Handlers (one class per use case)

Each use case has its own handler:

```csharp
public class CreateOrderCommandHandler { /* ... */ }
public class CancelOrderCommandHandler { /* ... */ }
public class GetOrderByIdQueryHandler { /* ... */ }
```

Comparison:

```text
Application Services
├── Fewer classes
├── Related operations grouped together
├── Can become large over time
└── Simpler for small applications

Use Case Handlers
├── More classes
├── Each handler has a single responsibility
├── Easier to maintain as the application grows
└── Works well with CQRS and MediatR
```

Both approaches are valid. The important thing is consistency.

---

## 08 — Using MediatR (Optional Pattern)

MediatR is a popular library that implements the Mediator pattern for use case handlers.

Define a request:

```csharp
public class CreateOrderCommand : IRequest<Guid>
{
    public Guid CustomerId { get; set; }
    public List<OrderItemDto> Items { get; set; }
}
```

Define a handler:

```csharp
public class CreateOrderCommandHandler : IRequestHandler<CreateOrderCommand, Guid>
{
    private readonly IOrderRepository _orderRepository;

    public CreateOrderCommandHandler(IOrderRepository orderRepository)
    {
        _orderRepository = orderRepository;
    }

    public async Task<Guid> Handle(
        CreateOrderCommand request,
        CancellationToken cancellationToken)
    {
        var order = new Order(request.CustomerId);

        // ... add items, validate, etc.

        await _orderRepository.AddAsync(order);
        await _orderRepository.SaveChangesAsync();

        return order.Id;
    }
}
```

The Presentation Layer sends the command through MediatR:

```csharp
[HttpPost]
public async Task<IActionResult> CreateOrder(CreateOrderCommand command)
{
    var orderId = await _mediator.Send(command);
    return CreatedAtAction(nameof(GetOrderById), new { id = orderId }, orderId);
}
```

Benefits:

```text
✔ Decouples Presentation from Application
✔ Each handler is a single class
✔ Easy to add cross-cutting concerns (logging, validation) via pipeline behaviors
```

> **MediatR is optional. The Application Layer pattern works with or without it.**

---

## 09 — Cross-Cutting Concerns with Pipeline Behaviors

When using MediatR, you can add behaviors that run before/after every use case:

### Logging Behavior

```csharp
public class LoggingBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
{
    private readonly ILogger<LoggingBehavior<TRequest, TResponse>> _logger;

    public LoggingBehavior(ILogger<LoggingBehavior<TRequest, TResponse>> logger)
    {
        _logger = logger;
    }

    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        _logger.LogInformation("Handling {RequestName}", typeof(TRequest).Name);

        var response = await next();

        _logger.LogInformation("Handled {RequestName}", typeof(TRequest).Name);

        return response;
    }
}
```

### Validation Behavior

```csharp
public class ValidationBehavior<TRequest, TResponse>
    : IPipelineBehavior<TRequest, TResponse>
{
    private readonly IEnumerable<IValidator<TRequest>> _validators;

    public ValidationBehavior(IEnumerable<IValidator<TRequest>> validators)
    {
        _validators = validators;
    }

    public async Task<TResponse> Handle(
        TRequest request,
        RequestHandlerDelegate<TResponse> next,
        CancellationToken cancellationToken)
    {
        var failures = _validators
            .Select(v => v.Validate(request))
            .SelectMany(result => result.Errors)
            .Where(error => error is not null)
            .ToList();

        if (failures.Any())
            throw new ValidationException(failures);

        return await next();
    }
}
```

The pipeline:

```text
Request
  ↓
Logging Behavior
  ↓
Validation Behavior
  ↓
Handler
  ↓
Response
```

---

## 10 — Application Layer Folder Structure

A typical Application Layer structure:

```text
Application/
├── Common/
│   ├── Interfaces/
│   │   ├── IOrderRepository.cs
│   │   ├── IProductRepository.cs
│   │   ├── IEmailService.cs
│   │   └── IPaymentGateway.cs
│   ├── Exceptions/
│   │   ├── NotFoundException.cs
│   │   ├── ValidationException.cs
│   │   └── ForbiddenException.cs
│   ├── Behaviors/
│   │   ├── LoggingBehavior.cs
│   │   └── ValidationBehavior.cs
│   └── Mappings/
│       └── MappingProfile.cs
├── Orders/
│   ├── Commands/
│   │   ├── CreateOrder/
│   │   │   ├── CreateOrderCommand.cs
│   │   │   ├── CreateOrderCommandHandler.cs
│   │   │   └── CreateOrderCommandValidator.cs
│   │   └── CancelOrder/
│   │       ├── CancelOrderCommand.cs
│   │       ├── CancelOrderCommandHandler.cs
│   │       └── CancelOrderCommandValidator.cs
│   ├── Queries/
│   │   └── GetOrderById/
│   │       ├── GetOrderByIdQuery.cs
│   │       ├── GetOrderByIdQueryHandler.cs
│   │       └── OrderDto.cs
│   └── DTOs/
│       └── OrderItemDto.cs
└── Customers/
    ├── Commands/
    │   └── RegisterCustomer/
    │       ├── RegisterCustomerCommand.cs
    │       └── RegisterCustomerCommandHandler.cs
    └── Queries/
        └── GetCustomerById/
            ├── GetCustomerByIdQuery.cs
            └── CustomerDto.cs
```

This structure groups by **feature** (Orders, Customers) then by **type** (Commands, Queries).

---

## 11 — Application Layer and Testing

Application Layer tests typically require **mocking** the abstractions:

```csharp
[Fact]
public async Task CreateOrder_ShouldReturnOrderId()
{
    // Arrange
    var mockOrderRepo = new Mock<IOrderRepository>();
    var mockProductRepo = new Mock<IProductRepository>();

    var product = new Product("Laptop", new Money(999.99m, "USD"));
    mockProductRepo
        .Setup(r => r.GetByIdAsync(It.IsAny<Guid>()))
        .ReturnsAsync(product);

    var handler = new CreateOrderCommandHandler(
        mockOrderRepo.Object,
        mockProductRepo.Object);

    var command = new CreateOrderCommand
    {
        CustomerId = Guid.NewGuid(),
        Items = new List<OrderItemDto>
        {
            new() { ProductId = Guid.NewGuid(), Quantity = 2 }
        }
    };

    // Act
    var orderId = await handler.Handle(command);

    // Assert
    Assert.NotEqual(Guid.Empty, orderId);
    mockOrderRepo.Verify(r => r.AddAsync(It.IsAny<Order>()), Times.Once);
    mockOrderRepo.Verify(r => r.SaveChangesAsync(), Times.Once);
}
```

```csharp
[Fact]
public async Task CreateOrder_ShouldThrow_WhenProductNotFound()
{
    // Arrange
    var mockOrderRepo = new Mock<IOrderRepository>();
    var mockProductRepo = new Mock<IProductRepository>();

    mockProductRepo
        .Setup(r => r.GetByIdAsync(It.IsAny<Guid>()))
        .ReturnsAsync((Product?)null);

    var handler = new CreateOrderCommandHandler(
        mockOrderRepo.Object,
        mockProductRepo.Object);

    var command = new CreateOrderCommand
    {
        CustomerId = Guid.NewGuid(),
        Items = new List<OrderItemDto>
        {
            new() { ProductId = Guid.NewGuid(), Quantity = 1 }
        }
    };

    // Act & Assert
    await Assert.ThrowsAsync<NotFoundException>(
        () => handler.Handle(command));
}
```

Comparison with Domain Layer testing:

```text
Domain Layer Tests
├── No mocks needed
├── Test business rules directly
└── Pure unit tests

Application Layer Tests
├── Mocks for abstractions (repositories, services)
├── Test orchestration and flow
└── Integration-style unit tests
```

---

## 12 — Common Mistakes

### Mistake 1: Business rules in the Application Layer

```text
❌ Handler checks "can this order be cancelled?"
✔ Order Entity checks "can I be cancelled?"
```

The handler should call `order.Cancel()` — the entity decides if it is valid.

---

### Mistake 2: Application Layer depends on Infrastructure

```text
❌ Application references EF Core
❌ Handler uses DbContext directly
✔ Application uses IOrderRepository
```

The Application Layer should only use abstractions.

---

### Mistake 3: Returning Domain Entities from handlers

```text
❌ Handler returns Order (Entity)
✔ Handler returns OrderDto (DTO)
```

Returning entities exposes internal domain state to outer layers.

---

### Mistake 4: Fat handlers with mixed concerns

```text
❌ Handler does validation + business logic + persistence + logging
✔ Validation in a validator / behavior
✔ Business logic in the Domain
✔ Logging in a behavior
✔ Handler only orchestrates
```

---

### Mistake 5: One giant Application Service

```text
❌ OrderService with 30 methods
✔ One handler per use case
```

Keep handlers focused on a single operation.

---

## 13 — Mental Model

Think of the Application Layer as the **manager** of the application:

```text
The manager does not do the work.
The manager assigns work to the right people.
The manager coordinates the workflow.
```

```text
Application Layer
├── Receives a request (Command/Query)
├── Validates the input
├── Loads required data (via abstractions)
├── Delegates to Domain logic
├── Triggers side effects (via abstractions)
└── Returns the result
```

The Application Layer is the **glue** between the outside world and the Domain:

```text
Outside World
      ↓
  Application Layer (orchestrates)
      ↓
  Domain Layer (business rules)
      ↓
  Infrastructure (via abstractions)
```

---

## 14 — Summary

```text
Application Layer
├── Contains use cases and application operations
├── Orchestrates Domain logic
├── Defines abstractions (interfaces) for Infrastructure
├── Uses DTOs for data transfer
├── Validates input (application-level validation)
├── Depends only on Domain Layer
├── Does NOT implement business rules
├── Does NOT depend on Infrastructure or Presentation
├── Can use patterns like CQRS and MediatR
└── Is testable with mocked abstractions
```

> **The Application Layer tells the Domain what to do — it does not decide the business rules.**

---
