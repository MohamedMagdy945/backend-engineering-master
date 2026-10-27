# HTTP Status Codes & Semantics

## 1. What Are HTTP Status Codes?

HTTP status codes are three-digit numbers that indicate the result of an HTTP request.

Example:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "message": "Success"
}
```

- `200`: The status code.
- `OK`: The reason phrase.
- `Content-Type`: Specifies the response body's media type.

## 2. Why Are Status Codes Important?

Status codes help developers:

- Identify successful and failed requests.
- Handle errors consistently.
- Debug API behavior.
- Distinguish client problems from server failures.
- Build predictable frontend and backend interactions.

## 3. Categories of HTTP Status Codes

| Range | Category | Meaning |
|---|---|---|
| `100–199` | Informational | The request process continues. |
| `200–299` | Success | The request was successfully handled. |
| `300–399` | Redirection | Further action may be required. |
| `400–499` | Client Error | The request cannot be fulfilled because of a client-related issue. |
| `500–599` | Server Error | The server failed to fulfill the request. |

## 4. Common HTTP Status Codes

### 4.1 Informational Responses (1xx)

| Code | Name | Description |
|---|---|---|
| `100` | Continue | The client can continue sending the request. |
| `101` | Switching Protocols | The server agrees to switch protocols. |

### 4.2 Successful Responses (2xx)

| Code | Name | Description |
|---|---|---|
| `200` | OK | The request succeeded. |
| `201` | Created | One or more resources were created. |
| `202` | Accepted | The request was accepted for processing, which may not be complete yet. |
| `204` | No Content | The request succeeded, and there is no response content. |
| `206` | Partial Content | The server returns part of a resource. |

Example of `201 Created`:

```http
HTTP/1.1 201 Created
Location: /users/15
```

The `Location` header identifies the newly created resource.

### 4.3 Redirection Responses (3xx)

| Code | Name | Description |
|---|---|---|
| `301` | Moved Permanently | The resource has a permanent new location. |
| `302` | Found | The resource is temporarily available at another location. |
| `303` | See Other | The client should retrieve another resource, typically using `GET`. |
| `304` | Not Modified | A cached representation can be reused. |
| `307` | Temporary Redirect | Redirects the request while preserving its method. |
| `308` | Permanent Redirect | Permanently redirects the request while preserving its method. |

**Important:** `301` and `302` can result in a `POST` changing to `GET` in common client behavior. `307` and `308` preserve the original method.

### 4.4 Client Error Responses (4xx)

| Code | Name | Description |
|---|---|---|
| `400` | Bad Request | The request is invalid or malformed. |
| `401` | Unauthorized | Valid authentication credentials are missing or unavailable. |
| `403` | Forbidden | The server refuses to authorize the operation. |
| `404` | Not Found | The requested resource cannot be found. |
| `405` | Method Not Allowed | The method is not supported for the target resource. |
| `408` | Request Timeout | The server did not receive the complete request in time. |
| `409` | Conflict | The request conflicts with the current resource state. |
| `410` | Gone | The resource is no longer available and that condition is likely permanent. |
| `413` | Content Too Large | The request content exceeds the server's limit. |
| `415` | Unsupported Media Type | The request content format is unsupported. |
| `416` | Range Not Satisfiable | The requested range cannot be provided. |
| `422` | Unprocessable Content | The content is understood but cannot be processed as requested. |
| `429` | Too Many Requests | The client has exceeded a rate limit. |

#### Example: 401 vs. 403

- `401`: The client has not provided valid authentication credentials.
- `403`: The server refuses to allow the requested operation.

#### Example: 409 Conflict

A user attempts to register an email address that must be unique, but the address already exists.

The API may return `409 Conflict`.

#### Example: 422 Unprocessable Content

A request contains valid JSON, but its values fail the API's validation rules.

```json
{
  "email": "invalid-email"
}
```

The API may return `422 Unprocessable Content`.

### 4.5 Server Error Responses (5xx)

| Code | Name | Description |
|---|---|---|
| `500` | Internal Server Error | An unexpected server failure occurred. |
| `501` | Not Implemented | The server does not support the required functionality. |
| `502` | Bad Gateway | A gateway received an invalid response from an upstream server. |
| `503` | Service Unavailable | The server cannot handle the request temporarily. |
| `504` | Gateway Timeout | A gateway did not receive a timely upstream response. |

Example:

```http
HTTP/1.1 500 Internal Server Error
Content-Type: application/json

{
  "message": "An unexpected error occurred"
}
```

Production APIs should not expose stack traces, credentials, or other sensitive implementation details.

## 5. What Is HTTP Semantics?

HTTP semantics define how requests and responses should be interpreted according to the protocol.

They establish the meaning of methods, status codes, headers, and representation metadata.

For example:

- `201 Created` communicates successful resource creation.
- `Retry-After` indicates when a client should retry in applicable situations.
- `Location` identifies a resource location or redirect destination, depending on the response.
- `Allow` lists the methods supported by a resource.

These standardized meanings allow different HTTP implementations to work consistently.

## 6. Status Codes vs. Response Bodies

The status code describes the HTTP outcome, while the response body can provide additional application-specific information.

Example:

```http
HTTP/1.1 422 Unprocessable Content
Content-Type: application/json

{
  "message": "Validation failed",
  "errors": {
    "email": "The email address is invalid"
  }
}
```

Here, `422` indicates the type of failure, while `errors` explains the specific problem.

An API can also use a standard error format, such as `application/problem+json`, defined in RFC 9457.

## 7. Conditional Requests and Caching

Conditional requests let clients validate representations or perform operations only when certain conditions are met.

### ETag

An `ETag` identifies a version of a resource representation.

```http
HTTP/1.1 200 OK
ETag: "v1"
```

A client can later send:

```http
GET /products/10
If-None-Match: "v1"
```

If the representation has not changed, the server can return `304 Not Modified`, allowing the client to reuse its cached data.

### If-Match

`If-Match` allows an operation to depend on the current resource version.

```http
PUT /products/10
If-Match: "v1"
```

If the condition fails, the server can return `412 Precondition Failed`. This helps prevent updates based on outdated representations.

## 8. Content Negotiation

Content negotiation allows clients to express preferences for response representations.

| Header | Purpose |
|---|---|
| `Accept` | Specifies acceptable response media types. |
| `Accept-Language` | Specifies preferred response languages. |
| `Accept-Encoding` | Specifies acceptable content encodings. |
| `Vary` | Identifies request headers that influenced the response and may affect caching. |

Example:

```http
GET /articles
Accept: application/json
Accept-Language: en
```

If the server cannot provide any acceptable representation, it may return `406 Not Acceptable`.

If the request body uses an unsupported media type, the server may return `415 Unsupported Media Type`.

## 9. Rate Limiting and Retry Behavior

Rate limiting restricts how frequently a client can make requests.

Example:

```http
HTTP/1.1 429 Too Many Requests
Retry-After: 30
```

The client should wait before retrying.

**Backend best practices:**

1. Respect `Retry-After` when applicable.
2. Use exponential backoff with jitter for eligible retries.
3. Avoid retrying indefinitely.
4. Do not assume that a timeout means the server did not perform the operation.
5. Use idempotency mechanisms when retrying operations that could cause duplicate side effects.

## 10. Best Practices for Backend Developers

1. Choose status codes that accurately describe the outcome.
2. Apply consistent conventions across endpoints.
3. Keep error responses useful and machine-readable.
4. Never expose sensitive server information.
5. Document possible status codes for each endpoint.
6. Use conditional requests when concurrent updates are a concern.
7. Handle retries carefully, particularly for state-changing operations.
8. Test both successful responses and error scenarios.

## 11. Key Takeaways

- HTTP status codes communicate standardized outcomes.
- Each status-code class represents a different type of response.
- HTTP semantics define how protocol elements should be interpreted.
- Conditional requests support caching and concurrent update protection.
- Content negotiation controls acceptable representations.
- Rate limiting and correct retry behavior help protect API reliability.
- Consistent error responses make APIs easier to maintain and integrate.

## Official References

- [HTTP Semantics — RFC 9110](https://www.rfc-editor.org/rfc/rfc9110)
- [HTTP/1.1 — RFC 9112](https://www.rfc-editor.org/rfc/rfc9112)
- [Problem Details for HTTP APIs — RFC 9457](https://www.rfc-editor.org/rfc/rfc9457)
- [HTTP Status Code Registry](https://www.iana.org/assignments/http-status-codes/http-status-codes.xhtml)