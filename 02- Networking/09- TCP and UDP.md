# TCP and UDP

## What Are TCP and UDP?

**TCP** and **UDP** are communication protocols that work at the **Transport Layer**.

Their main job is to move data between applications running on different devices.

```text
Application
    │
    │ HTTP / DNS / etc.
    ▼
Transport Layer
┌─────────────┐
│ TCP or UDP  │
└─────────────┘
    │
    ▼
Internet Layer
    │
    │ IP
    ▼
Network
```

Both TCP and UDP usually work on top of **IP**.

```text
HTTP
  ↓
TCP
  ↓
IP
  ↓
Ethernet / Wi-Fi
```

Or:

```text
DNS
 ↓
UDP
 ↓
IP
 ↓
Ethernet / Wi-Fi
```

---

# TCP

**TCP (Transmission Control Protocol)** is a connection-oriented protocol that provides reliable data delivery.

TCP tries to make sure that:

- Data arrives successfully
    
- Data arrives in the correct order
    
- Lost data can be retransmitted
    
- Duplicate data is handled
    
- Both sides are ready to communicate
    

## TCP Connection

Before sending application data, TCP establishes a connection.

This is commonly done using the **Three-Way Handshake**.

```text
Client                          Server

  SYN
   ─────────────────────────────►

             SYN + ACK
   ◄─────────────────────────────

  ACK
   ─────────────────────────────►

        Connection Established
```

After the connection is established, data can be exchanged.

---

## TCP Reliability

Imagine an application sends:

```text
Message 1
Message 2
Message 3
```

TCP uses sequence numbers to help ensure that the receiver can reconstruct the data in the correct order.

```text
Sender                        Receiver

Data #1  ────────────────────►
Data #2  ────────────────────►
Data #3  ────────────────────►
```

The receiver sends acknowledgments.

```text
Sender                        Receiver

Data #1  ────────────────────►

          ◄────────────────── ACK #1
```

If data is lost:

```text
Sender                        Receiver

Data #1  ────────────────────►

Data #2  ─────── X Lost

Data #3  ────────────────────►
```

TCP can detect the missing data and retransmit it.

```text
Data #2  ────────────────────►
```

---

# UDP

**UDP (User Datagram Protocol)** is a connectionless protocol.

Unlike TCP, UDP does not establish a connection before sending data.

```text
Application
     │
     ▼
Send Data
     │
     ▼
UDP
     │
     ▼
IP
     │
     ▼
Network
```

UDP simply sends data to the destination.

```text
Client                          Server

Data
   ─────────────────────────────►
```

There is no built-in guarantee that:

- The data will arrive
    
- The data will arrive in order
    
- Lost data will be retransmitted
    

Because UDP has less overhead, it can be useful when speed and low latency are more important than perfect reliability.

---

# TCP vs UDP

|Feature|TCP|UDP|
|---|---|---|
|Connection|Connection-oriented|Connectionless|
|Reliability|Reliable|No delivery guarantee|
|Data order|Guaranteed|Not guaranteed|
|Retransmission|Yes|No|
|Acknowledgments|Yes|No|
|Overhead|Higher|Lower|
|Speed / Latency|Usually higher overhead|Usually lower overhead|
|Example use|Web, APIs, databases|DNS, gaming, streaming|

---

# Ports

TCP and UDP use **port numbers** to identify the destination application.

For example:

```text
IP Address: 192.168.1.10
Port:       443
```

The IP address identifies the device.

The port identifies the application or service.

```text
Internet
    │
    ▼
192.168.1.10
    │
    ├── Port 80   → HTTP
    ├── Port 443  → HTTPS
    ├── Port 53   → DNS
    └── Port 3306 → MySQL
```

A communication endpoint is often represented as:

```text
IP Address + Port
```

For example:

```text
192.168.1.10:443
```

---

# TCP and UDP with HTTP

Traditionally, HTTP commonly uses TCP.

```text
Browser
   │
   │ HTTP Request
   ▼
TCP
   │
   ▼
IP
   │
   ▼
Internet
```

For example:

```text
GET /users HTTP/1.1
```

The HTTP request is application data.

TCP is responsible for reliably transporting that data.

---

# Simple Analogy

## TCP

TCP is like sending an important package.

```text
Send Package
      ↓
Receiver Confirms Delivery
      ↓
If Lost → Send Again
```

You care about making sure the package arrives.

---

## UDP

UDP is like broadcasting information.

```text
Send Message
      ↓
Move On
```

You send the message without waiting for confirmation.

If a message is lost, UDP itself does not automatically resend it.

---

# TCP and UDP Are Not Application Protocols

It is important to understand that TCP and UDP are different from protocols such as HTTP.

```text
HTTP → Application Protocol
TCP  → Transport Protocol
IP   → Internet Protocol
```

They work together.

Example:

```text
┌─────────────────────┐
│ HTTP                │
│ "GET /users"        │
├─────────────────────┤
│ TCP                 │
│ Reliable Transport  │
├─────────────────────┤
│ IP                  │
│ Addressing/Routing  │
├─────────────────────┤
│ Ethernet / Wi-Fi    │
│ Physical Delivery   │
└─────────────────────┘
```

---

# Summary

```text
TCP
├── Connection-oriented
├── Reliable
├── Ordered delivery
├── Retransmits lost data
└── Higher overhead

UDP
├── Connectionless
├── Lightweight
├── No delivery guarantee
├── No ordering guarantee
└── Lower overhead
```

The most important idea is:

> **TCP and UDP are responsible for transporting data between applications.**

TCP focuses on **reliability**.

UDP focuses on **simplicity and low overhead**.

The application chooses which transport protocol is appropriate depending on its requirements.