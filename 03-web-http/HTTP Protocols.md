# HTTP: What, Why, How, Benefits, and Its Versions

---

## What is HTTP?

**HTTP (HyperText Transfer Protocol)** is the application-layer protocol used to transfer data on the web. It defines a common language between:

- **Clients:** browsers, mobile apps, scripts, IoT devices.
- **Servers:** machines that host websites, APIs, files, and services.

The client sends a **request**, and the server sends back a **response**.

```
Client  ---- HTTP request  ---->  Server
        <--- HTTP response -----
```

Key characteristics:

- **Request/response model:** the client always starts the conversation.
- **Stateless:** each request is independent; the server does not remember earlier requests unless cookies or tokens are used.
- **Resource-based:** everything is identified by a URL.
- **Runs over TCP or QUIC:** default ports are `80` (HTTP) and `443` (HTTPS).

## Why Use HTTP?

Without a shared standard, every website and every browser would need custom code to talk to each other. HTTP solves this by giving everyone one agreed format:

- **Interoperability:** any client can talk to any server, whatever language or platform they use.
- **Universal access:** every browser, phone, and programming language has built-in HTTP support.
- **Simplicity:** messages are easy to read, debug, and test.
- **Foundation for the web and APIs:** websites, REST APIs, microservices, and mobile backends all rely on it.

## How is HTTP Used?

### 1. Browsing websites
Typing a URL makes the browser send a `GET` request and render the HTML, CSS, JS, and images it receives.

### 2. APIs and web services
Apps exchange data (usually JSON) with servers. For example:

```http
POST /api/orders HTTP/1.1
Host: shop.example.com
Content-Type: application/json
Authorization: Bearer <token>

{"item": "laptop", "qty": 1}
```

### 3. File transfer
Downloading and uploading files, images, and videos (including streaming).

### 4. Microservices and backend communication
Services inside a company call each other over HTTP.

### 5. IoT and automation
Devices send sensor data or fetch commands through HTTP endpoints and webhooks.

### Request cycle in short

1. DNS turns the domain into an IP address.
2. The client opens a connection (TCP, plus TLS for HTTPS).
3. The client sends a request: method, path, headers, optional body.
4. The server replies: status code, headers, optional body.
5. The connection is reused or closed.

### Main building blocks

| Part         | Examples                                              | Role |
|--------------|-------------------------------------------------------|------|
| Methods      | `GET`, `POST`, `PUT`, `PATCH`, `DELETE`               | What action to perform |
| Status codes | `200 OK`, `404 Not Found`, `500 Server Error`         | Result of the request |
| Headers      | `Content-Type`, `Authorization`, `Cache-Control`      | Extra information about the message |
| Body         | HTML, JSON, images                                    | The actual data |

## Benefits of HTTP

- **Universal and standardized:** one protocol understood everywhere.
- **Easy to learn and debug:** readable messages, plus tools like curl and browser DevTools.
- **Flexible:** can carry any type of content (text, JSON, images, video, binary files).
- **Scalable:** statelessness makes it easy to add servers behind a load balancer.
- **Cache-friendly:** built-in caching (`Cache-Control`, `ETag`) saves bandwidth and speeds up pages.
- **Extensible:** custom headers, new methods, and new versions can be added without breaking older clients.
- **Secure when combined with TLS:** HTTPS adds encryption, integrity, and server authentication.
- **Huge ecosystem:** CDNs, proxies, gateways, and frameworks all support it.

## Limitations of HTTP

- Plain HTTP is **not encrypted**, so use HTTPS for anything sensitive.
- **Stateless** by design, so sessions need cookies or tokens.
- The client must start every exchange, so real-time server-to-client updates need extras such as WebSockets or SSE.
- Headers can be large and repetitive (improved in HTTP/2 and HTTP/3).

---

## HTTP Versions

Each version changed **how messages are carried**, not **what they mean**. Methods, status codes, and headers behave the same, so application code usually works unchanged.

### HTTP/1.1

**Published:** 1997 (RFC 9112)

**How it works**
- Plain-text messages over **TCP**.
- **Persistent connections** (keep-alive), so one connection serves many requests.
- Required `Host` header, enabling many websites on one IP.
- **Chunked transfer encoding** for streaming responses.
- Improved caching and range requests.

```http
GET /index.html HTTP/1.1
Host: example.com
Connection: keep-alive
```

**Limitations**
- **Head-of-line (HOL) blocking:** responses must arrive in order, so one slow response blocks the rest.
- Browsers open about **6 parallel connections per host** as a workaround, which costs extra handshakes and memory.
- Headers (such as cookies) are sent in full on every request.
- Developers used workarounds such as domain sharding, sprites, and file bundling.

### HTTP/2

**Published:** 2015 (RFC 9113)

**How it works**
- **Binary framing:** messages are split into frames, faster and less error-prone to parse.
- **Multiplexing:** many requests and responses share **one TCP connection** simultaneously.
- **Header compression (HPACK):** repeated headers are compressed.
- **Stream prioritization:** clients can hint which resources matter most.
- **Server push:** the server could send resources before they were requested, but it is deprecated and removed from major browsers.
- Used with **TLS** in practice (negotiated via ALPN).

```
Single TCP connection
 ├─ Stream 1: GET /index.html
 ├─ Stream 3: GET /app.css
 └─ Stream 5: GET /app.js     (all in flight at once)
```

**Limitations**
- **TCP-level HOL blocking:** one lost packet stalls *all* streams until it is retransmitted.
- Needs both the TCP and TLS handshakes before data flows.
- Connections break when the client's network changes (for example, Wi-Fi to mobile data).

### HTTP/3

**Published:** 2022 (RFC 9114)

**How it works**
- Runs on **QUIC** (RFC 9000) over **UDP** instead of TCP.
- **Independent streams:** packet loss only affects the stream it belongs to.
- **TLS 1.3 built in:** faster setup, with **1-RTT** for new connections and **0-RTT** resumption for returning clients (only safe for idempotent requests).
- **Connection migration:** connections survive IP/network changes.
- **Header compression (QPACK)**, designed for out-of-order delivery.
- Always encrypted.

```
HTTP/3 stack:        HTTP/2 stack:
HTTP/3               HTTP/2
QUIC (+TLS 1.3)      TLS
UDP                  TCP
IP                   IP
```

**Limitations**
- Some networks and firewalls block or throttle UDP.
- Higher CPU use than kernel-optimized TCP in some setups.
- Tooling and infrastructure support are still maturing.
- Clients need a fallback to HTTP/2 or HTTP/1.1.

### Comparison

| Feature                    | HTTP/1.1       | HTTP/2               | HTTP/3                   |
|----------------------------|----------------|----------------------|--------------------------|
| Format                     | Text           | Binary               | Binary                   |
| Transport                  | TCP            | TCP                  | QUIC over UDP            |
| Encryption                 | Optional       | Practically required | Always (TLS 1.3)         |
| Multiplexing               | No             | Yes                  | Yes                      |
| HOL blocking (HTTP level)  | Yes            | No                   | No                       |
| HOL blocking (transport)   | Yes            | Yes                  | No                       |
| Header compression         | None           | HPACK                | QPACK                    |
| Connection setup           | TCP (+ TLS)    | TCP + TLS            | 1-RTT (0-RTT resumption) |
| Connection migration       | No             | No                   | Yes                      |
| Connections per origin     | ~6             | 1                    | 1                        |

**Which to use?** Serve HTTP/2 everywhere, enable HTTP/3 where your CDN or server supports it, and keep HTTP/1.1 as a fallback for older clients.

**How a client picks the version:** HTTP/1.1 vs HTTP/2 is negotiated during the TLS handshake (ALPN). HTTP/3 is advertised by the server with an `Alt-Svc` header, and the client then tries QUIC on UDP port 443.

**Check the version in use:**

```bash
curl -sI --http2 https://example.com -o /dev/null -w "%{http_version}\n"
curl -sI https://example.com | grep -i alt-svc
```

In browser DevTools, open the Network tab and enable the **Protocol** column (`http/1.1`, `h2`, `h3`).
