# Application Layer

> **Goal:** Understand how applications communicate over a network and learn the most important application-layer protocols for a Backend Engineer, especially DNS, HTTP, and HTTPS.

---

# 1. What Is the Application Layer?

The **Application Layer** is the highest layer in the TCP/IP model.

It provides network communication services directly to applications.

```text
┌─────────────────────────────┐
│ Application Layer           │
│                             │
│ HTTP   DNS   SMTP   FTP     │
├─────────────────────────────┤
│ Transport Layer             │
│ TCP / UDP                   │
├─────────────────────────────┤
│ Internet Layer              │
│ IP                          │
├─────────────────────────────┤
│ Link Layer                  │
│ Ethernet / Wi-Fi            │
└─────────────────────────────┘
```

Applications do not need to understand how IP routing, Ethernet frames, or physical signals work.

Instead, they use **application-layer protocols**.

For example:

```text
Browser
   │
   │ HTTP
   ▼
Web Server
```

Or:

```text
Application
   │
   │ DNS Query
   ▼
DNS Server
```

The Application Layer defines the rules for communication between applications.

---

# 2. What Is an Application Protocol?

An **application protocol** defines how applications communicate.

It specifies things such as:

- The format of messages.
    
- How requests are created.
    
- How responses are created.
    
- What operations are available.
    
- How errors are represented.
    
- How applications interpret received data.
    

For example, HTTP defines messages such as:

```http
GET /users HTTP/1.1
Host: example.com
```

And responses such as:

```http
HTTP/1.1 200 OK
Content-Type: application/json

[
  {
    "id": 1,
    "name": "Mohamed"
  }
]
```

The application protocol focuses on **what applications communicate**.

Lower layers handle **how the data is delivered**.

```text
Application
     │
     │ HTTP Message
     ▼
Transport
     │
     │ TCP Segment
     ▼
Internet
     │
     │ IP Packet
     ▼
Link
     │
     │ Frame
     ▼
Network
```

---

# 3. Application Layer and Backend Development

As a Backend Engineer, the Application Layer is one of the most important networking layers.

Your backend application communicates using protocols such as:

```text
Client
   │
   │ HTTPS
   ▼
ASP.NET Core API
   │
   ├── SQL Protocol ──────► Database
   │
   ├── Redis Protocol ────► Redis
   │
   └── AMQP ──────────────► RabbitMQ
```

A backend application may act as both:

- A **server**, receiving requests.
    
- A **client**, sending requests to other services.
    

For example:

```text
Browser
   │
   │ HTTPS Request
   ▼
Your API
   │
   │ HTTPS Request
   ▼
Payment Service
```

The same backend application can receive and send network requests.

---

# 4. DNS

## What Is DNS?

**DNS (Domain Name System)** translates domain names into IP addresses.

Humans prefer names:

```text
google.com
api.example.com
github.com
```

Networks use IP addresses:

```text
142.250.x.x
```

DNS connects these two worlds.

```text
Browser
   │
   │ "What is the IP address of example.com?"
   ▼
DNS
   │
   │ IP Address
   ▼
Browser
   │
   ▼
Connect to Server
```

---

## Why Do We Need DNS?

Imagine if every website required you to remember its IP address.

Instead of:

```text
https://example.com
```

You would need something like:

```text
https://93.184.216.34
```

DNS allows applications to use human-readable names.

```text
example.com
     │
     ▼
DNS Resolution
     │
     ▼
IP Address
     │
     ▼
Connect to Server
```

---

## How DNS Resolution Works

Suppose you enter:

```text
https://api.example.com/users
```

Before the browser can connect to the server, it needs an IP address.

The simplified process is:

```text
Browser
   │
   │ api.example.com?
   ▼
DNS Resolver
   │
   │
   ▼
DNS System
   │
   │
   ▼
IP Address
   │
   ▼
Browser connects to the server
```

A more detailed conceptual flow:

```text
Client
   │
   ▼
Browser / OS Cache
   │
   │ Not found
   ▼
DNS Resolver
   │
   │
   ▼
Root DNS Server
   │
   ▼
TLD DNS Server
   │
   ▼
Authoritative DNS Server
   │
   ▼
IP Address
```

The resolver can cache the result, so the complete lookup is not necessarily required every time.

---

## Important DNS Records

DNS stores different types of records.

### A Record

Maps a domain name to an IPv4 address.

```text
example.com
    │
    ▼
93.184.216.34
```

---

### AAAA Record

Maps a domain name to an IPv6 address.

```text
example.com
    │
    ▼
2001:db8::1
```

---

### CNAME Record

Maps one domain name to another domain name.

```text
api.example.com
       │
       ▼
service.example.com
       │
       ▼
IP Address
```

---

### MX Record

Defines mail servers for a domain.

```text
example.com
     │
     ▼
Mail Server
```

---

### TXT Record

Stores text information.

It is commonly used for:

- Domain verification
    
- Email configuration
    
- Security policies
    

---

## DNS and Backend Applications

Suppose your API communicates with another service:

```text
Your API
   │
   │ https://payment-service.com
   ▼
DNS
   │
   ▼
IP Address
   │
   ▼
Payment Service
```

Your application usually communicates using a **domain name**, while DNS finds the destination IP address.

---

# 5. HTTP

## What Is HTTP?

**HTTP (Hypertext Transfer Protocol)** is an application-layer protocol used for communication between clients and servers.

It follows a request-response model.

```text
Client
   │
   │ HTTP Request
   ▼
Server
   │
   │ HTTP Response
   ▼
Client
```

For a backend API:

```text
Frontend
   │
   │ GET /users
   ▼
Backend API
   │
   │ 200 OK + JSON
   ▼
Frontend
```

---

# 6. HTTP Request

An HTTP request usually contains:

```text
Request Line
Headers
Body (optional)
```

Example:

```http
POST /users HTTP/1.1
Host: api.example.com
Content-Type: application/json
Authorization: Bearer <token>

{
  "name": "Mohamed"
}
```

### Request Line

```text
POST /users HTTP/1.1
│    │      │
│    │      └── HTTP Version
│    └───────── Resource
└────────────── Method
```

---

# 7. HTTP Methods

HTTP methods describe the intended operation.

|Method|Common Purpose|
|---|---|
|GET|Retrieve data|
|POST|Create or process data|
|PUT|Replace or update a resource|
|PATCH|Partially update a resource|
|DELETE|Remove a resource|
|HEAD|Retrieve headers without a response body|
|OPTIONS|Discover communication options|

Example:

```text
GET    /users
GET    /users/10

POST   /users

PUT    /users/10

PATCH  /users/10

DELETE /users/10
```

---

# 8. HTTP Headers

Headers contain additional information about the request or response.

Example:

```http
GET /users HTTP/1.1
Host: api.example.com
Authorization: Bearer <token>
Accept: application/json
```

Common headers:

```text
Content-Type
Authorization
Accept
Cookie
Cache-Control
User-Agent
Host
```

A header describes metadata about the communication.

---

# 9. HTTP Request Body

The body contains data sent with the request.

Example:

```http
POST /users HTTP/1.1
Content-Type: application/json

{
  "name": "Mohamed",
  "email": "mohamed@example.com"
}
```

The request body can contain different formats, including:

```text
JSON
XML
Form Data
Multipart Data
Binary Data
```

---

# 10. HTTP Response

An HTTP response usually contains:

```text
Status Line
Headers
Body (optional)
```

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "name": "Mohamed"
}
```

---

# 11. HTTP Status Codes

Status codes describe the result of a request.

## 2xx — Success

```text
200 OK
201 Created
204 No Content
```

## 3xx — Redirection

```text
301 Moved Permanently
302 Found
304 Not Modified
```

## 4xx — Client Error

```text
400 Bad Request
401 Unauthorized
403 Forbidden
404 Not Found
409 Conflict
429 Too Many Requests
```

## 5xx — Server Error

```text
500 Internal Server Error
502 Bad Gateway
503 Service Unavailable
504 Gateway Timeout
```

---

# 12. HTTP Is Stateless

HTTP is **stateless**.

Each request is independent.

```text
Request 1 ───► Server
Request 2 ───► Server
Request 3 ───► Server
```

The protocol itself does not require the server to remember previous requests.

Applications can build state using mechanisms such as:

- Cookies
    
- Sessions
    
- Tokens
    
- Databases
    
- Distributed caches
    

For example:

```text
Client
   │
   │ Login
   ▼
Server
   │
   │ Token
   ▼
Client

--------------------

Client
   │
   │ Request + Token
   ▼
Server
```

---

# 13. HTTP Versions

HTTP has evolved over time.

```text
HTTP/1.0
   ↓
HTTP/1.1
   ↓
HTTP/2
   ↓
HTTP/3
```

At a high level:

### HTTP/1.1

Introduced persistent connections and became the foundation of modern web communication.

### HTTP/2

Improved performance with features such as multiplexing and more efficient header handling.

### HTTP/3

Uses QUIC as its transport instead of TCP.

```text
HTTP/1.1
     │
     ▼
TCP
     │
     ▼
IP


HTTP/2
     │
     ▼
TCP
     │
     ▼
IP


HTTP/3
     │
     ▼
QUIC
     │
     ▼
UDP
     │
     ▼
IP
```

---

# 14. HTTPS

## What Is HTTPS?

**HTTPS is HTTP secured using TLS.**

```text
HTTPS
  │
  ├── HTTP
  │
  └── TLS
        │
        ▼
       TCP
        │
        ▼
        IP
```

HTTPS provides security for communication between the client and server.

---

# 15. Why Do We Need HTTPS?

Without HTTPS:

```text
Client
   │
   │ HTTP
   ▼
Internet
   │
   ▼
Server
```

The communication is not protected by TLS encryption.

HTTPS helps provide:

- **Confidentiality** — others cannot easily read the transmitted data.
    
- **Integrity** — data cannot be modified without detection.
    
- **Authentication** — the client can verify the server's identity through certificates.
    

---

# 16. TLS Certificates

A server uses a **TLS certificate** to prove its identity.

Conceptually:

```text
Browser
   │
   │ Connect securely
   ▼
Server
   │
   │ Certificate
   ▼
Browser
   │
   │ Verify certificate
   ▼
Secure Communication
```

The certificate is associated with the server's domain name and is validated through a chain of trust.

---

# 17. Simplified TLS Handshake

Before encrypted HTTP communication begins, the client and server establish security parameters.

A simplified view:

```text
Client                          Server
   │                               │
   │ ─────── Hello ───────────────► │
   │                               │
   │ ◄──── Certificate ─────────── │
   │                               │
   │ Establish shared secrets      │
   │                               │
   │ ─────── Secure Connection ───►│
   │                               │
```

After the handshake:

```text
Client
   │
   │ Encrypted HTTP Data
   ▼
Server
```

TLS is responsible for securing the communication.

HTTP still defines the application-level request and response.

---

# 18. The Complete Flow

When you visit:

```text
https://api.example.com/users
```

a simplified flow looks like this:

```text
1. Browser needs the server address
        │
        ▼
2. DNS resolves api.example.com
        │
        ▼
3. Browser receives an IP address
        │
        ▼
4. Connection is established
        │
        ▼
5. TLS handshake happens
        │
        ▼
6. Secure connection is established
        │
        ▼
7. Browser sends an HTTP request
        │
        ▼
8. Server processes the request
        │
        ▼
9. Server sends an HTTP response
        │
        ▼
10. Browser receives the response
```

This flow connects multiple networking concepts together:

```text
Application Layer
│
├── DNS → Find the server
│
├── HTTP → Define communication
│
└── TLS → Secure communication
        │
        ▼
Transport Layer
│
└── TCP / UDP
        │
        ▼
Internet Layer
│
└── IP
        │
        ▼
Link Layer
│
└── Ethernet / Wi-Fi
```

---

# 19. Other Important Application Protocols

You do not need to study these in the same depth right now, but you should know what they do.

|Protocol|Purpose|
|---|---|
|SMTP|Sending email|
|IMAP|Accessing and synchronizing email|
|POP3|Retrieving email|
|FTP|File transfer|
|SSH|Secure remote access|
|DHCP|Automatically assigning network configuration|
|WebSocket|Persistent two-way communication|
|AMQP|Messaging between applications|
|MQTT|Lightweight messaging, commonly used in IoT|
|gRPC|High-performance remote procedure communication|
|LDAP|Accessing directory services|

---

# Key Takeaway

The Application Layer contains the protocols that applications use to communicate.

For a Backend Engineer, the most important concepts are:

```text
DNS
│
└── Finds where the server is.

HTTP
│
└── Defines how the client and server communicate.

HTTPS
│
└── HTTP communication secured by TLS.
```

When you access a backend API:

```text
https://api.example.com/users
        │
        ▼
DNS
Find the IP address
        │
        ▼
TLS
Create a secure connection
        │
        ▼
HTTP
Send the request
        │
        ▼
Backend Application
Process the request
        │
        ▼
HTTP Response
Return the result
```

Understanding this flow is essential because it connects the application you build with everything happening underneath the application.