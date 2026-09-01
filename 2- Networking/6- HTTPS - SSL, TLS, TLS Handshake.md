# HTTPS, SSL/TLS, and the TLS Handshake

When you open a secure website:

```text
https://example.com
```

Your browser and the server need to solve three important problems:

1. **Privacy** — Other people should not be able to read the data.
2. **Authentication** — The browser must verify that it is talking to the real server.
3. **Integrity** — The data should not be modified while traveling through the network.

HTTPS solves these problems using **TLS**.

---

# 1. What is HTTPS?

HTTPS means:

```text
HTTPS
=
HTTP + TLS
```

HTTP is responsible for communication between the client and server.

For example:

```text
GET /products HTTP/1.1
Host: example.com
```

TLS adds security around that communication.

So instead of sending plain HTTP data:

```text
Client
   │
   │ HTTP Request
   │ GET /users
   ▼
Server
```

The communication becomes:

```text
Client
   │
   │ Encrypted HTTP Request
   ▼
TLS
   │
   ▼
TCP
   │
   ▼
Internet
   │
   ▼
TCP
   │
   ▼
TLS
   │
   ▼
Server
```

The application still uses HTTP.

TLS protects the HTTP communication.

---

# 2. SSL vs TLS

You will often hear:

```text
SSL
TLS
```

SSL is the older technology.

TLS is the modern protocol.

Today, HTTPS uses TLS, although people still commonly say:

```text
SSL Certificate
SSL/TLS
SSL Connection
```

A more accurate description is:

```text
HTTPS
└── TLS
```

---

# 3. The Big Picture

The image can be divided into four main stages:

```text
┌─────────────────────────────────────────────┐
│                                             │
│  1. TCP Handshake                           │
│                                             │
│  2. TLS Handshake + Certificate Check       │
│                                             │
│  3. Key Exchange                            │
│                                             │
│  4. Encrypted Data Transmission             │
│                                             │
└─────────────────────────────────────────────┘
```

The complete simplified flow looks like this:

```text
CLIENT                                  SERVER
  │                                        │
  │────────── TCP SYN ────────────────────►│
  │◄─────── TCP SYN + ACK ─────────────────│
  │────────── TCP ACK ────────────────────►│
  │                                        │
  │========= TCP CONNECTION READY =========│
  │                                        │
  │────────── Client Hello ───────────────►│
  │◄───────── Server Hello ─────────────────│
  │◄───────── Certificate ──────────────────│
  │                                        │
  │========= TLS HANDSHAKE ================│
  │                                        │
  │────────── Key Exchange ───────────────►│
  │                                        │
  │========= SECURE SESSION ===============│
  │                                        │
  │────── Encrypted HTTP Data ────────────►│
  │◄───── Encrypted HTTP Data ─────────────│
  │                                        │
```

---

# 4. Step One — TCP Handshake

Before HTTPS can start, the client usually needs a TCP connection.

The TCP connection is established using the three-way handshake.

```text
Client                              Server
   │                                   │
   │──────────── SYN ─────────────────►│
   │                                   │
   │◄──────── SYN + ACK ───────────────│
   │                                   │
   │──────────── ACK ─────────────────►│
   │                                   │
   │======= Connection Established =====│
```

### SYN

The client says:

```text
"I want to establish a TCP connection."
```

### SYN + ACK

The server replies:

```text
"I received your request and I am ready."
```

### ACK

The client confirms:

```text
"Connection established."
```

Now TCP is ready.

But the communication is **not encrypted yet**.

At this point:

```text
TCP Connection
=
Reliable connection

NOT
=
Secure connection
```

Security comes from TLS.

---

# 5. Step Two — TLS Handshake

After the TCP connection is established, the TLS handshake begins.

The first message is usually:

```text
Client Hello
```

Simplified:

```text
Client                                  Server
  │                                        │
  │──────────── Client Hello ─────────────►│
  │                                        │
```

The client sends information about what it supports.

For example:

```text
Client Hello
├── TLS versions supported
├── Cryptographic algorithms supported
├── Random data
└── Other TLS parameters
```

The server responds:

```text
Client                                  Server
  │                                        │
  │──────────── Client Hello ─────────────►│
  │                                        │
  │◄──────────── Server Hello ─────────────│
  │                                        │
```

The server selects the TLS settings that will be used.

---

# 6. The Certificate

The server then proves its identity by sending a **digital certificate**.

```text
Client                                  Server
  │                                        │
  │◄──────────── Certificate ──────────────│
  │                                        │
```

The certificate contains information such as:

```text
Certificate
├── Domain Name
├── Server Public Key
├── Certificate Authority information
├── Validity Period
└── Digital Signature
```

For example:

```text
example.com
```

The browser checks:

```text
Is this certificate valid?
        │
        ├── Is it expired?
        │
        ├── Does it belong to example.com?
        │
        ├── Is it trusted?
        │
        └── Has it been modified?
```

If the verification succeeds:

```text
✓ Server identity verified
```

If something is wrong:

```text
⚠ Your connection is not private
```

---

# 7. Public Key and Private Key

The image shows two important keys:

```text
Public Key
Private Key
```

These are used in **asymmetric cryptography**.

The keys are mathematically related.

```text
Server

Public Key
    │
    │ Can be shared
    ▼
Everyone can know it


Private Key
    │
    │ Must remain secret
    ▼
Only the server should have it
```

The important idea is:

```text
Public Key
=
Public information

Private Key
=
Secret information
```

The server can send its public key inside its certificate.

But it never sends its private key.

---

# 8. Why Do We Need Asymmetric Encryption?

At the beginning, the client and server do not yet share a secret.

So they need a secure way to establish trust and create shared encryption keys.

This is where public/private key cryptography is used.

Simplified:

```text
Client                           Server

                                 Private Key
                                     🔒

        Public Key
            🔑
◄────────────── Certificate ─────────────
```

The client can obtain the server's public key.

The private key remains only on the server.

---

# 9. Key Exchange

The next important step is establishing shared secret material.

The image shows this as:

```text
Key Exchange
```

The goal is to create a shared:

```text
Session Key
```

Simplified:

```text
Client                                  Server
  │                                        │
  │                                        │
  │         Key Exchange Process           │
  │───────────────────────────────────────►│
  │◄───────────────────────────────────────│
  │                                        │
  │                                        │
  │========= Shared Session Keys ==========│
```

Both sides now have cryptographic keys that can protect the connection.

---

# 10. What Is a Session Key?

A **session key** is a temporary key used to encrypt the actual communication between the client and server.

```text
Client                            Server

Session Key 🔑                    Session Key 🔑
       │                                  │
       └──────── Same Secure Keys ────────┘
```

After the TLS handshake:

```text
HTTP Request
        │
        ▼
Encrypt using session keys
        │
        ▼
Send through TCP
        │
        ▼
Server receives encrypted data
        │
        ▼
Decrypt using session keys
        │
        ▼
HTTP Request
```

---

# 11. Why Not Use Asymmetric Encryption for Everything?

Asymmetric encryption is useful for authentication and establishing secure keys.

However, it is more computationally expensive than symmetric encryption.

Therefore, HTTPS uses a combination of both.

```text
                    TLS

        ┌────────────┴────────────┐
        │                         │
        ▼                         ▼

Asymmetric                    Symmetric
Cryptography                  Cryptography

Authentication                Data Encryption
Key Exchange                  HTTP Traffic
```

The general idea is:

```text
Asymmetric Cryptography
        │
        ▼
Establish Trust + Keys
        │
        ▼
Symmetric Cryptography
        │
        ▼
Encrypt Actual Data
```

---

# 12. Symmetric Encryption

With symmetric encryption, both sides use shared secret keys.

```text
Client                              Server

Session Key 🔑                      Session Key 🔑

Plain Data                          Plain Data
    │                                    ▲
    ▼                                    │
Encrypt                              Decrypt
    │                                    │
    ▼                                    │
Encrypted Data ──────────────────────────┘
```

Example:

```text
GET /api/users
```

Before encryption:

```text
GET /api/users
Authorization: Bearer token
```

After TLS encryption, someone observing the network sees encrypted data instead of the original HTTP request.

Conceptually:

```text
GET /api/users

        │
        ▼

TLS Encryption

        │
        ▼

X82$K!@#Q...Encrypted Data...
```

The server can decrypt it because both sides have the required session keys.

---

# 13. HTTPS Layering

HTTPS is not a completely separate application protocol.

It is essentially HTTP protected by TLS.

A simplified protocol stack:

```text
┌───────────────────────────────┐
│         HTTP                  │
│   GET /api/users              │
├───────────────────────────────┤
│         TLS                   │
│   Encryption + Authentication │
├───────────────────────────────┤
│         TCP                   │
│   Reliable Transport          │
├───────────────────────────────┤
│         IP                    │
│   Routing Between Networks    │
├───────────────────────────────┤
│      Ethernet / Wi-Fi         │
│   Local Network Transmission  │
└───────────────────────────────┘
```

So when you access:

```text
https://example.com
```

The communication can be viewed as:

```text
HTTP
  │
  ▼
TLS
  │
  ▼
TCP
  │
  ▼
IP
  │
  ▼
Network
```

---

# 14. Complete HTTPS Flow

Let's combine everything.

Suppose you open:

```text
https://example.com
```

## Step 1 — DNS

The browser needs the server IP address.

```text
example.com
     │
     ▼
DNS
     │
     ▼
93.xxx.xxx.xxx
```

---

## Step 2 — TCP Connection

The browser establishes a TCP connection.

```text
Client                              Server

   SYN ─────────────────────────────►

       ◄──────────────────── SYN + ACK

   ACK ─────────────────────────────►
```

Now:

```text
TCP Connection Established
```

---

## Step 3 — TLS Handshake

The client starts TLS.

```text
Client                              Server

Client Hello ───────────────────────►

             ◄────────────── Server Hello

             ◄────────────── Certificate
```

The browser verifies the certificate.

```text
Certificate
      │
      ▼
Is it trusted?
      │
      ▼
Is it valid for this domain?
      │
      ▼
Is it expired?
      │
      ▼
✓ Trusted Server
```

---

## Step 4 — Key Exchange

The client and server establish shared session keys.

```text
Client                              Server

        Key Exchange
             │
             ▼
      Shared Secrets / Keys
```

The handshake completes.

```text
Client                              Server

Finished ───────────────────────────►

         ◄────────────────── Finished
```

Now the secure connection is ready.

---

## Step 5 — Encrypted HTTP Communication

The browser sends the HTTP request.

```text
Client

GET /products HTTP/1.1
Host: example.com

        │
        ▼

TLS Encryption

        │
        ▼

Encrypted Data
        │
        │
        ▼

===============================
              TCP
===============================

        │
        ▼

Internet
```

The server receives the data:

```text
Encrypted Data
        │
        ▼
TLS Decryption
        │
        ▼

GET /products HTTP/1.1
```

The server processes the request and sends an encrypted response.

---

# 15. The Full Picture

```text
┌──────────┐                                  ┌──────────┐
│  CLIENT  │                                  │  SERVER  │
└────┬─────┘                                  └────┬─────┘
     │                                             │
     │  1. TCP HANDSHAKE                           │
     │────────────────────────────────────────────►│
     │◄───────────────────────────────────────────►│
     │                                             │
     │════════ TCP CONNECTION ESTABLISHED ════════│
     │                                             │
     │  2. TLS HANDSHAKE                           │
     │                                             │
     │────────── Client Hello ────────────────────►│
     │                                             │
     │◄───────── Server Hello ─────────────────────│
     │◄───────── Certificate ──────────────────────│
     │                                             │
     │  Certificate Verification                   │
     │                                             │
     │  3. KEY EXCHANGE                            │
     │◄───────────────────────────────────────────►│
     │                                             │
     │════════ SECURE SESSION ESTABLISHED ════════│
     │                                             │
     │  4. ENCRYPTED DATA                          │
     │                                             │
     │────── Encrypted HTTP Request ─────────────►│
     │                                             │
     │◄───── Encrypted HTTP Response ─────────────│
     │                                             │
┌────┴─────┐                                  ┌────┴─────┐
│  CLIENT  │                                  │  SERVER  │
└──────────┘                                  └──────────┘
```

---

# 16. Important Correction About the Diagram

The image represents the **general idea** of HTTPS correctly, but modern TLS is slightly more complex.

In modern **TLS 1.3**, the process is optimized compared with older TLS versions.

Also, modern TLS commonly uses key exchange mechanisms such as:

```text
ECDHE
```

This allows the client and server to derive shared session keys without simply sending a session key encrypted with the server's public key.

The main concept is still:

```text
1. Establish TCP connection
        │
        ▼
2. Start TLS handshake
        │
        ▼
3. Verify server identity
        │
        ▼
4. Establish shared encryption keys
        │
        ▼
5. Use symmetric encryption
        │
        ▼
6. Exchange encrypted HTTP data
```

---

# 17. HTTP vs HTTPS

## HTTP

```text
Client
   │
   │ Plain HTTP Data
   ▼
Internet
   │
   ▼
Server
```

Data may potentially be readable or modified by an attacker in certain positions on the network.

---

## HTTPS

```text
Client
   │
   │ HTTP Data
   ▼
TLS Encrypts Data
   │
   ▼
Encrypted Data
   │
   ▼
Internet
   │
   ▼
TLS Decrypts Data
   │
   ▼
Server
```

HTTPS provides:

```text
🔒 Confidentiality
   Only the intended parties can read the protected data.

✓ Authentication
   The client can verify the server's identity.

🛡 Integrity
   Changes to protected data can be detected.
```

---

# Summary

The relationship between the technologies is:

```text
HTTPS
  │
  ├── HTTP
  │     └── Application communication
  │
  └── TLS
        ├── Server Authentication
        ├── Certificate Verification
        ├── Key Exchange
        └── Encryption

              │

              ▼

             TCP
        Reliable Transport

              │

              ▼

              IP
        Network Routing
```

The complete connection can be remembered like this:

```text
1. DNS
   "Where is the server?"

        ↓

2. TCP
   "Let's establish a reliable connection."

        ↓

3. TLS
   "Prove who you are and establish secure keys."

        ↓

4. HTTPS
   "Now exchange encrypted HTTP requests and responses."
```

## The Most Important Idea

```text
TCP
=
Reliable connection

TLS
=
Security

HTTP
=
Application communication

HTTPS
=
HTTP protected by TLS
```