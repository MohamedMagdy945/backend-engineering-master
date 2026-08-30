# OSI Model and TCP/IP Model

> **Goal:** Understand how network communication is organized into layers and how data moves between applications across a network.

---

# 1. Why Do We Need Network Models?

Network communication is complex.

When one application sends data to another application, many things must happen:

- The application creates the data.
    
- The data needs to be prepared for transmission.
    
- The destination application must be identified.
    
- The destination device must be located.
    
- The data must travel through networks.
    
- The receiving device must process the data.
    
- The receiving application must receive the correct data.
    

Instead of putting all these responsibilities into one system, networking separates them into **layers**.

```text
Application
     │
     ▼
Transport
     │
     ▼
Network
     │
     ▼
Data Link
     │
     ▼
Physical Network
```

Each layer has its own responsibility.

---

# 2. What Is a Layered Model?

A layered model divides networking into separate responsibilities.

For example:

```text
┌──────────────────────────┐
│ Application Layer        │  What data does the application need?
├──────────────────────────┤
│ Transport Layer          │  Which application/process?
├──────────────────────────┤
│ Network Layer            │  Which network/device?
├──────────────────────────┤
│ Data Link Layer          │  Which device on the local network?
├──────────────────────────┤
│ Physical Layer           │  How are bits physically transmitted?
└──────────────────────────┘
```

Each layer:

- Has a specific responsibility.
    
- Provides services to the layer above it.
    
- Uses services from the layer below it.
    

---

# 3. The OSI Model

The **OSI (Open Systems Interconnection) Model** is a conceptual model that divides network communication into seven layers.

```text
┌──────────────────────────┐
│ 7. Application           │
├──────────────────────────┤
│ 6. Presentation          │
├──────────────────────────┤
│ 5. Session               │
├──────────────────────────┤
│ 4. Transport             │
├──────────────────────────┤
│ 3. Network               │
├──────────────────────────┤
│ 2. Data Link             │
├──────────────────────────┤
│ 1. Physical              │
└──────────────────────────┘
```

The OSI model is mainly used to understand and reason about networking.

---

# 4. OSI Layers

## Layer 7 — Application

The Application layer is where network services are provided to applications.

Examples:

```text
HTTP
HTTPS
DNS
SMTP
FTP
```

For a backend engineer, this is where many familiar protocols exist.

Example:

```text
Browser
   │
   │ HTTP Request
   ▼
Backend API
```

---

## Layer 6 — Presentation

The Presentation layer is responsible for how data is represented.

It can involve:

- Data formatting
    
- Serialization
    
- Encryption
    
- Compression
    

Examples:

```text
JSON
XML
UTF-8
Encryption
Compression
```

Conceptually:

```text
Application Data
      │
      ▼
Convert / Encode / Encrypt
      │
      ▼
Ready for transmission
```

---

## Layer 5 — Session

The Session layer is responsible for managing communication sessions between systems.

Its responsibilities conceptually include:

- Establishing sessions
    
- Maintaining sessions
    
- Synchronizing communication
    
- Terminating sessions
    

In modern networking, Session and Presentation responsibilities are often handled by applications or other protocols rather than existing as clearly separate layers.

---

## Layer 4 — Transport

The Transport layer provides communication between **processes and applications**.

Important concepts:

- TCP
    
- UDP
    
- Ports
    
- Reliability
    
- Ordering
    
- Flow control
    
- Multiplexing
    

Example:

```text
Client
192.168.1.10
Port: 5000
      │
      │ TCP
      ▼
Server
10.0.0.5
Port: 443
```

The Transport layer helps deliver data to the correct application.

---

## Layer 3 — Network

The Network layer is responsible for communication between different networks.

Important concepts:

- IP addresses
    
- Routing
    
- Routers
    
- Packets
    

Example:

```text
Network A
    │
    ▼
 Router
    │
    ▼
Network B
```

IP operates at this layer.

The main question is:

> How can this data reach the destination network and host?

---

## Layer 2 — Data Link

The Data Link layer handles communication within a local network.

Important concepts:

- MAC addresses
    
- Ethernet
    
- Switches
    
- Frames
    

Example:

```text
PC ────── Switch ────── Server
```

The Data Link layer helps devices communicate over the same local network.

---

## Layer 1 — Physical

The Physical layer represents how raw bits are transmitted.

Examples:

- Ethernet cables
    
- Fiber optic cables
    
- Radio signals
    
- Wi-Fi signals
    

Conceptually:

```text
Bits
 │
 ▼
Electrical / Optical / Radio Signals
 │
 ▼
Network
```

---

# 5. OSI Model Summary

```text
┌────────────────────────────────────────────┐
│ 7. Application   → HTTP, DNS, SMTP         │
├────────────────────────────────────────────┤
│ 6. Presentation  → Encoding, Encryption    │
├────────────────────────────────────────────┤
│ 5. Session       → Session Management      │
├────────────────────────────────────────────┤
│ 4. Transport     → TCP, UDP, Ports         │
├────────────────────────────────────────────┤
│ 3. Network       → IP, Routing             │
├────────────────────────────────────────────┤
│ 2. Data Link     → MAC, Ethernet, Frames   │
├────────────────────────────────────────────┤
│ 1. Physical      → Signals, Cables, Wi-Fi  │
└────────────────────────────────────────────┘
```

---

# 6. The TCP/IP Model

The **TCP/IP model** is the practical model used by the Internet.

It is usually represented using four layers.

```text
┌──────────────────────────┐
│ Application              │
├──────────────────────────┤
│ Transport                │
├──────────────────────────┤
│ Internet                 │
├──────────────────────────┤
│ Network Access / Link    │
└──────────────────────────┘
```

Unlike the OSI model, TCP/IP combines some responsibilities into the same layer.

---

# 7. TCP/IP Layers

## Application Layer

The Application layer contains protocols used directly by applications.

Examples:

```text
HTTP
HTTPS
DNS
SMTP
FTP
SSH
WebSocket
```

This layer combines the responsibilities of:

```text
OSI Application
OSI Presentation
OSI Session
```

---

## Transport Layer

The Transport layer is responsible for communication between processes.

Protocols:

```text
TCP
UDP
```

It corresponds directly to:

```text
OSI Transport Layer
```

Important concept:

```text
IP Address + Port
```

For example:

```text
192.168.1.10:443
```

---

## Internet Layer

The Internet layer is responsible for moving packets between networks.

The main protocol is:

```text
IP
```

Other important protocols include:

```text
ICMP
```

This corresponds mainly to:

```text
OSI Network Layer
```

---

## Network Access / Link Layer

This layer handles communication over the local network and physical medium.

Examples:

```text
Ethernet
Wi-Fi
MAC Addresses
```

It combines:

```text
OSI Data Link
+
OSI Physical
```

---

# 8. OSI vs TCP/IP

```text
OSI Model                    TCP/IP Model

┌───────────────┐
│ Application   │
├───────────────┤
│ Presentation  │ ───────────┐
├───────────────┤            │
│ Session       │            ▼
└───────────────┘       ┌───────────────┐
                        │ Application   │
┌───────────────┐       ├───────────────┤
│ Transport     │ ────► │ Transport     │
├───────────────┤       ├───────────────┤
│ Network       │ ────► │ Internet      │
├───────────────┤       ├───────────────┤
│ Data Link     │ ──────┤               │
├───────────────┤       │ Link          │
│ Physical      │ ──────┤               │
└───────────────┘       └───────────────┘
```

### Simple Comparison

|OSI|TCP/IP|
|---|---|
|7 layers|4 layers|
|Conceptual model|Practical Internet model|
|Application|Application|
|Presentation|Application|
|Session|Application|
|Transport|Transport|
|Network|Internet|
|Data Link|Link|
|Physical|Link|

---

# 9. How Data Moves Through the Layers

Imagine a browser sending a request to a backend API.

```text
Browser
   │
   ▼
HTTP Request
   │
   ▼
TCP
   │
   ▼
IP
   │
   ▼
Ethernet / Wi-Fi
   │
   ▼
Network
```

Each layer adds information required for its responsibility.

This is called **encapsulation**.

---

# 10. Encapsulation

Suppose the application creates this data:

```text
GET /users
```

The Application layer creates the HTTP message:

```text
┌──────────────────────┐
│ HTTP Request         │
└──────────────────────┘
```

The Transport layer adds a TCP header:

```text
┌──────────────┬──────────────────────┐
│ TCP Header   │ HTTP Request         │
└──────────────┴──────────────────────┘
```

The Internet layer adds an IP header:

```text
┌───────────┬──────────────┬──────────────────────┐
│ IP Header │ TCP Header   │ HTTP Request         │
└───────────┴──────────────┴──────────────────────┘
```

The Link layer adds its header:

```text
┌─────────────┬───────────┬────────────┬──────────────┐
│ Link Header │ IP Header │ TCP Header │ HTTP Data    │
└─────────────┴───────────┴────────────┴──────────────┘
```

The data is then transmitted through the network.

---

# 11. Decapsulation

When the data reaches the destination, the process happens in reverse.

```text
Network Data
     │
     ▼
Remove Link Information
     │
     ▼
Remove IP Information
     │
     ▼
Remove TCP Information
     │
     ▼
HTTP Request
     │
     ▼
Application
```

This is called **decapsulation**.

---

# 12. The Complete Picture

When a browser communicates with a backend server:

```text
CLIENT                                              SERVER

Application                                         Application
HTTP                                                HTTP
   │                                                   ▲
   ▼                                                   │
Transport                                           Transport
TCP                                                 TCP
   │                                                   ▲
   ▼                                                   │
Internet                                            Internet
IP                                                  IP
   │                                                   ▲
   ▼                                                   │
Link                                                Link
Ethernet / Wi-Fi                                    Ethernet
   │                                                   ▲
   └────────────── Network ────────────────────────────┘
```

The sender moves data **down** through the network stack.

The receiver moves data **up** through the network stack.

---

# 13. Important Terms

|Term|Meaning|
|---|---|
|Protocol|Rules that define communication|
|Layer|A separated networking responsibility|
|Encapsulation|Adding protocol information as data moves down the stack|
|Decapsulation|Removing protocol information as data moves up the stack|
|Payload|The actual data carried by a protocol|
|Header|Control information added by a protocol|
|Port|Identifies a network application/process|
|IP Address|Identifies a host/interface on an IP network|
|MAC Address|Identifies a network interface on the local link|

---

# Key Takeaway

The most important thing to understand is:

```text
Application Data
       │
       ▼
Application Protocol
       │
       ▼
Transport Protocol
       │
       ▼
Internet Protocol
       │
       ▼
Link Protocol
       │
       ▼
Physical Network
```

The **OSI model** helps us understand networking conceptually.

The **TCP/IP model** describes the practical architecture used by modern Internet communication.

Each layer solves a different problem, and together they allow:

```text
Application on Computer A
            │
            │ Network Communication
            ▼
Application on Computer B
```