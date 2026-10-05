# Transport Layer

The **Transport Layer** is responsible for communication between applications running on different computers.

While the **Network Layer** delivers data from one machine to another using IP addresses, the Transport Layer delivers data to the correct **application or process**.

```text
Application
     │
     ▼
Transport Layer
(TCP / UDP)
     │
     ▼
Network Layer
(IP)
     │
     ▼
Internet
```

## What Does the Transport Layer Do?

The Transport Layer is responsible for:

- Process-to-process communication
    
- Using ports to identify applications
    
- Reliable communication with TCP
    
- Fast communication with UDP
    
- Breaking data into smaller pieces
    
- Reassembling received data
    
- Flow control
    
- Error detection
    

The main Transport Layer protocols are:

```text
TCP
UDP
```

---

# TCP

**TCP (Transmission Control Protocol)** is a **connection-oriented** protocol.

Before two applications exchange data, TCP establishes a connection.

TCP focuses on:

- Reliability
    
- Ordered delivery
    
- Error detection
    
- Flow control
    
- Congestion control
    

## TCP Connection

A TCP connection starts with the **Three-Way Handshake**.

```text
Client                      Server

  SYN  ───────────────────►

       ◄───────────────────  SYN + ACK

  ACK  ───────────────────►

Connection Established
```

### Step 1: SYN

The client sends a `SYN` packet.

```text
Client:
"I want to establish a connection."
```

### Step 2: SYN + ACK

The server accepts the request.

```text
Server:
"I received your request, and I am ready."
```

### Step 3: ACK

The client confirms.

```text
Client:
"Connection confirmed."
```

Now both sides can exchange data.

---

## TCP Reliability

TCP guarantees that data is delivered:

- Reliably
    
- In the correct order
    
- Without duplication
    

For example:

```text
Application sends:

Message 1
Message 2
Message 3
```

Even if packets arrive like this:

```text
Message 2
Message 1
Message 3
```

TCP can reorder them before delivering them to the application.

TCP uses:

- Sequence Numbers
    
- Acknowledgments
    
- Retransmission
    

Example:

```text
Client ───── Data #1 ─────► Server

Client ◄──── ACK #1 ─────── Server
```

If the sender does not receive an acknowledgment, TCP may resend the data.

---

# UDP

**UDP (User Datagram Protocol)** is a **connectionless** protocol.

UDP does not establish a connection before sending data.

```text
Client ───── Data ─────► Server
```

UDP is simpler and usually has less overhead than TCP.

However, UDP does **not guarantee**:

- Delivery
    
- Order
    
- Retransmission
    

A UDP packet may:

```text
Arrive
Arrive late
Arrive out of order
Never arrive
```

---

## TCP vs UDP

|TCP|UDP|
|---|---|
|Connection-oriented|Connectionless|
|Reliable|Best effort|
|Ordered delivery|No ordering guarantee|
|Retransmits lost data|No automatic retransmission|
|More overhead|Less overhead|
|Usually slower|Usually faster|
|Uses handshake|No handshake|

Examples:

### TCP

Commonly used when reliability is important:

```text
HTTP
HTTPS
Database connections
SSH
File transfer
```

### UDP

Commonly used when low latency is more important:

```text
DNS
Online games
Voice calls
Video streaming
Real-time applications
```

---

# Ports

An **IP address identifies a machine**.

But one machine can run many applications.

For example:

```text
Computer
│
├── Browser
├── Web Server
├── Database
├── SSH Server
└── Email Server
```

The IP address tells the network:

```text
Which machine?
```

The **port number** tells the operating system:

```text
Which application or service?
```

Example:

```text
192.168.1.10:443
```

```text
192.168.1.10 → Machine
443          → Port
```

A common example:

```text
https://example.com
```

HTTPS usually uses:

```text
Port 443
```

HTTP usually uses:

```text
Port 80
```

---

## Common Ports

|Protocol / Service|Port|
|---|--:|
|HTTP|80|
|HTTPS|443|
|SSH|22|
|DNS|53|
|SMTP|25|
|FTP|21|
|SQL Server|1433|

---

## Server and Client Ports

A server usually listens on a known port.

For example:

```text
Client
   │
   │ Request
   ▼
Server
IP: 192.168.1.10
Port: 5000
```

In ASP.NET Core:

```text
http://localhost:5000
```

Your application is listening on:

```text
IP Address: localhost
Port: 5000
```

The operating system receives network traffic and forwards the data to the process listening on that port.

```text
Internet
    │
    ▼
Operating System
    │
    ▼
Port 5000
    │
    ▼
ASP.NET Core Application
```

---

# What Is a Socket?

A **socket** is an endpoint used for network communication.

It allows an application to send and receive data through the network.

A socket is commonly associated with:

```text
IP Address + Port + Protocol
```

Example:

```text
TCP Socket

192.168.1.10:5000
```

A client can connect to a server:

```text
Client Socket
192.168.1.20:53000
        │
        │ TCP Connection
        ▼
Server Socket
192.168.1.10:5000
```

The client usually receives a temporary port called an **ephemeral port**.

Example:

```text
Client

192.168.1.20:53000
```

The server listens on a well-known or configured port:

```text
Server

192.168.1.10:5000
```

---

# TCP Connection Identification

A TCP connection is identified using multiple values:

```text
Protocol
Source IP
Source Port
Destination IP
Destination Port
```

Example:

```text
TCP
192.168.1.20:53000
        │
        ▼
192.168.1.10:443
```

This allows the server to handle many clients connecting to the same port.

```text
Client A
192.168.1.20:50001
        │
        ▼
Server
192.168.1.10:443


Client B
192.168.1.30:50002
        │
        ▼
Server
192.168.1.10:443
```

Both clients connect to the same server and port, but each connection is different.

---

# How Everything Works Together

Imagine opening a website:

```text
https://example.com
```

The process is simplified as follows:

```text
1. Browser wants to connect to example.com

2. DNS finds the server IP address

   example.com
        │
        ▼
   93.xxx.xxx.xxx

3. Browser creates a connection

   Client IP: 192.168.1.20
   Client Port: 53000

4. Browser connects to:

   Server IP: 93.xxx.xxx.xxx
   Server Port: 443

5. TCP establishes a connection

   SYN
   SYN + ACK
   ACK

6. TLS secures the connection

7. HTTP request is sent

8. Server processes the request

9. HTTP response is returned
```

The complete flow:

```text
Browser
   │
   │ HTTP Request
   ▼
TCP
   │
   │ Source Port: 53000
   │ Destination Port: 443
   ▼
IP
   │
   ▼
Internet
   │
   ▼
Server IP
   │
   ▼
Port 443
   │
   ▼
Web Server
   │
   ▼
Application
```

---

# Important Concept

A useful way to remember the difference is:

```text
IP Address
=
Which computer?
```

```text
Port
=
Which application?
```

```text
Socket
=
A communication endpoint used by an application
```

```text
TCP
=
Reliable communication
```

```text
UDP
=
Fast, lightweight communication without delivery guarantees
```

---

# Summary

The Transport Layer allows applications to communicate over a network.

```text
Network Layer
IP Address
↓
Find the correct machine

Transport Layer
TCP / UDP + Ports
↓
Find the correct application
```

The main concepts are:

- **TCP** provides reliable, ordered communication.
    
- **UDP** provides lightweight, connectionless communication.
    
- **Ports** identify services or applications on a machine.
    
- **Sockets** are communication endpoints used by applications.
    
- A TCP connection is identified by the source and destination IP addresses and ports.