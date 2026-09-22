# Network Layer — IP Addressing & Subnetting

The **Network Layer** is responsible for delivering data between different networks.

Its main job is answering:

```text
Where should this packet go?
```

The most important concepts are:

- IP Addressing
    
- Network and Host portions
    
- Subnet Masks
    
- CIDR Notation
    
- Subnetting
    
- Default Gateway
    
- Routing
    

---

# 1. What Is an IP Address?

An **IP address** is a logical address used to identify a device on a network.

Example:

```text
192.168.1.10
```

IPv4 consists of **32 bits**.

```text
192        .168        .1          .10
11000000   10101000    00000001    00001010
```

Each section is called an **octet**.

```text
192.168.1.10

 192  →  8 bits
 168  →  8 bits
 1    →  8 bits
 10   →  8 bits

Total = 32 bits
```

---

# 2. Network Part and Host Part

An IP address has two logical parts:

```text
Network Part | Host Part
```

For example:

```text
192.168.1.10/24
```

The `/24` means:
 
```text
First 24 bits  → Network
Last 8 bits    → Hosts
```

```text
192.168.1.10

11000000.10101000.00000001 | 00001010
<--------- Network --------> < Host >
```

So:

```text
Network = 192.168.1.0
Host    = 10
```

Devices in the same network can communicate directly.

Example:

```text
192.168.1.10/24
192.168.1.20/24
192.168.1.50/24
```

They all belong to:

```text
192.168.1.0/24
```

---

# 3. Subnet Mask

A **Subnet Mask** determines which part of an IP address represents the network and which part represents the host.

Example:

```text
IP Address:

192.168.1.10

Subnet Mask:

255.255.255.0
```

In binary:

```text
255.255.255.0

11111111.11111111.11111111.00000000
```

```text
1 = Network bit
0 = Host bit
```

Therefore:

```text
11111111.11111111.11111111.00000000
<----------- Network -----------> <Host>
```

This subnet mask is also written as:

```text
/24
```

So these mean exactly the same thing:

```text
192.168.1.10
255.255.255.0
```

```text
192.168.1.10/24
```

---

# 4. CIDR Notation

CIDR notation uses a slash followed by the number of network bits.

Example:

```text
192.168.1.10/24
```

Common examples:

|CIDR|Subnet Mask|Usable Hosts|
|---|---|--:|
|/8|255.0.0.0|16,777,214|
|/16|255.255.0.0|65,534|
|/24|255.255.255.0|254|
|/25|255.255.255.128|126|
|/26|255.255.255.192|62|
|/27|255.255.255.224|30|
|/28|255.255.255.240|14|
|/29|255.255.255.248|6|
|/30|255.255.255.252|2|

For a traditional IPv4 subnet:

```text
Usable Hosts = 2^(Host Bits) - 2
```

The `-2` represents:

```text
Network Address
Broadcast Address
```

---

# 5. Network Address

The **Network Address** identifies the network itself.

Example:

```text
IP Address:

192.168.1.10/24
```

Network:

```text
192.168.1.0
```

The network address cannot normally be assigned to a host.

```text
192.168.1.0  → Network Address
192.168.1.1  → Host
192.168.1.2  → Host
...
192.168.1.254 → Host
192.168.1.255 → Broadcast Address
```

---

# 6. Broadcast Address

The **Broadcast Address** is used to send traffic to all devices in the subnet.

For:

```text
192.168.1.0/24
```

The broadcast address is:

```text
192.168.1.255
```

So:

```text
Network Address:   192.168.1.0
First Host:        192.168.1.1
Last Host:         192.168.1.254
Broadcast Address: 192.168.1.255
```

---

# 7. What Is Subnetting?

**Subnetting** means dividing one large network into smaller networks.

For example:

```text
192.168.1.0/24
```

Contains:

```text
256 total addresses
254 usable host addresses
```

We can divide it into two smaller networks:

```text
192.168.1.0/25
192.168.1.128/25
```

```text
Original Network
192.168.1.0/24

│
├── 192.168.1.0/25
│   Hosts: 126
│
└── 192.168.1.128/25
    Hosts: 126
```

---

# 8. Example: /24 to /26

Start with:

```text
192.168.1.0/24
```

We borrow 2 bits:

```text
/24 → /26
```

That creates:

```text
2² = 4 subnets
```

Each subnet contains:

```text
2^(32 - 26) = 64 addresses
```

The networks are:

```text
192.168.1.0/26
192.168.1.64/26
192.168.1.128/26
192.168.1.192/26
```

The first subnet:

```text
Network:   192.168.1.0
First IP:  192.168.1.1
Last IP:   192.168.1.62
Broadcast: 192.168.1.63
```

The second subnet:

```text
Network:   192.168.1.64
First IP:  192.168.1.65
Last IP:   192.168.1.126
Broadcast: 192.168.1.127
```

---

# 9. How to Calculate the Block Size

Example:

```text
/26
```

Subnet mask:

```text
255.255.255.192
```

The block size is:

```text
256 - 192 = 64
```

Therefore, the networks increase by `64`:

```text
0
64
128
192
```

Result:

```text
192.168.1.0/26
192.168.1.64/26
192.168.1.128/26
192.168.1.192/26
```

Another example:

```text
/27

Subnet Mask:

255.255.255.224
```

Block size:

```text
256 - 224 = 32
```

Networks:

```text
192.168.1.0/27
192.168.1.32/27
192.168.1.64/27
192.168.1.96/27
192.168.1.128/27
192.168.1.160/27
192.168.1.192/27
192.168.1.224/27
```

---

# 10. How Does a Device Know if the Destination Is Local?

Suppose your computer has:

```text
IP Address:

192.168.1.10/24
```

It wants to send data to:

```text
192.168.1.20
```

The computer checks the subnet.

```text
192.168.1.10/24
192.168.1.20/24
```

Both belong to:

```text
192.168.1.0/24
```

Therefore:

```text
Same Network
      │
      ▼
Send directly to destination
```

But if the destination is:

```text
8.8.8.8
```

It is not part of the local network.

Therefore:

```text
Different Network
      │
      ▼
Send packet to Default Gateway
```

---

# 11. Default Gateway

A **Default Gateway** is usually a router that connects your local network to other networks.

Example:

```text
Computer

IP Address:
192.168.1.10/24

Default Gateway:
192.168.1.1
```

Communication:

```text
192.168.1.10
     │
     │ Local Network
     ▼
192.168.1.1
Default Gateway
     │
     ▼
Router
     │
     ▼
Other Networks
     │
     ▼
Internet
```

The gateway does not necessarily know the final destination directly.

It uses a **routing table** to decide where the packet should go next.

---

# 12. Routing

**Routing** is the process of deciding where an IP packet should be forwarded.

A router contains a routing table.

Example:

```text
Destination Network       Next Hop

192.168.1.0/24           Directly Connected
10.0.0.0/8               192.168.1.2
172.16.0.0/16            192.168.1.3
0.0.0.0/0                Internet Gateway
```

The route:

```text
0.0.0.0/0
```

means:

```text
Default Route
```

In simple terms:

```text
"If I don't know a more specific route,
send the packet here."
```

---

# 13. Longest Prefix Match

Routers choose the **most specific route**.

Example:

```text
10.0.0.0/8     → Router A
10.1.0.0/16    → Router B
```

Destination:

```text
10.1.5.10
```

Both routes match.

But:

```text
10.1.0.0/16
```

is more specific than:

```text
10.0.0.0/8
```

Therefore the router chooses:

```text
Router B
```

This is called:

```text
Longest Prefix Match
```

---

# 14. Complete Example

Your computer:

```text
IP Address:
192.168.1.10/24

Default Gateway:
192.168.1.1
```

You want to access:

```text
8.8.8.8
```

The flow is:

```text
Application
    │
    ▼
Transport Layer
TCP / UDP
    │
    ▼
Network Layer
Destination IP: 8.8.8.8
    │
    ▼
Is 8.8.8.8 in my local subnet?
    │
    ├── Yes → Send directly
    │
    └── No
         │
         ▼
    Send to Default Gateway
    192.168.1.1
         │
         ▼
       Router
         │
         ▼
    Routing Table
         │
         ▼
      Next Router
         │
         ▼
      Internet
         │
         ▼
       8.8.8.8
```

---

# Important Concepts

```text
IP Address
=
Identifies a device on an IP network
```

```text
Subnet Mask
=
Determines the Network and Host portions
```

```text
CIDR
=
Number of bits used for the network

Example:

/24
```

```text
Subnetting
=
Dividing a larger network into smaller networks
```

```text
Network Address
=
Identifies the subnet
```

```text
Broadcast Address
=
Sends traffic to all devices in the subnet
```

```text
Default Gateway
=
The router used to reach other networks
```

```text
Routing
=
Choosing where an IP packet goes next
```

---

# Summary

The Network Layer is responsible for moving packets between networks.

```text
Source Application
        │
        ▼
TCP / UDP
        │
        ▼
IP Packet
        │
        ▼
Is the destination local?
        │
   ┌────┴────┐
   │         │
  Yes        No
   │         │
   ▼         ▼
Direct     Default
Delivery   Gateway
              │
              ▼
           Router
              │
              ▼
         Destination
```

The key idea is:

```text
IP Address tells us:

WHERE the destination is.

Subnet Mask tells us:

WHICH network the IP belongs to.

Router tells us:

WHERE to send the packet next.
```