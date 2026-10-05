#  How Web Browsers Work

##  Overview

A web browser is the application that allows users to access and interact with web applications.

Examples include:

- Google Chrome
    
- Microsoft Edge
    
- Mozilla Firefox
    
- Safari
    

From a backend developer's perspective, the browser is not just a program that displays websites.

The browser is an important part of the web architecture.

It can:

```text
Send HTTP Requests
Receive HTTP Responses
Parse HTML
Load CSS
Execute JavaScript
Manage Cookies
Store Data
Cache Resources
Handle Security
Render Web Pages
```

A simple model:

```text
User
  │
  ▼
Browser
  │
  │ 
HTTP Request
  ▼
Web Server / API
  │
  │ HTTP Response
  ▼
Browser
  │
  ▼
Render Page
```

---

# 1. What Is a Browser?

A browser is a **client application**.

It communicates with servers using web protocols such as:

```text
HTTP
HTTPS
WebSocket
```

For example, when you open:

```text
https://example.com
```

The browser starts a process.

```text
Browser
   │
   ▼
Find Server
   │
   ▼
Connect
   │
   ▼
Send Request
   │
   ▼
Receive Response
   │
   ▼
Process Response
   │
   ▼
Display Result
```

---

# 2. Browser Architecture

A modern browser contains multiple important components.

```text
Browser
│
├── User Interface
│
├── Browser Engine
│
├── Rendering Engine
│
├── JavaScript Engine
│
├── Networking
│
├── Storage
│
└── Security
```

Each component has a different responsibility.

---

# 3. Browser User Interface

This is the part the user interacts with.

Examples:

```text
Address Bar
Back Button
Forward Button
Tabs
Bookmarks
Settings
```

For example:

```text
┌──────────────────────────────────────┐
│ ← → ⟳   https://example.com          │
├──────────────────────────────────────┤
│                                      │
│          Website Content             │
│                                      │
└──────────────────────────────────────┘
```

The browser UI itself is separate from the website.

---

# 4. What Happens When You Enter a URL?

Suppose you enter:

```text
https://api.example.com/users
```

The browser must understand different parts of the URL.

```text
https://api.example.com/users
  │          │              │
  │          │              └── Path
  │          │
  │          └── Host
  │
  └── Protocol
```

The browser then starts processing the request.

```text
Enter URL
    │
    ▼
Parse URL
    │
    ▼
DNS Lookup
    │
    ▼
Find Server IP
    │
    ▼
Create Connection
    │
    ▼
TLS Handshake
    │
    ▼
Send HTTP Request
    │
    ▼
Receive HTTP Response
```

---

# 5. DNS Lookup

The browser needs to find the IP address of:

```text
api.example.com
```

It may first check caches.

```text
Browser Cache
      │
      ▼
Operating System Cache
      │
      ▼
DNS Resolver
      │
      ▼
DNS Server
```

Eventually:

```text
api.example.com
        │
        ▼
203.xxx.xxx.xxx
```

Now the browser knows where the server is.

---

# 6. Creating the Connection

After getting the IP address, the browser connects to the server.

For HTTP:

```text
Browser
   │
   │ TCP Connection
   ▼
Server
```

For HTTPS:

```text
Browser
   │
   │ TCP / QUIC Connection
   ▼
Server
```

The exact transport can depend on the HTTP version.

For example:

```text
HTTP/1.1 → TCP

HTTP/2   → TCP

HTTP/3   → QUIC / UDP
```

---

# 7. HTTPS and TLS

If the URL uses:

```text
https://
```

The browser must establish a secure connection.

```text
Browser
   │
   │ TLS Handshake
   ▼
Server
```

The server provides a certificate.

The browser checks things such as:

```text
Is the certificate valid?

Is the certificate trusted?

Does the certificate match the domain?

Has the certificate expired?
```

If everything is valid:

```text
Secure Connection Established
```

Then HTTP communication happens through the encrypted connection.

---

# 8. Sending an HTTP Request

The browser sends an HTTP request.

Example:

```http
GET /users HTTP/1.1
Host: api.example.com
Accept: application/json
```

The request may also contain:

```text
Headers
Cookies
Authorization
Request Body
Browser Metadata
```

For example:

```http
GET /profile HTTP/1.1
Host: example.com
Authorization: Bearer TOKEN
Cookie: sessionId=123
```

The request travels through:

```text
Browser
   │
   ▼
Internet
   │
   ▼
Reverse Proxy
   │
   ▼
Application
```

---

# 9. Receiving the Response

The server processes the request.

Example:

```text
Browser
   │
   │ GET /users
   ▼
Server
   │
   │ Process Request
   ▼
Database
   │
   ▼
Server
   │
   │ HTTP Response
   ▼
Browser
```

The response might be:

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

The browser checks the response.

```text
Status Code
Headers
Content Type
Response Body
```

Then it decides how to process the content.

---

# 10. Different Types of Responses

The server can return many types of content.

```text
text/html

text/css

application/javascript

application/json

image/png

image/jpeg

application/pdf
```

Examples:

```text
HTML → Render Page

CSS → Apply Styles

JavaScript → Execute Code

JSON → Used by JavaScript

Image → Display Image

PDF → Open / Download
```

---

# 11. HTML Parsing

Suppose the server returns:

```html
<html>
<head>
    <title>My Website</title>
</head>

<body>
    <h1>Hello</h1>
    <img src="/image.png">
</body>
</html>
```

The browser parses the HTML.

```text
HTML
  │
  ▼
HTML Parser
  │
  ▼
DOM
```

DOM means:

```text
Document Object Model
```

The browser converts the HTML into a tree.

```text
html
│
├── head
│   └── title
│
└── body
    │
    ├── h1
    │
    └── img
```

---

# 12. Loading Additional Resources

While parsing HTML, the browser may discover:

```html
<link href="style.css">

<script src="app.js"></script>

<img src="logo.png">
```

The browser then sends more requests.

```text
Browser
   │
   ├── GET /
   │
   ├── GET /style.css
   │
   ├── GET /app.js
   │
   └── GET /logo.png
```

One page can require many HTTP requests.

---

# 13. CSS and the CSSOM

The browser downloads CSS.

```text
CSS
 │
 ▼
CSS Parser
 │
 ▼
CSSOM
```

CSSOM means:

```text
CSS Object Model
```

Example:

```css
h1 {
    font-size: 32px;
}
```

The browser understands how elements should look.

---

# 14. JavaScript Engine

Browsers contain JavaScript engines.

Examples:

```text
Chrome / Edge → V8

Firefox → SpiderMonkey

Safari → JavaScriptCore
```

The JavaScript engine executes JavaScript.

Example:

```javascript
fetch("/api/users")
```

This can create another HTTP request.

```text
JavaScript
     │
     ▼
Browser APIs
     │
     ▼
Network Request
     │
     ▼
Backend API
```

This is important for backend developers.

Your API is often called by JavaScript running inside the browser.

---

# 15. Rendering the Page

The browser combines different structures.

```text
DOM
 │
 ├── HTML Structure
 │
CSSOM
 │
 ├── Styling Information
 │
 ▼
Render Tree
```

Then:

```text
Render Tree
    │
    ▼
Layout
    │
    ▼
Paint
    │
    ▼
Display
```

The simplified rendering process is:

```text
HTML
  │
  ▼
DOM

CSS
  │
  ▼
CSSOM

DOM + CSSOM
     │
     ▼
Render Tree
     │
     ▼
Layout
     │
     ▼
Paint
     │
     ▼
Screen
```

---

# 16. Layout

The browser calculates where elements should appear.

Example:

```text
┌──────────────────────┐
│ Header               │
├──────────────────────┤
│                      │
│ Main Content         │
│                      │
├──────────────────────┤
│ Footer               │
└──────────────────────┘
```

The browser calculates:

```text
Width
Height
Position
Spacing
```

This process is called:

```text
Layout
```

---

# 17. Paint and Compositing

After layout, the browser paints the elements.

```text
Text
Backgrounds
Borders
Images
Shadows
```

Then the browser combines layers.

```text
Layer 1
Layer 2
Layer 3
   │
   ▼
Compositing
   │
   ▼
Final Screen
```

---

# 18. JavaScript and the Page

JavaScript can modify the page after it has loaded.

Example:

```javascript
document.querySelector("h1").textContent = "Hello";
```

The flow:

```text
JavaScript
    │
    ▼
Modify DOM
    │
    ▼
Browser Updates Rendering
    │
    ▼
Screen Changes
```

JavaScript can also:

```text
Call APIs

Handle User Events

Modify HTML

Modify CSS

Store Data

Open WebSocket Connections
```

---

# 19. Browser Storage

Browsers can store data locally.

Common storage mechanisms:

```text
Cookies

Local Storage

Session Storage

IndexedDB

Cache Storage
```

---

## Cookies

Cookies are often sent automatically with requests to the relevant domain.

```text
Browser
   │
   │ Request
   │ Cookie: sessionId=123
   ▼
Server
```

They are commonly used for:

```text
Sessions
Authentication
Preferences
```

---

## Local Storage

Stores data in the browser.

Example:

```javascript
localStorage.setItem("theme", "dark");
```

Example use:

```text
Theme

Language

User Preferences
```

---

## Session Storage

Similar to Local Storage but scoped to a browsing session/tab context.

```text
Open Tab
   │
   ▼
Session Storage

Close Relevant Browsing Context
   │
   ▼
Session Data Removed
```

---

## IndexedDB

Used for more complex client-side storage.

It can store larger and structured data.

```text
Web Application
      │
      ▼
IndexedDB
      │
      ▼
Browser Storage
```

---

# 20. Browser Cache

The browser can cache resources.

Example:

```text
First Request

Browser
   │
   ▼
GET /logo.png
   │
   ▼
Server
   │
   ▼
Image
```

Later:

```text
Browser
   │
   ▼
Cache
   │
   ▼ 
Use Existing Resource
```

Caching can reduce:

```text
Network Requests

Page Load Time

Server Load
```

HTTP headers control much of this behavior.

Examples:

```http
Cache-Control

ETag

Last-Modified
```

---

# 21. Browser Security

Browsers protect users using multiple security mechanisms.

Important concepts:

```text
Same-Origin Policy

CORS

Content Security Policy

HTTPS

Cookie Security
```

---

# 22. Same-Origin Policy

An origin is generally based on:

```text
Scheme
Host
Port
```

Example:

```text
https://example.com
```

is different from:

```text
http://example.com
```

And different from:

```text
https://api.example.com
```

The browser restricts how pages from one origin can access resources from another origin.

---

# 23. CORS

Suppose your frontend runs on:

```text
https://app.example.com
```

And your API runs on:

```text
https://api.example.com
```

The browser sees these as different origins.

```text
Browser
   │
   │ Request
   ▼
API
```

The API must allow the browser to access it through appropriate CORS headers.

Conceptually:

```text
API Response

Access-Control-Allow-Origin
```

Important:

> CORS is primarily a browser security mechanism.

Your server can receive requests from many types of clients, such as:

```text
Browser

Mobile Application

Postman

curl

Backend Service
```

CORS rules are enforced by browsers, not by Postman or server-to-server clients in the same way.

---

# 24. Authentication in Browsers

A browser can authenticate using different approaches.

For example:

```text
Cookie-Based Authentication
```

```text
Browser
   │
   │ Cookie
   ▼
Server
```

Or:

```text
Token-Based Authentication
```

```text
Browser
   │
   │ Authorization Header
   │ Bearer TOKEN
   ▼
API
```

Example:

```http
Authorization: Bearer eyJ...
```

Security details depend on where and how credentials are stored and transmitted.

---

# 25. Browser Developer Tools

Modern browsers provide Developer Tools.

Useful tabs include:

```text
Elements

Console

Network

Application

Performance

Memory
```

For backend development, the most important one is often:

```text
Network
```

You can inspect:

```text
Request URL

HTTP Method

Request Headers

Request Body

Response Status

Response Headers

Response Body

Timing
```

Example:

```text
GET /api/users

Status: 200 OK

Time: 120ms
```

If your API fails:

```text
POST /api/users

Status: 500 Internal Server Error
```

The browser's Network tab helps you understand what actually happened between the frontend and backend.

---

# 26. Complete Browser Request Flow

Let's put everything together.

User enters:

```text
https://app.example.com
```

The complete simplified flow:

```text
1. User enters URL
        │
        ▼
2. Browser parses URL
        │
        ▼
3. DNS resolves domain
        │
        ▼
4. Browser connects to server
        │
        ▼
5. TLS establishes HTTPS security
        │
        ▼
6. Browser sends HTTP request
        │
        ▼
7. Server sends HTML
        │
        ▼
8. Browser parses HTML
        │
        ├── Load CSS
        ├── Load JavaScript
        ├── Load Images
        └── Load Fonts
        │
        ▼
9. Build DOM and CSSOM
        │
        ▼
10. Create Render Tree
        │
        ▼
11. Layout
        │
        ▼
12. Paint
        │
        ▼
13. Display Page
        │
        ▼
14. JavaScript runs
        │
        ▼
15. JavaScript calls Backend API
```

---

# 27. The Most Important Concept for a Backend Developer

Think of the browser as a powerful HTTP client.

```text
Browser
   │
   │ HTTP Request
   ▼
Your ASP.NET Core API
   │
   ▼
Process Request
   │
   ▼
Database
   │
   ▼
HTTP Response
   │
   ▼
Browser
```

But the browser also has additional responsibilities:

```text
Rendering

JavaScript Execution

Storage

Caching

Security

Cookie Management

Connection Management
```

Understanding the browser helps you understand why problems such as these happen:

```text
Why is CORS blocking my request?

Why was my cookie not sent?

Why is the API response cached?

Why is the request sending an OPTIONS request?

Why does HTTPS fail?

Why is JavaScript unable to access the response?

Why does the page make multiple API requests?
```

---
