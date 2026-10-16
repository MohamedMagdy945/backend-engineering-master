## 1. What Is HTTP?

**HTTP (Hypertext Transfer Protocol)** is an application-layer protocol that defines how clients and servers communicate over a network.

It allows a client, such as a web browser or mobile application, to request resources from a server and receive responses.

HTTP can transfer different types of data, including:

- HTML pages
    
- Images and videos
    
- CSS and JavaScript files
    
- JSON data used by APIs
    
- Documents and other files
    

HTTP follows a **client-server architecture** and uses a request-response model.

## 2. How Does HTTP Work?

When you visit a website, HTTP enables your browser and the web server to exchange information.

### Step 1: Enter a URL

You enter a website address in your browser.

```
https://example.com/products
```

The URL identifies the resource you want to access.

### Step 2: Find the Server

The browser uses DNS to find the IP address associated with the domain name.

DNS is a separate system that helps the browser locate the server.

### Step 3: Establish a Connection

The browser establishes or reuses a connection to communicate with the server.

For HTTPS, TLS provides encryption and helps authenticate the server.

### Step 4: Send an HTTP Request

The browser sends a request describing the resource or action it wants.

Example:

```
GET /products HTTP/1.1
Host: example.com
Accept: text/html
```

- `GET`: Requests a resource.
    
- `/products`: Identifies the requested path.
    
- `Host`: Identifies the target host.
    
- `Accept`: Indicates the response format the client prefers.
    

### Step 5: The Server Processes the Request

The server receives the request and determines how to handle it.

Depending on the application, it might:

- Read a file.
    
- Query a database.
    
- Check authentication and permissions.
    
- Execute application logic.
    
- Generate a response.
    

### Step 6: Send an HTTP Response

The server returns a response containing a status code, headers, and sometimes a response body.

```
HTTP/1.1 200 OK
Content-Type: text/html

<h1>Our Products</h1>
```

`200 OK` means the request succeeded.

### Step 7: The Browser Processes the Response

The browser reads the response and displays the page. It may send additional HTTP requests for images, stylesheets, JavaScript, and other resources.

### Complete Request-Response Cycle

```
Browser                     Web Server
   |                             |
   |------ HTTP Request -------->|
   |                             |
   |                     Process Request
   |                             |
   |<----- HTTP Response --------|
   |                             |
   |      Display the Page       |
```

## 3. Why Do We Use HTTP?

HTTP solves the problem of how different applications communicate using a common set of rules.

### 1. Web Browsing

HTTP allows browsers to request web pages and download the resources needed to display them.

### 2. Client-Server Communication

It provides a standard way for clients and servers to exchange requests and responses, even when they run on different operating systems or use different programming languages.

### 3. Building APIs

Applications use HTTP to communicate with backend services.

For example, a mobile application can request user information from a server and receive it as JSON.

```
GET /api/users/42 HTTP/1.1
Host: api.example.com
Accept: application/json
```

Possible response:

```
{
  "id": 42,
  "name": "Alex"
}
```

### 4. Sending and Receiving Data

HTTP supports many content types, including text, JSON, images, and files. Requests can also carry data, such as information submitted through a form.

### 5. Standardization and Flexibility

HTTP defines common methods, status codes, headers, and message semantics. This makes communication more predictable and easier to implement across different systems.

### 6. Caching and Performance

HTTP supports caching, which allows suitable responses to be reused instead of repeatedly downloading the same resources.

This can reduce loading times, network traffic, and server workload.

## 4. HTTP Request and Response

An HTTP message can contain several important components.

### Request

- **Method:** The intended action, such as `GET` or `POST`.
    
- **Path:** The requested resource.
    
- **Headers:** Additional request information.
    
- **Body:** Optional data sent to the server.
    

### Response

- **Status code:** Indicates the result of the request.
    
- **Headers:** Describe the response.
    
- **Body:** Optional returned content.
    

## 5. Important HTTP Methods

|Method|Purpose|
|---|---|
|`GET`|Retrieve a resource.|
|`POST`|Submit data or request processing.|
|`PUT`|Create or replace a resource.|
|`PATCH`|Partially modify a resource.|
|`DELETE`|Request deletion of a resource.|

## 6. Common Status Codes

|Code|Meaning|
|---|---|
|`200 OK`|Request succeeded.|
|`201 Created`|A resource was created.|
|`301 Moved Permanently`|Permanent redirect.|
|`400 Bad Request`|Invalid request.|
|`401 Unauthorized`|Authentication is missing or unsuccessful.|
|`403 Forbidden`|Access is refused.|
|`404 Not Found`|Resource not found.|
|`500 Internal Server Error`|Server encountered an unexpected error.|

## 7. HTTP vs HTTPS

**HTTP** defines the communication protocol. **HTTPS** is HTTP protected by TLS.

- HTTP alone does not encrypt data in transit.
    
- HTTPS protects data in transit and helps authenticate the server.
    
- HTTP commonly uses port `80`.
    
- HTTPS commonly uses port `443`.
    

HTTPS is the preferred choice for modern websites and APIs.

## 8. Important Concepts

- **Statelessness:** HTTP does not automatically remember previous requests. Applications can use cookies and sessions to maintain state.
    
- **Headers:** Carry metadata about requests and responses.
    
- **Caching:** Allows suitable responses to be reused.
    
- **HTTP versions:** HTTP/1.1, HTTP/2, and HTTP/3 differ in message transport and connection behavior.
    
- **Status codes:** Tell the client whether a request succeeded or encountered a problem.
    

## Summary

HTTP is an application-layer protocol that defines how clients and servers exchange data.

Its basic workflow is:

1. The client identifies the server.
    
2. A connection is established or reused.
    
3. The client sends an HTTP request.
    
4. The server processes the request.
    
5. The server returns an HTTP response.
    
6. The client processes the response.
    

HTTP is essential for web browsing, APIs, data exchange, and communication between frontend applications and backend services.

**Reference:** [MDN — Overview of HTTP](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Overview)