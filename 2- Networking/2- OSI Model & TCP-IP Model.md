# OSI Model & TCP/IP Model

## OSI Model

The **OSI (Open Systems Interconnection) Model** is a conceptual model that explains how network communication works by dividing the communication process into **7 layers**.

### The 7 Layers

1. Application
2. Presentation
3. Session
4. Transport
5. Network
6. Data Link
7. Physical

### 7. Application Layer

The Application Layer provides network services directly to applications.

Examples:

- HTTP
- HTTPS
- DNS
- SMTP
- FTP
- SSH

It is where applications interact with network protocols.

### 6. Presentation Layer

The Presentation Layer is responsible for the representation and transformation of data.

Responsibilities:

- Data encoding
- Data decoding
- Encryption
- Decryption
- Compression
- Decompression

Its purpose is to make data understandable between different systems.

### 5. Session Layer

The Session Layer manages communication sessions between applications.

Responsibilities:

- Establishing sessions
- Maintaining sessions
- Terminating sessions
- Synchronizing communication

A session represents an ongoing communication between two systems.

### 4. Transport Layer

The Transport Layer provides **end-to-end communication** between applications.

Responsibilities:

- Segmentation
- Reliability
- Flow control
- Error recovery
- Port addressing

Main protocols:

- TCP
- UDP

#### TCP

TCP provides reliable and ordered communication.

It provides:

- Reliable delivery
- Ordered data
- Retransmission
- Flow control

#### UDP

UDP provides connectionless communication with lower overhead.

UDP does not guarantee:

- Delivery
- Ordering
- Retransmission

### 3. Network Layer

The Network Layer is responsible for **logical addressing and routing** packets between different networks.

Main protocol:

- IP

Responsibilities:

- Logical addressing
- Routing
- Packet forwarding

### 2. Data Link Layer

The Data Link Layer provides communication between devices on the same local network.

Responsibilities:

- Frames
- MAC addresses
- Local delivery
- Error detection

Examples:

- Ethernet
- Wi-Fi

### 1. Physical Layer

The Physical Layer is responsible for transmitting raw bits over a physical medium.

Examples:

- Ethernet cables
- Fiber optic cables
- Radio signals
- Network hardware

At this layer, data is transmitted as bits:

```
1011010010110101
```

---

## Encapsulation

Encapsulation is the process of adding information to data as it moves down through the OSI layers.

```
Application Data
       ↓
Transport Segment
       ↓
Network Packet
       ↓
Data Link Frame
       ↓
Physical Bits
```

---

## Decapsulation

Decapsulation is the opposite of encapsulation.

When the receiver gets the data, the information added by each layer is processed and removed as the data moves upward.

```
Bits
 ↓
Frame
 ↓
Packet
 ↓
Segment
 ↓
Data
```

# TCP/IP Model

The **TCP/IP Model** is a practical networking model used to describe how communication works on the Internet.

It is commonly represented using **4 layers**:

1. Application
2. Transport
3. Internet
4. Network Access

### 4. Application Layer

The TCP/IP Application Layer handles application-level network communication.

It combines the responsibilities of the OSI:

- Application
- Presentation
- Session

Examples:

- HTTP
- HTTPS
- DNS
- SMTP
- FTP
- SSH

### 3. Transport Layer

The Transport Layer provides end-to-end communication between applications.

Main protocols:

- TCP
- UDP

TCP provides reliable and ordered communication.

UDP provides connectionless communication with lower overhead.

### 2. Internet Layer

The Internet Layer is responsible for logical addressing and routing.

Main protocol:

- IP

Its main responsibilities are:

- IP addressing
- Routing
- Packet forwarding

### 1. Network Access Layer

The Network Access Layer handles communication over the local network and physical transmission.

It combines responsibilities from the OSI:

- Data Link
- Physical

Examples:

- Ethernet
- Wi-Fi

# OSI Model vs TCP/IP Model

```
OSI Model                  TCP/IP Model

Application      ┐
Presentation     ├───────→ Application
Session          ┘

Transport        ────────→ Transport

Network          ────────→ Internet

Data Link        ┐
Physical         ┘───────→ Network Access
```

|OSI Model|TCP/IP Model|
|---|---|
|7 layers|4 layers|
|Conceptual reference model|Practical networking model|
|Application, Presentation, Session are separate|Combined into Application|
|Data Link and Physical are separate|Combined into Network Access|
|Useful for understanding networking|Used to describe Internet communication|

## Layer Mapping

|OSI|TCP/IP|
|---|---|
|Application|Application|
|Presentation|Application|
|Session|Application|
|Transport|Transport|
|Network|Internet|
|Data Link|Network Access|
|Physical|Network Access|

## Main Idea

The OSI Model explains networking using **7 layers**:

```
Application
Presentation
Session
Transport
Network
Data Link
Physical
```

The TCP/IP Model explains networking using **4 layers**:

```
Application
Transport
Internet
Network Access
```

The two models describe the same general communication process from different perspectives.