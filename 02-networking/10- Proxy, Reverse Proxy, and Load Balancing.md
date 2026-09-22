# Proxy, Reverse Proxy, and Load Balancing

## Introduction

When building a simple web application, the architecture might look like this:

```text
Client
   │
   ▼
ASP.NET Application
```

The client sends an HTTP request directly to the application.

For example:

```text
Browser
   │
   │ GET /api/users
   ▼
ASP.NET API
```

This architecture can work for a small application.

However, as the application grows, we may need additional features such as:

- HTTPS and TLS certificates
    
- Security
    
- Request filtering
    
- Rate limiting
    
- Caching
    
- Multiple application servers
    
- High availability
    
- Traffic distribution
    
- Health checks
    

This is where **proxies**, **reverse proxies**, and **load balancers** become important.

A more realistic architecture might look like this:

```text
                Internet
                    │
                    ▼
              Reverse Proxy
                    │
                    ▼
               Load Balancer
                    │
          ┌─────────┼─────────┐
          ▼         ▼         ▼
        API 1     API 2     API 3
```

In many systems, the reverse proxy and load balancer can even be the same component.

---

# 1. What Is a Proxy?

A **proxy** is an intermediary that sits between two parties and forwards communication between them.

Instead of direct communication:

```text
Client ─────────────────► Server
```

The communication goes through another component:

```text
Client ───► Proxy ───► Server
```

The proxy receives the request first.

Then it can:

1. Inspect the request.
    
2. Apply rules.
    
3. Modify the request.
    
4. Forward the request.
    
5. Receive the response.
    
6. Return the response to the original client.
    

```text
Client
   │
   │ Request
   ▼
Proxy
   │
   │ Forward Request
   ▼
Server
   │
   │ Response
   ▼
Proxy
   │
   │ Forward Response
   ▼
Client
```

The important idea is:

> A proxy is a middleman in network communication.

There are two important types:

```text
Proxy
├── Forward Proxy
└── Reverse Proxy
```

---

# 2. Forward Proxy

A **forward proxy** sits in front of the client.

The client sends requests to the proxy, and the proxy communicates with external servers.

```text
Client
   │
   ▼
Forward Proxy
   │
   ▼
Internet
   │
   ▼
Server
```

For example, imagine a company with many employees.

Without a proxy:

```text
Employee 1 ───► Internet
Employee 2 ───► Internet
Employee 3 ───► Internet
```

With a forward proxy:

```text
Employee 1 ─┐
Employee 2 ─┼──► Company Proxy ───► Internet
Employee 3 ─┘
```

The company can control the employees' internet access.

For example:

```text
Client Request
      │
      ▼
Forward Proxy
      │
      ├── Is this website allowed?
      │
      ├── Should this request be blocked?
      │
      ├── Should the response be cached?
      │
      └── Should this activity be logged?
```

## Common Uses of Forward Proxies

### Access Control

A company can block specific websites.

```text
Employee
   │
   ▼
Proxy
   │
   ├── example.com      ✓ Allowed
   │
   └── blocked-site.com ✗ Blocked
```

---

### Hiding the Client

The external server may communicate with the proxy instead of directly with the original client.

```text
Client IP: 192.168.1.10

Client
   │
   ▼
Forward Proxy
Public IP: 50.10.20.30
   │
   ▼
Website
```

The website may see the proxy's public IP address.

---

### Caching

The proxy can store frequently requested resources.

```text
Client
   │
   ▼
Proxy
   │
   ├── Cached Response? ── Yes ──► Return Cached Data
   │
   └── No
        │
        ▼
      Server
```

This can reduce network traffic and improve response time.

---

# 3. What Is a Reverse Proxy?

A **reverse proxy** sits in front of one or more servers.

The client sends a request to the reverse proxy instead of directly connecting to the backend application.

```text
Client
   │
   ▼
Reverse Proxy
   │
   ▼
Backend Server
```

For example:

```text
Internet
    │
    ▼
api.example.com
    │
    ▼
Reverse Proxy
    │
    ▼
ASP.NET API
```

The client usually does not know:

- The internal server IP address.
    
- The internal port.
    
- How many backend servers exist.
    
- Which server processes the request.
    

From the client's perspective:

```text
Client
   │
   ▼
https://api.example.com
```

But internally:

```text
                    ┌──► API Server 1
Client ──► Proxy ───┤
                    └──► API Server 2
```

The reverse proxy hides the backend infrastructure from the client.

---

# 4. Reverse Proxy and ASP.NET Core

Suppose your ASP.NET application is running with Kestrel.

```text
ASP.NET Core
     │
     ▼
Kestrel
     │
     ▼
Port 5000
```

The application may be available internally at:

```text
http://localhost:5000
```

However, exposing the application directly to the internet is often not the preferred architecture.

Instead:

```text
Internet
    │
    ▼
Reverse Proxy
    │
    ▼
Kestrel
    │
    ▼
ASP.NET Core Application
```

For example:

```text
                    Internal Network

Internet
   │
   ▼
Nginx
   │
   │ http://localhost:5000
   ▼
Kestrel
   │
   ▼
ASP.NET Core API
```

The reverse proxy receives:

```text
https://api.example.com/api/users
```

Then forwards it internally:

```text
http://localhost:5000/api/users
```

The client does not need to know that the actual application is running on port `5000`.

---

# 5. Request Flow Through a Reverse Proxy

Let's follow a request step by step.

The client sends:

```http
GET /api/users HTTP/1.1
Host: api.example.com
```

The request first reaches the reverse proxy.

```text
Client
   │
   │ GET /api/users
   ▼
Reverse Proxy
```

The reverse proxy checks its configuration.

For example:

```text
/api/*  ───► ASP.NET API
/images/* ─► Image Server
/auth/* ───► Authentication Service
```

Then it forwards the request to the correct backend.

```text
Client
   │
   ▼
Reverse Proxy
   │
   │ GET /api/users
   ▼
ASP.NET API
```

The API processes the request:

```text
ASP.NET API
    │
    ├── Routing
    ├── Authentication
    ├── Authorization
    ├── Business Logic
    └── Database
```

Then the response travels back:

```text
Database
   │
   ▼
ASP.NET API
   │
   ▼
Reverse Proxy
   │
   ▼
Client
```

---

# 6. TLS / HTTPS Termination

One of the common responsibilities of a reverse proxy is handling HTTPS.

Without a reverse proxy:

```text
Client
   │ HTTPS
   ▼
ASP.NET Application
```

The application itself handles the TLS certificate and encryption.

With a reverse proxy:

```text
Client
   │
   │ HTTPS
   ▼
Reverse Proxy
   │
   │ HTTP
   ▼
ASP.NET Application
```

The reverse proxy:

1. Receives the encrypted HTTPS request.
    
2. Uses the TLS certificate.
    
3. Decrypts the traffic.
    
4. Sends the request to the backend.
    

This is called:

```text
TLS Termination
```

Example:

```text
Client
   │
   │ HTTPS (Encrypted)
   ▼
Reverse Proxy
   │
   │ TLS Termination
   ▼
HTTP (Internal Network)
   │
   ▼
ASP.NET API
```

The connection between the proxy and backend may also use HTTPS depending on the security requirements.

---

# 7. Request Routing

A reverse proxy can route requests to different services.

Imagine a system with multiple applications.

```text
                       ┌──► User Service
                       │
Client ──► Proxy ──────┼──► Product Service
                       │
                       └──► Order Service
```

The proxy can route based on the URL.

```text
/api/users/*

        ▼

User Service
```

```text
/api/products/*

        ▼

Product Service
```

```text
/api/orders/*

        ▼

Order Service
```

Example:

```text
Client Request

/api/users
      │
      ▼
Reverse Proxy
      │
      ▼
User Service
```

Another request:

```text
/api/orders
      │
      ▼
Reverse Proxy
      │
      ▼
Order Service
```

This allows multiple backend services to be exposed through one public domain.

```text
https://api.example.com/users
https://api.example.com/products
https://api.example.com/orders
```

---

# 8. What Is Load Balancing?

A **load balancer** distributes incoming traffic across multiple servers.

Without load balancing:

```text
              ┌───────────┐
Clients ─────►│ Server 1  │
              └───────────┘
```

All traffic goes to one server.

If the server receives too many requests:

```text
Too Many Requests
        │
        ▼
Server Overloaded
        │
        ▼
Slow Responses
        │
        ▼
Possible Failure
```

Instead, we can run multiple instances of the application.

```text
                 ┌──► Server 1
                 │
Clients ──► LB ──┼──► Server 2
                 │
                 └──► Server 3
```

The load balancer distributes requests.

---

# 9. Why Do We Need Multiple Servers?

Imagine your API receives:

```text
10,000 Requests / Second
```

One server may not be enough.

Instead:

```text
                    ┌──► API 1
                    │
10,000 Requests ────┼──► API 2
                    │
                    ├──► API 3
                    │
                    └──► API 4
```

The traffic can be distributed.

For example:

```text
API 1 → 2,500 Requests
API 2 → 2,500 Requests
API 3 → 2,500 Requests
API 4 → 2,500 Requests
```

The actual distribution may not be perfectly equal because it depends on the load-balancing algorithm and application behavior.

---

# 10. Load Balancing Algorithms

A load balancer needs a rule to decide where the next request should go.

---

## Round Robin

Requests are distributed sequentially.

```text
Request 1 ──► Server 1

Request 2 ──► Server 2

Request 3 ──► Server 3

Request 4 ──► Server 1

Request 5 ──► Server 2
```

Diagram:

```text
          Request 1
              │
              ▼
           Server 1

          Request 2
              │
              ▼
           Server 2

          Request 3
              │
              ▼
           Server 3
```

Round Robin is simple and works well when servers have similar capacity.

---

## Least Connections

The load balancer checks how many active connections each server currently has.

```text
Server 1 → 100 Connections
Server 2 → 20 Connections
Server 3 → 50 Connections
```

A new request may go to:

```text
Server 2
```

Because it currently has the fewest active connections.

---

## Weighted Load Balancing

Servers may have different capacities.

For example:

```text
Server 1 → Powerful → Weight 3
Server 2 → Medium   → Weight 2
Server 3 → Small    → Weight 1
```

The more powerful server receives more traffic.

```text
            ┌──► Server 1 █████████
            │
Requests ───┼──► Server 2 ██████
            │
            └──► Server 3 ███
```

---

## IP Hash

The load balancer uses information such as the client's IP address to consistently route a client to the same backend.

```text
Client A ──► Server 1
Client B ──► Server 2
Client C ──► Server 3

Client A ──► Server 1
```

This can be useful for some session-based applications.

---

# 11. Health Checks

A load balancer should not send traffic to a server that is unhealthy.

For example:

```text
Load Balancer

Server 1 → ✓ Healthy
Server 2 → ✗ Unhealthy
Server 3 → ✓ Healthy
```

Traffic should only go to healthy servers.

```text
                 ┌──► Server 1 ✓
Client ──► LB ───┤
                 ├──X Server 2 ✗
                 │
                 └──► Server 3 ✓
```

The load balancer periodically checks the servers.

For example:

```http
GET /health
```

The application responds:

```http
HTTP/1.1 200 OK
```

If the server stops responding correctly:

```text
Server
   │
   └── Health Check Failed
             │
             ▼
      Remove From Traffic
```

When the server becomes healthy again:

```text
Health Check Successful
          │
          ▼
Add Back to Traffic
```

---

# 12. High Availability

Multiple servers improve availability.

Without load balancing:

```text
Client
   │
   ▼
Server
   │
   X
   │
Application Down
```

If the only server fails, the application becomes unavailable.

With multiple servers:

```text
                   ┌──► Server 1 ✓
                   │
Client ──► LB ─────┼──► Server 2 ✗
                   │
                   └──► Server 3 ✓
```

The failed server can be removed from traffic.

Users can continue using the application through the remaining servers.

This is one of the main reasons for using load balancing.

---

# 13. Scaling

Load balancing makes **horizontal scaling** possible.

## Vertical Scaling

Increase the power of one server.

```text
Before:

Server
CPU: 4 Cores
RAM: 8 GB
```

After:

```text
Server
CPU: 16 Cores
RAM: 64 GB
```

This is called:

```text
Scale Up
```

---

## Horizontal Scaling

Add more application instances.

```text
Before:

Load Balancer
      │
      ▼
    API 1
```

After:

```text
                ┌──► API 1
                ├──► API 2
Load Balancer ──┤
                ├──► API 3
                └──► API 4
```

This is called:

```text
Scale Out
```

Modern cloud applications often use horizontal scaling.

---

# 14. Reverse Proxy vs Load Balancer

These concepts are related but have different primary responsibilities.

|Reverse Proxy|Load Balancer|
|---|---|
|Sits in front of backend servers|Distributes traffic|
|Routes requests|Selects a backend server|
|Can handle TLS/HTTPS|Can perform health checks|
|Can cache responses|Supports high availability|
|Can apply security rules|Supports horizontal scaling|

However, a single component can perform both roles.

```text
                     ┌──► API 1
                     │
Client ──► Nginx ────┼──► API 2
                     │
                     └──► API 3
```

Here, Nginx can act as:

```text
Nginx
├── Reverse Proxy
├── Request Router
├── TLS Terminator
└── Load Balancer
```

So the difference is about **responsibility**, not necessarily about using different software.

---

# 15. A Complete Real-World Request Flow

Let's imagine a user opens:

```text
https://api.example.com/api/products
```

The request may travel like this:

```text
1. Client
       │
       ▼

2. DNS
       │
       │ Finds IP address for api.example.com
       ▼

3. Public IP Address
       │
       ▼

4. Reverse Proxy / Load Balancer
       │
       │ TLS Termination
       │ Security Checks
       │ Rate Limiting
       │ Select Backend
       ▼

5. ASP.NET Core / Kestrel
       │
       │ Routing
       │ Authentication
       │ Authorization
       │ Business Logic
       ▼

6. Database
```

The response travels back:

```text
Database
   │
   ▼
ASP.NET Application
   │
   ▼
Kestrel
   │
   ▼
Reverse Proxy / Load Balancer
   │
   ▼
Client
```

---

# 16. Where Does Kestrel Fit?

Kestrel is the web server used by ASP.NET Core applications.

A simplified architecture:

```text
Internet
    │
    ▼
Reverse Proxy
    │
    ▼
Kestrel
    │
    ▼
ASP.NET Core Middleware Pipeline
    │
    ▼
Endpoint
```

For example:

```text
Client
   │
   │ HTTPS Request
   ▼
Nginx
   │
   │ HTTP Request
   ▼
Kestrel
   │
   ▼
ASP.NET Core
   │
   ▼
GET /api/users
```

Kestrel receives the HTTP request and passes it into the ASP.NET Core request pipeline.

---

# 17. One Domain, Multiple Applications

A reverse proxy can expose multiple applications using a single public domain.

```text
https://example.com
```

Internally:

```text
                         ┌──► Frontend
                         │
Client ──► Reverse Proxy ├──► API
                         │
                         ├──► Authentication Service
                         │
                         └──► Admin Panel
```

For example:

```text
example.com/           → Frontend

example.com/api        → ASP.NET API

example.com/auth       → Authentication Service

example.com/admin      → Admin Application
```

The client only needs to know the public URLs.

The internal infrastructure remains hidden.

---

# 18. Important Problem: Client IP Address

When a reverse proxy sits between the client and the backend:

```text
Real Client
IP: 100.20.30.40
      │
      ▼
Reverse Proxy
IP: 10.0.0.10
      │
      ▼
ASP.NET Application
```

The backend may see the reverse proxy as the direct connection.

Therefore, proxies often forward information about the original client.

Conceptually:

```text
X-Forwarded-For: 100.20.30.40
```

The application can use forwarded headers to understand information about the original request.

For example:

```text
Original Client
      │
      ▼
Reverse Proxy
      │
      ├── X-Forwarded-For
      ├── X-Forwarded-Proto
      └── Other Forwarded Information
      │
      ▼
ASP.NET Application
```

This is important for:

- Logging
    
- Security
    
- Rate limiting
    
- Generating correct URLs
    
- Understanding whether the original request used HTTP or HTTPS
    

---

# 19. Example Architecture for a .NET Application

A production architecture might look like this:

```text
                         Internet
                            │
                            ▼
                      DNS Resolution
                            │
                            ▼
                     Public IP Address
                            │
                            ▼
                  Reverse Proxy / Nginx
                            │
                ┌───────────┴───────────┐
                ▼                       ▼
            API Instance 1          API Instance 2
                │                       │
                └───────────┬───────────┘
                            ▼
                         Database
```

The flow is:

```text
Client
  ↓
DNS
  ↓
Reverse Proxy
  ↓
Load Balancing Decision
  ↓
Kestrel
  ↓
ASP.NET Core
  ↓
Endpoint
  ↓
Business Logic
  ↓
Database
```

---

# 20. Stateless Applications and Load Balancing

When running multiple application instances, the application should ideally be **stateless**.

Bad example:

```text
User Login
    │
    ▼
API 1
    │
    └── Stores Session Only in Memory
```

The next request might go to:

```text
Load Balancer
      │
      ▼
API 2
```

API 2 does not have the session stored in API 1's memory.

This creates a problem.

A better approach is to use shared storage.

```text
              ┌──► API 1 ──┐
Client ──► LB ┤             │
              ├──► API 2 ──┼──► Shared Cache / Database
              │             │
              └──► API 3 ──┘
```

For example:

```text
API 1
API 2
API 3
   │
   ▼
Redis / Distributed Cache
```

Now any application instance can process the request.

This is an important concept when building scalable applications.

---

# 21. Proxy vs Reverse Proxy vs Load Balancer

```text
Forward Proxy
```

Works mainly on behalf of the client.

```text
Client
   │
   ▼
Forward Proxy
   │
   ▼
Internet
```

---

```text
Reverse Proxy
```

Works in front of the servers.

```text
Client
   │
   ▼
Reverse Proxy
   │
   ▼
Server
```

---

```text
Load Balancer
```

Distributes traffic across multiple servers.

```text
                 ┌──► Server 1
Client ──► LB ───┼──► Server 2
                 └──► Server 3
```

---

# Summary

The complete relationship can be understood like this:

```text
Forward Proxy
    │
    └── Represents the Client

Reverse Proxy
    │
    └── Represents / Protects the Servers

Load Balancer
    │
    └── Distributes Traffic Between Servers
```

A modern backend architecture might look like:

```text
Client
   │
   ▼
DNS
   │
   ▼
Reverse Proxy
   │
   ├── TLS / HTTPS
   ├── Security
   ├── Rate Limiting
   ├── Request Routing
   └── Load Balancing
          │
          ▼
   ┌───────────────────┐
   │ Application Layer │
   │                   │
   │  ASP.NET Core     │
   │       +           │
   │     Kestrel       │
   └───────────────────┘
          │
          ▼
       Database
```

## Key Takeaways

```text
Proxy
= A middleman between two parties.

Forward Proxy
= Sits in front of clients and forwards their requests.

Reverse Proxy
= Sits in front of servers and receives client requests.

Load Balancer
= Distributes traffic between multiple backend servers.

Kestrel
= The web server that receives HTTP requests for an ASP.NET Core application.

TLS Termination
= The reverse proxy handles HTTPS encryption and certificates.

Health Check
= Checks whether a backend server is healthy before sending traffic to it.

Horizontal Scaling
= Adding more application instances.

High Availability
= Keeping the application available even if some servers fail.
```

## Complete Request Flow

```text
Client
   │
   ▼
DNS
   │
   ▼
Reverse Proxy / Load Balancer
   │
   ├── HTTPS / TLS
   ├── Security
   ├── Routing
   └── Select Healthy Server
             │
             ▼
          Kestrel
             │
             ▼
      ASP.NET Core Pipeline
             │
             ▼
          Endpoint
             │
             ▼
        Business Logic
             │
             ▼
          Database
```

The most important thing to understand is that **a reverse proxy and load balancer usually exist before your ASP.NET application**.

Your application does not necessarily communicate directly with the internet.

Instead, a request may pass through several layers before finally reaching:

```text
Kestrel → ASP.NET Core → Middleware → Endpoint
```