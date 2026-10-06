# Microservices Architecture

Microservices Architecture is a software architecture style where an application is divided into **small, independently deployable services**.

Each service is responsible for a specific **business capability** and can be developed, deployed, scaled, and operated independently.

For example, a food platform might contain:

```text id="m9u8xj"
Restaurant Service
Menu Service
Order Service
Review Service
Payment Service
Identity Service
```

Each service is a separate application.

The main idea is:

```text id="v2q3kl"
Multiple Services
        ↓
Independent Deployment
        ↓
Business Capabilities
```

---

## 01 — Overview

A monolithic application might look like:

```text id="r7s0c1"
                 Monolithic Application
                        │
        ┌───────────────┼───────────────┐
        │               │               │
   Restaurants         Menus          Orders
        │               │               │
        └───────────────┼───────────────┘
                        │
                     Database
```

A microservices system divides those capabilities into separate applications:

```text id="3ld4fz"
            ┌─────────────────────┐
            │   Restaurant Service │
            └──────────┬──────────┘
                       │
                    Database

            ┌─────────────────────┐
            │     Menu Service    │
            └──────────┬──────────┘
                       │
                    Database

            ┌─────────────────────┐
            │    Order Service    │
            └──────────┬──────────┘
                       │
                    Database

            ┌─────────────────────┐
            │   Review Service    │
            └──────────┬──────────┘
                       │
                    Database
```

The services communicate over the network or through messaging.

---

## 02 — Core Idea

Each microservice owns a specific business capability.

For example:

```text id="i2qmfk"
Restaurant Service
→ Restaurant registration
→ Restaurant approval
→ Restaurant management
```

```text id="vx7f5n"
Menu Service
→ Menu management
→ Menu items
→ Availability
```

```text id="h3v8iw"
Order Service
→ Place order
→ Cancel order
→ Accept order
→ Complete order
```

The services are:

```text id="p9q6uh"
Independently Deployable
Independently Scalable
Independently Testable
Independently Operated
```

The goal is not simply to create many small applications.

The goal is to create **meaningful business boundaries**.

---

## 03 — What Is a Microservice?

A microservice is an independently deployable application responsible for a bounded business capability.

For example:

```text id="y1q5zv"
Orders Service
```

may own:

```text id="p7ld1e"
Order
OrderItem
Order Status
Order Pricing
Order Workflow
```

Another service should not directly modify those concepts.

Instead, it communicates with the Orders Service through its API or messaging contracts.

```text id="6d70l0"
Customer
   ↓
Order Service API
   ↓
Order Service
```

The internal implementation remains inside the service.

---

## 04 — Service Boundaries

A good microservices design starts with **business boundaries**.

For example:

```text id="y8gcn8"
Menuhat
│
├── Identity Service
├── Restaurant Service
├── Menu Service
├── Order Service
├── Review Service
└── Payment Service
```

Each service should have a clear responsibility.

Avoid splitting based only on technical concerns:

```text id="6j52dm"
Validation Service
Repository Service
Controller Service
Database Service
```

These usually do not represent meaningful business capabilities.

A better boundary is:

```text id="7qmcwa"
Orders
Payments
Restaurants
Menus
```

---

## 05 — Independent Deployment

One of the defining characteristics of microservices is **independent deployment**.

For example:

```text id="g41wq0"
Restaurant Service
       ↓
   Deploy v2.4

Menu Service
       ↓
   Still v2.1

Order Service
       ↓
   Deploy v5.0
```

Changing the Menu Service does not require deploying the Restaurant Service.

This is one of the biggest differences from a traditional monolith.

---

## 06 — Independent Scaling

Different services may have different workloads.

For example:

```text id="p4ymlq"
Order Service
    ↓
100 instances
```

while:

```text id="j5sqz8"
Review Service
    ↓
2 instances
```

The services can scale independently.

For example:

```text id="ae2u7a"
                    Traffic
                       │
             ┌─────────┼─────────┐
             │         │         │
           Orders     Menus    Reviews
             │         │         │
          Scale ↑    Scale →   Scale ↓
```

This can be useful when one business capability receives significantly more traffic than another.

---

## 07 — Service Ownership

A microservice should own its business logic and data.

For example:

```text id="ktw4f9"
Restaurant Service
       │
   Restaurant DB
```

```text id="n0tq7z"
Order Service
       │
    Order DB
```

```text id="58w5o7"
Review Service
       │
   Review DB
```

A strong principle is:

> **A service owns its data.**

The Order Service should not directly connect to the Restaurant Service's database.

Bad:

```text id="ipwb9b"
Order Service
      ↓
Restaurant Database
```

Better:

```text id="w7m38s"
Order Service
      ↓
Restaurant Service API
      ↓
Restaurant Database
```

---

## 08 — Database per Service

Microservices commonly use the **Database per Service** pattern.

For example:

```text id="g1c5o7"
Restaurant Service
       ↓
RestaurantDb

Menu Service
       ↓
MenuDb

Order Service
       ↓
OrderDb

Review Service
       ↓
ReviewDb
```

The databases could even use different technologies:

```text id="4sv69d"
RestaurantDb → PostgreSQL
MenuDb       → SQL Server
OrderDb      → PostgreSQL
ReviewDb     → MongoDB
```

The service owns its persistence technology.

This provides strong data ownership, but it also introduces distributed-data challenges.

---

## 09 — Communication Between Services

Microservices communicate through remote communication mechanisms.

Common choices include:

```text id="tl9d7j"
HTTP / REST
gRPC
Message Queues
Event Streaming
```

For example:

```text id="9vy36h"
Order Service
      ↓
HTTP
      ↓
Restaurant Service
```

Or asynchronous communication:

```text id="wm4i58"
Order Service
      ↓
OrderPlaced Event
      ↓
Message Broker
      ↓
Notification Service
```

The communication method should match the business requirement.

---

## 10 — Synchronous Communication

With synchronous communication, one service directly waits for another service.

For example:

```text id="cx8r7e"
Order Service
      ↓
GET Restaurant
      ↓
Restaurant Service
      ↓
Response
      ↓
Order Service
```

Example using `HttpClient`:

```csharp
using System.Net.Http.Json;

namespace Menuhat.OrderService.Clients;

public sealed class RestaurantClient
{
    private readonly HttpClient _httpClient;

    public RestaurantClient(HttpClient httpClient)
    {
        _httpClient = httpClient;
    }

    public async Task<bool> IsActiveAsync(
        int restaurantId,
        CancellationToken cancellationToken)
    {
        var result = await _httpClient.GetFromJsonAsync<RestaurantStatusResponse>(
            $"/api/restaurants/{restaurantId}/status",
            cancellationToken);

        return result?.IsActive ?? false;
    }
}

public sealed record RestaurantStatusResponse(
    bool IsActive);
```

The Order Service does not access the Restaurant database directly.

---

## 11 — Asynchronous Communication

With asynchronous communication, a service publishes an event and does not need to wait for another service to finish processing it.

For example:

```text id="j8f2tq"
Order Service
      ↓
OrderPlaced
      ↓
Message Broker
      ↓
Notification Service
```

An event could look like:

```csharp
namespace Menuhat.Contracts.Orders;

public sealed record OrderPlacedEvent(
    Guid OrderId,
    int CustomerId,
    DateTime OccurredAt);
```

The event describes something that **already happened**.

For example:

```text id="13xtby"
OrderPlaced
```

rather than:

```text id="0vx8qv"
CreateOrder
```

Commands ask for an action.

Events communicate that an action happened.

---

## 12 — Example

Suppose a customer places an order.

The system might require:

```text id="g86b2q"
1. Verify restaurant is active
2. Verify menu items are available
3. Create order
4. Request payment
5. Send confirmation
```

A possible microservice architecture is:

```text id="0apbt3"
Customer
   ↓
Order Service
   │
   ├────→ Restaurant Service
   │
   ├────→ Menu Service
   │
   └────→ Payment Service
```

After the order is created:

```text id="g5kmr5"
Order Service
      ↓
OrderPlaced Event
      ↓
Message Broker
      ↓
Notification Service
```

The system is now distributed across several services.

---

## 13 — Example Service Structure

A .NET microservice can internally use any suitable architecture.

For example:

```text id="1g1jn2"
Menuhat.OrderService/
│
├── Api/
│   ├── Endpoints/
│   └── Middleware/
│
├── Application/
│   ├── PlaceOrder/
│   └── CancelOrder/
│
├── Domain/
│   ├── Orders/
│   └── OrderItems/
│
├── Infrastructure/
│   ├── Persistence/
│   └── Clients/
│
└── Program.cs
```

The microservice itself can therefore use:

```text id="6or7r5"
Onion Architecture
Hexagonal Architecture
Vertical Slice Architecture
Clean Architecture
```

Microservices and these architectures are not mutually exclusive.

---

## 14 — Example: Order Service

Suppose the Order Service exposes:

```text id="ihy1k4"
POST /api/orders
```

Request:

```csharp
namespace Menuhat.OrderService.Contracts;

public sealed record PlaceOrderRequest(
    int RestaurantId,
    IReadOnlyList<PlaceOrderItemRequest> Items);

public sealed record PlaceOrderItemRequest(
    int MenuItemId,
    int Quantity);
```

The Order Service receives the request:

```text id="72h2kk"
HTTP Request
      ↓
Order API
      ↓
Place Order Handler
```

The handler may communicate with other services:

```csharp
namespace Menuhat.OrderService.Application.PlaceOrder;

public sealed class PlaceOrderHandler
{
    private readonly IRestaurantClient _restaurants;
    private readonly IMenuClient _menus;
    private readonly IOrderRepository _orders;

    public PlaceOrderHandler(
        IRestaurantClient restaurants,
        IMenuClient menus,
        IOrderRepository orders)
    {
        _restaurants = restaurants;
        _menus = menus;
        _orders = orders;
    }

    public async Task<Guid> Handle(
        PlaceOrderCommand command,
        CancellationToken cancellationToken)
    {
        var restaurantIsActive =
            await _restaurants.IsActiveAsync(
                command.RestaurantId,
                cancellationToken);

        if (!restaurantIsActive)
            throw new InvalidOperationException(
                "Restaurant is not active.");

        var items = await _menus.GetAvailableItemsAsync(
            command.Items,
            cancellationToken);

        var order = Order.Create(
            command.RestaurantId,
            items);

        await _orders.AddAsync(
            order,
            cancellationToken);

        return order.Id;
    }
}
```

The service owns the order creation workflow.

---

## 15 — Service-to-Service Dependencies

Services can depend on other services.

For example:

```text id="z7h1y4"
Order Service
      ↓
Restaurant Service
```

However, excessive synchronous dependencies can create a chain:

```text id="0axv1s"
Order
 ↓
Restaurant
 ↓
Menu
 ↓
Payment
 ↓
Customer
```

Now one request depends on many services being available.

This increases system complexity.

A good microservices architecture tries to minimize unnecessary runtime dependencies.

---

## 16 — Distributed Data

One of the biggest differences from a monolith is that transactions may cross service boundaries.

In a monolith:

```text id="m1un1d"
Order
  +
Payment
  +
Inventory
       ↓
One Database Transaction
```

In microservices:

```text id="a0v3mt"
Order Service
      ↓
Order DB

Payment Service
      ↓
Payment DB

Inventory Service
      ↓
Inventory DB
```

There may be no single database transaction covering all operations.

This introduces problems such as:

```text id="f7s0yk"
Partial Failure
Eventual Consistency
Retry
Duplicate Messages
Compensation
```

Distributed data is one of the major complexities of microservices.

---

## 17 — Eventual Consistency

Suppose an order is created:

```text id="f4c5ri"
Order Created
      ↓
Payment Requested
      ↓
Payment Processing
```

The payment result may not be immediately available.

The system may eventually become:

```text id="d0nqik"
Order
   ↓
Pending Payment
   ↓
Payment Completed
   ↓
Order Confirmed
```

Different services may temporarily have different views of the system.

This is called **eventual consistency**.

A microservices system needs to explicitly handle these states.

---

## 18 — Resilience

Network communication can fail.

For example:

```text id="1yd1ul"
Order Service
      ↓
Restaurant Service
      X
   Timeout
```

Unlike a method call inside a monolith, a network call can fail because of:

```text id="8r5y4t"
Timeout
Network Failure
Service Unavailable
DNS Failure
Overload
Connection Failure
```

Microservices therefore often need resilience techniques such as:

```text id="x9q3bz"
Timeouts
Retries
Circuit Breakers
Rate Limiting
Fallbacks
```

These must be used carefully.

A retry policy that retries everything indefinitely can make an outage worse.

---

## 19 — Observability

A request can cross multiple services:

```text id="g0e0f1"
Client
  ↓
API Gateway
  ↓
Order Service
  ↓
Restaurant Service
  ↓
Menu Service
  ↓
Payment Service
```

When something fails, logs from one service may not be enough.

Microservices therefore typically require strong observability:

```text id="m74x8t"
Logging
Metrics
Distributed Tracing
Health Checks
Correlation IDs
```

A trace can show:

```text id="9juyoe"
Request
  │
  ├── Order Service     120ms
  ├── Restaurant       40ms
  ├── Menu             70ms
  └── Payment          210ms
```

This helps identify where the problem occurred.

---

## 20 — API Gateway

A system with many services may place an API Gateway in front of them.

```text id="7t4bmy"
                   Client
                      ↓
                API Gateway
                      │
          ┌───────────┼───────────┐
          ↓           ↓           ↓
      Restaurant     Menu        Order
       Service      Service     Service
```

The gateway can handle concerns such as:

```text id="v9q2t4"
Routing
Authentication
Rate Limiting
Request Aggregation
TLS Termination
```

An API Gateway is common, but it is not a mandatory requirement for every microservices system.

---

## 21 — Testing

Microservices require multiple levels of testing.

### Unit Tests

Test business logic inside one service.

```text id="q8n0w9"
Order Service
     ↓
Unit Tests
```

---

### Integration Tests

Test a service with its infrastructure.

```text id="g6l0qt"
Order Service
     ↓
Order Database
```

---

### Contract Tests

Verify that services agree on their communication contracts.

```text id="w1x0q3"
Order Service
      ↕
API / Event Contract
      ↕
Restaurant Service
```

---

### End-to-End Tests

Test the entire distributed workflow:

```text id="a3q9s0"
Client
 ↓
Gateway
 ↓
Order
 ↓
Restaurant
 ↓
Menu
 ↓
Payment
```

Because distributed systems contain more moving parts, testing strategy becomes more important.

---

## 22 — Advantages

### Independent Deployment

Each service can be deployed independently.

```text id="4nnwn5"
Order Service → Deploy
Menu Service  → No Change
Review Service → No Change
```

---

### Independent Scaling

A service can scale according to its own workload.

```text id="f3r9yb"
Orders → 20 instances
Menus  → 5 instances
Reviews → 2 instances
```

---

### Business Isolation

A service owns a specific business capability.

```text id="ex6rpn"
Orders
Menus
Restaurants
Payments
```

This can make large systems easier to divide among teams.

---

### Technology Flexibility

Different services can potentially use different technologies.

For example:

```text id="uc1rhz"
Order Service
→ ASP.NET Core

Analytics Service
→ Python

Notification Service
→ Node.js
```

This flexibility can be useful when there is a strong reason to use different technologies.

---

### Fault Isolation

A failure in one service does not necessarily mean the entire system must stop.

For example:

```text id="mef1hi"
Review Service
      X
   Failure
```

The Order Service may continue operating.

However, the degree of isolation depends on how services are coupled.

---

## 23 — Common Problems

### Distributed Monolith

A system can technically use microservices while behaving like one large application.

For example:

```text id="vbl0wh"
Order Service
   ↓
Restaurant Service
   ↓
Menu Service
   ↓
Payment Service
   ↓
Customer Service
```

If every request requires every service, the system becomes highly coupled.

This is sometimes called a **distributed monolith**.

The system gets the complexity of microservices without getting their intended benefits.

---

### Too Many Services

Splitting everything into tiny services creates operational overhead.

For example:

```text id="c0ivti"
Product Name Service
Price Service
Image Service
Category Service
```

This is usually excessive.

Service boundaries should represent meaningful business capabilities.

---

### Shared Database

If every service accesses the same tables:

```text id="vwl6n1"
Order Service
      ↓
Shared Database
      ↑
Menu Service
      ↑
Restaurant Service
```

the services are not truly independent.

They become coupled through the database.

---

### Distributed Transactions

Operations crossing multiple services can be difficult to make atomic.

For example:

```text id="u94xqe"
Create Order
     ↓
Payment
     ↓
Inventory
```

If Payment succeeds but Inventory fails, the system needs a strategy for handling the inconsistent state.

---

### Network Complexity

A method call:

```csharp
_orderService.PlaceOrder();
```

is fundamentally different from:

```text id="j0v9gt"
HTTP
 ↓
Order Service
```

The second introduces latency, failures, serialization, authentication, and network concerns.

---

### Operational Complexity

Microservices require additional infrastructure and operational practices:

```text id="5qcl6e"
Containers
Orchestration
Logging
Tracing
Metrics
Service Discovery
Secrets
Configuration
Message Brokers
CI/CD
```

This can become expensive in engineering effort.

---

## 24 — When to Use

Microservices can be a good fit when:

* The system has clear business boundaries.
* Different parts need independent deployment.
* Different parts have significantly different scaling requirements.
* Multiple teams need strong ownership boundaries.
* Independent technology choices provide real value.
* The organization can support distributed-system operations.
* The business complexity justifies the additional infrastructure.

Microservices are usually most useful when there is a **real need for independent services**.

---

## 25 — When Not to Use

Microservices may not be a good fit when:

* The application is small.
* The team is small and inexperienced with distributed systems.
* Business boundaries are still unclear.
* Independent deployment is not needed.
* The system has mostly simple CRUD operations.
* Operational simplicity is more important than independent scaling.
* A Modular Monolith would solve the problem.

For example:

```text id="p8w9f4"
Small Startup Application
```

does not automatically need:

```text id="g6u2xj"
10 Microservices
```

Starting with a well-designed monolith or modular monolith can be much more practical.

---

## 26 — Microservices vs Monolith

### Monolith

```text id="b8v7k2"
                 One Application
                       │
       ┌───────────────┼───────────────┐
       │               │               │
   Restaurants        Menus          Orders
       │               │               │
       └───────────────┼───────────────┘
                       │
                    Database
```

### Microservices

```text id="g8y3c1"
Restaurant Service
       │
Restaurant DB

Menu Service
       │
Menu DB

Order Service
       │
Order DB
```

Main difference:

```text id="qj5w1v"
Monolith
→ One Deployable Application

Microservices
→ Multiple Independently Deployable Applications
```

---

## 27 — Microservices vs Modular Monolith

These are closely related concepts.

### Modular Monolith

```text id="n3w8tb"
                 One Application
                       │
       ┌───────────────┼───────────────┐
       │               │               │
   Restaurants        Menus          Orders
       │               │               │
       └───────────────┼───────────────┘
                       │
                    Database
```

### Microservices

```text id="x4q7rs"
Restaurant Service
       │
   Database

Menu Service
       │
   Database

Order Service
       │
   Database
```

Modular Monolith:

```text id="h9o0qv"
One Deployment
```

Microservices:

```text id="a8w4sm"
Independent Deployments
```

A Modular Monolith provides strong module boundaries without requiring network communication between internal modules.

---

## 28 — Practical Request Flow

Suppose a customer places an order.

```text id="e2yq5x"
Client
  ↓
API Gateway
  ↓
Order Service
  │
  ├──→ Restaurant Service
  │
  ├──→ Menu Service
  │
  └──→ Payment Service
           │
           ▼
       Payment DB
```

Then:

```text id="5pk8u4"
Order Service
      ↓
OrderPlaced Event
      ↓
Message Broker
      ↓
Notification Service
```

This workflow is more powerful than a simple monolith, but also much more complex.

---

## 29 — Evolution Toward Microservices

A common evolution can be:

```text id="b4c9z1"
Monolith
    ↓
Structured Monolith
    ↓
Modular Monolith
    ↓
Identify Real Boundaries
    ↓
Extract Selected Module
    ↓
Microservice
```

For example:

```text id="s9m1ws"
Menuhat Modular Monolith

Restaurants
Menus
Orders
Reviews
```

Later, if Orders genuinely needs independent scaling and deployment:

```text id="5r1mmy"
Menuhat
   │
   ├── Restaurants Module
   ├── Menus Module
   └── Reviews Module
             │
             └── Order Service
```

This is often safer than starting with many services before the boundaries are understood.

---

## 30 — Summary

Microservices Architecture divides an application into **independently deployable services**, where each service owns a meaningful business capability.

The basic model is:

```text id="5pmvtb"
             Microservices
                  │
      ┌───────────┼───────────┐
      │           │           │
 Restaurants    Menus       Orders
      │           │           │
    DB            DB          DB
```

The main principles are:

```text id="m2jn3q"
Business Boundary    → Each service owns a capability
Independent Deploy   → Services can deploy separately
Data Ownership       → Each service owns its data
Communication        → APIs or Messages
Independent Scaling  → Scale services separately
Fault Isolation      → Failures can be isolated
```

The main challenges are:

```text id="x3h7vj"
Distributed Data
Network Failures
Eventual Consistency
Observability
Deployment Complexity
Testing Complexity
Operational Overhead
```

The most important distinction is:

```text id="9xdj8b"
Monolith
→ One Deployable Application

Modular Monolith
→ One Application + Strong Business Modules

Microservices
→ Multiple Independently Deployable Services
```

Microservices should therefore not be introduced simply because they are considered "more professional."

The real question is:

```text id="s0qj7r"
Do we have a real business,
scaling,
deployment,
team,
or organizational reason
to separate this capability?
```

When the answer is yes, microservices can be a powerful architecture.

When the answer is no, a well-designed **Modular Monolith** can often provide most of the architectural benefits with significantly less complexity.
