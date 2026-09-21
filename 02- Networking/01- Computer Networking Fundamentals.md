# Computer Networking Fundamentals

> **Goal:** Understand what a computer network is, why it exists, how devices communicate, and the fundamental concepts required before studying TCP/IP, HTTP, DNS, and other networking protocols.

---

## 1. What Is a Computer Network?

A **computer network** is a group of devices connected together so they can **communicate and exchange data**.

These devices can include:

- Computers
    
- Servers
    
- Phones
    
- Routers
    
- Switches
    
- Printers
    
- IoT devices
    

A network allows devices to share:

- Data
    
- Applications
    
- Services
    
- Internet access
    
- Hardware resources
    

### Simple Example

```text
Computer A
     │
     │
     ▼
   Switch
     │
     ├────────── Computer B
     │
     └────────── Server
```

The devices can communicate because they are connected through a network.

---

# 2. Why Do We Need Networks?

Without networking, computers would operate mostly as isolated systems.

Networking allows systems to:

- Communicate with each other
    
- Share resources
    
- Access remote services
    
- Transfer files
    
- Access databases
    
- Access the Internet
    
- Build distributed applications
    

For a backend engineer, networking is fundamental because a backend application almost never works alone.

For example:

```text
Client
   │
   │ Network
   ▼
Backend API
   │
   │ Network
   ▼
Database
```

The API communicates with the client and database through networks.

---

# 3. Network Communication

At the most basic level, network communication means:

```text
Sender
   │
   │ Data
   ▼
Network
   │
   │ Data
   ▼
Receiver
```

For communication to work, both sides need to agree on **how the communication happens**.

These rules are called **protocols**.

---

# 4. What Is a Protocol?

A **protocol** is a set of rules that defines how devices communicate.

For example:

```text
HTTP
TCP
IP
DNS
TLS
```

Each protocol solves a different problem.

### Example

When you access:

```text
https://example.com
```

many protocols may be involved:

```text
HTTP
  ↓
TLS
  ↓
TCP
  ↓
IP
  ↓
Ethernet / Wi-Fi
```

Each layer has a responsibility.

---

# 5. Network Devices

Different devices perform different networking responsibilities.

## 5.1 Network Interface Card (NIC)

A **NIC** allows a device to connect to a network.

Examples:

- Ethernet adapter
    
- Wi-Fi adapter
    

```text
Computer
   │
   ▼
  NIC
   │
   ▼
 Network
```

---

## 5.2 Switch

A **switch** connects devices inside a local network.

```text
PC ─────┐
        │
PC ─────┼── Switch
        │
Server ─┘
```

A switch primarily operates using **MAC addresses**.

---

## 5.3 Router

A **router** connects different networks.

```text
Network A
    │
    ▼
 Router
    │
    ▼
Network B
```

For example:

```text
Home Network
     │
     ▼
  Router
     │
     ▼
   Internet
```

Routers use **IP addresses** to determine where packets should go.

---

## 5.4 Access Point

A wireless access point allows devices to connect to a network using Wi-Fi.

```text
Laptop
   )))
   )))
Access Point
     │
     ▼
  Network
```

---

# 6. Network Types

Networks can be classified by their scope.

## LAN — Local Area Network

A network covering a small area.

Examples:

- Home
    
- Office
    
- School
    

```text
PC ──┐
PC ──┼── Switch ── Router
PC ──┘
```

---

## WAN — Wide Area Network

A network covering a large geographical area.

The Internet is the largest example of an interconnected WAN.

```text
Network A
    │
    │
   WAN
    │
    │
Network B
```

---

## Internet

The **Internet is a global network of interconnected networks**.

It is not one single network.

Conceptually:

```text
Home Network
      │
      ▼
   ISP Network
      │
      ▼
Internet Backbone
      │
      ▼
   Data Center
      │
      ▼
    Server
```

---

# 7. Addressing

Devices need identifiers so that data can be delivered to the correct destination.

Networking commonly uses different types of addresses.

### MAC Address

Identifies a network interface at the link layer.

Example:

```text
00:1A:2B:3C:4D:5E
```

### IP Address

Identifies a device/interface within an IP network.

Example:

```text
192.168.1.10
```

Later we will study:

- IPv4
    
- IPv6
    
- Public IP
    
- Private IP
    
- Subnetting
    
- CIDR
    

---

# 8. Packets

Data sent across a network is divided into smaller units.

One important concept is the **packet**.

Instead of sending:

```text
Large Data
```

as one enormous piece, networking protocols can break data into smaller pieces:

```text
Data
 │
 ├── Packet 1
 ├── Packet 2
 ├── Packet 3
 └── Packet 4
```

Packets contain information needed to deliver and process the data.

Conceptually:

```text
┌─────────────────────────────┐
│ Header                      │
├─────────────────────────────┤
│ Data / Payload              │
└─────────────────────────────┘
```

Different protocols add different headers.

---

# 9. Ports

An IP address identifies a network endpoint, but a computer can run many network applications at the same time.

**Ports** help identify the destination application/service.

For example:

```text
192.168.1.10:80
192.168.1.10:443
192.168.1.10:5000
```

The IP identifies the host.

The port identifies the service/application endpoint.

Conceptually:

```text
IP Address
     │
     ▼
192.168.1.10
     │
     ├── :80     HTTP
     ├── :443    HTTPS
     └── :5000   Application
```

---

# 10. Client and Server

Many network applications use a **client-server model**.

```text
Client
   │
   │ Request
   ▼
Server
   │
   │ Response
   ▼
Client
```

### Client

The system that initiates communication.

Examples:

- Browser
    
- Mobile application
    
- Another backend service
    

### Server

The system that provides a service.

Examples:

- Web server
    
- API server
    
- Database server
    

---

# 11. Request and Response

A common communication pattern is:

```text
Client
   │
   │ Request
   ▼
Server
   │
   │ Response
   ▼
Client
```

For example:

```text
Client
   │
   │ GET /users
   ▼
API Server
   │
   │ 200 OK
   │
   │ Users
   ▼
Client
```

HTTP builds on this basic communication model.

---

# 12. Connection-Oriented vs Connectionless

Different protocols provide different communication models.

### Connection-Oriented

A connection is established before data is exchanged.

Example:

```text
Client
   │
   │ Establish connection
   ▼
Server
   │
   │ Data exchange
   ▼
Client
```

TCP is connection-oriented.

### Connectionless

Data can be sent without establishing a connection first.

UDP is connectionless.

```text
Client ───────────► Server
```

We will study TCP and UDP in detail later.

---

# 13. Reliable vs Unreliable Communication

Networking protocols can provide different guarantees.

A reliable transport protocol may provide:

- Ordered delivery
    
- Retransmission
    
- Error detection
    
- Flow control
    
- Congestion control
    

TCP provides many of these guarantees.

UDP provides a simpler transport model with fewer guarantees.

---

# 14. Network Layers

Networking is divided into layers to separate responsibilities.

Two important models are:

### OSI Model

```text
7. Application
8. Presentation
9. Session
10. Transport
11. Network
12. Data Link
13. Physical
```

### TCP/IP Model

```text
Application
Transport
Internet
Link
```

The layers help us reason about networking without putting every responsibility into one protocol.

---

# 15. Encapsulation

When data travels through the network, each layer can add its own information.

Conceptually:

```text
Application Data
       ↓
Transport Header + Data
       ↓
IP Header + Transport Header + Data
       ↓
Link Header + IP Header + Transport Header + Data
```

At the destination, the process is reversed.

```text
Received Frame
      ↓
Remove Link information
      ↓
Remove IP information
      ↓
Remove Transport information
      ↓
Application receives data
```

This process is called **encapsulation and decapsulation**.

---

# 16. The Big Picture

A simplified network communication path looks like this:

```text
Application
     │
     ▼
HTTP
     │
     ▼
TCP / UDP
     │
     ▼
IP
     │
     ▼
Ethernet / Wi-Fi
     │
     ▼
Physical Network
```

At the receiving machine:

```text
Physical Network
     │
     ▼
Ethernet / Wi-Fi
     │
     ▼
IP
     │
     ▼
TCP / UDP
     │
     ▼
HTTP
     │
     ▼
Application
```

---

# 17. What You Should Understand Before Moving On

Before continuing to deeper networking topics, you should understand these concepts:

- What a computer network is
    
- Why networks exist
    
- Client and server
    
- Network devices
    
- LAN and WAN
    
- Internet
    
- Protocols
    
- MAC addresses
    
- IP addresses
    
- Ports
    
- Packets
    
- Request and response
    
- TCP vs UDP
    
- Reliable vs unreliable communication
    
- Network layers
    
- Encapsulation
    
- Basic network topology
    

---


> **Core idea:** Networking is not just about memorizing protocols. The goal is to understand **how data moves from one process on one machine to another process on another machine.**