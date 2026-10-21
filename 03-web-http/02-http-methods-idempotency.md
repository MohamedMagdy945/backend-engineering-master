# HTTP Methods & Idempotency

## 1. What Are HTTP Methods?

HTTP methods define the action a client wants to perform on a resource through an HTTP request.

For example, a browser or frontend application can use HTTP methods to retrieve, create, update, or delete data on a server.

### Common HTTP Methods

| Method | Purpose | Example |
|---|---|---|
| `GET` | Retrieve a resource | Get a user's proIdempotencyfile |
| `POST` | Create a resource or trigger an operation | Create a new user |
| `PUT` | Replace a resource completely | Replace a user's profile |
| `PATCH` | Partially update a resource | Update a user's email |
| `DELETE` | Delete a resource | Delete a user |
| `HEAD` | Retrieve headers without the response body | Check resource metadata |
| `OPTIONS` | Discover communication options supported by a resource | Check allowed HTTP methods |

### Examples

**GET**
```http
GET /users/10
```
Retrieves the user with ID `10`.

**POST**
```http
POST /users
Content-Type: application/json

{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```
Requests the creation of a new user.

**PUT**
```http
PUT /users/10
Content-Type: application/json

{
  "name": "Ahmed Ali",
  "email": "ahmed@example.com"
}
```
Requests that the existing user representation be replaced with the supplied data.

**PATCH**
```http
PATCH /users/10
Content-Type: application/json

{
  "name": "Ahmed Ali"
}
```
Requests an update to only the specified field.

**DELETE**
```http
DELETE /users/10
```
Requests the deletion of the user with ID `10`.

> Note: The exact behavior of an endpoint depends on its API contract. HTTP defines the general semantics of each method, while the server implements the corresponding operation.

---

## 2. What Is Idempotency?

Idempotency is a property of an HTTP method where making the same request multiple times has the same intended effect on the server as making it once.

In simple terms, repeating an idempotent request should not repeatedly change the final intended state of the resource.

### Example

Consider this request:

```http
DELETE /users/10
```

- First request: The user is deleted.
- Second request: The user is already deleted.
- Third request: The user is still deleted.

The final server state is the same, even if the responses differ.

This makes `DELETE` an idempotent method.

### Idempotent vs. Non-Idempotent Methods

| Method | Idempotent? | Reason |
|---|---|---|
| `GET` | Yes | Repeating the request does not change the intended server state |
| `HEAD` | Yes | Like `GET`, but without the response body |
| `OPTIONS` | Yes | Repeating it has no additional intended effect |
| `PUT` | Yes | Replacing a resource with the same representation produces the same intended state |
| `DELETE` | Yes | Deleting a resource repeatedly has the same intended effect |
| `POST` | No, by default | Repeating a request may create multiple resources or trigger an operation multiple times |
| `PATCH` | Not necessarily | Its idempotency depends on the specific operation |

---

## 3. Idempotency Examples

### Example A: Idempotent PUT

```http
PUT /users/10
Content-Type: application/json

{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

Sending this request three times results in the same intended user representation, assuming the request replaces that representation and no other changes intervene.

```text
Request 1 → User updated
Request 2 → Same user representation
Request 3 → Same user representation
```

### Example B: Non-Idempotent POST

```http
POST /orders
Content-Type: application/json

{
  "productId": 25,
  "quantity": 1
}
```

Sending the same request three times may create three separate orders.

```text
Request 1 → Order #101 created
Request 2 → Order #102 created
Request 3 → Order #103 created
```

Each request can produce an additional side effect.

### Example C: PATCH Can Be Idempotent

```http
PATCH /users/10
Content-Type: application/json

{
  "name": "Ahmed Ali"
}
```

If the operation sets the user's name to `"Ahmed Ali"`, repeating it leaves the name unchanged after the first successful update.

Therefore, this particular `PATCH` operation is idempotent.

However, consider this operation:

```json
{
  "incrementBalanceBy": 100
}
```

If each request increases the balance by 100, repeating it changes the balance each time. This operation is not idempotent.

**Important:** `PATCH` is neither inherently idempotent nor inherently non-idempotent. Its behavior depends on the operation.

---

## 4. Why Is Idempotency Important?

Idempotency is especially important in distributed systems and backend APIs because requests can be repeated unintentionally.

Common causes include:

- Network timeouts.
- Temporary connection failures.
- Client-side retries.
- Load balancer or proxy retries.
- Users clicking a submit button multiple times.

### Example: Payment Processing

Imagine a user submits a payment request.

```http
POST /payments
```

The server processes the payment, but the response never reaches the client because of a network failure.

The client cannot determine whether the payment succeeded.

If it sends the request again, the user might be charged twice.

Idempotency helps prevent this problem.

### Using an Idempotency Key

A client can send a unique key with the request:

```http
POST /payments
Idempotency-Key: payment-abc-123
Content-Type: application/json

{
  "amount": 500,
  "currency": "EGP"
}
```

The server stores the key and the result of the operation.

If the client retries the same operation using the same key, the server can return the previously stored result instead of processing the payment again.

```text
First request
    ↓
Process payment
    ↓
Store result with idempotency key
    ↓
Client retries with the same key
    ↓
Return the stored result
```

The exact implementation depends on the API. A robust design also needs to handle concurrent requests, key expiration, and reuse of the same key with different request data.

**Important:** An `Idempotency-Key` header does not automatically make a request idempotent. The server must implement the required behavior.

---

## 5. Idempotency vs. Safe Methods

Idempotency and safety are two different HTTP properties.

### Safe Methods

A safe method is intended to retrieve information without requesting a change to the server's resource state.

Examples:
- `GET`
- `HEAD`
- `OPTIONS`

### Idempotent Methods

An idempotent method has the same intended server-side effect when the same request is repeated.

Examples:
- `GET`
- `HEAD`
- `OPTIONS`
- `PUT`
- `DELETE`

| Method | Safe? | Idempotent? |
|---|---|---|
| `GET` | Yes | Yes |
| `HEAD` | Yes | Yes |
| `OPTIONS` | Yes | Yes |
| `PUT` | No | Yes |
| `DELETE` | No | Yes |
| `POST` | No | No, by default |
| `PATCH` | No | Depends on the operation |

A `DELETE` request is not safe because it requests a change, but it is idempotent because repeating the deletion has the same intended effect.

---

## 6. Key Takeaways

- HTTP methods describe the intended action of a request.
- `GET` retrieves data, `POST` submits or creates, `PUT` replaces, `PATCH` partially updates, and `DELETE` removes.
- Idempotency means repeating a request has the same intended effect as performing it once.
- `PUT` and `DELETE` are idempotent by definition of their HTTP semantics.
- `POST` is not idempotent by default, but an API can implement idempotency using an idempotency key.
- `PATCH` may or may not be idempotent, depending on the operation.
- HTTP responses do not have to be identical for a method to be idempotent.
- Idempotency is important when implementing retries, payment processing, and reliable distributed systems.

### Reference

- RFC 9110 — HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
# HTTP Methods & Idempotency

## 1. What Are HTTP Methods?

HTTP methods define the action a client wants to perform on a resource through an HTTP request.

For example, a browser or frontend application can use HTTP methods to retrieve, create, update, or delete data on a server.

### Common HTTP Methods

| Method | Purpose | Example |
|---|---|---|
| `GET` | Retrieve a resource | Get a user's profile |
| `POST` | Create a resource or trigger an operation | Create a new user |
| `PUT` | Replace a resource completely | Replace a user's profile |
| `PATCH` | Partially update a resource | Update a user's email |
| `DELETE` | Delete a resource | Delete a user |
| `HEAD` | Retrieve headers without the response body | Check resource metadata |
| `OPTIONS` | Discover communication options supported by a resource | Check allowed HTTP methods |

### Examples

**GET**
```http
GET /users/10
```
Retrieves the user with ID `10`.

**POST**
```http
POST /users
Content-Type: application/json

{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```
Requests the creation of a new user.

**PUT**
```http
PUT /users/10
Content-Type: application/json

{
  "name": "Ahmed Ali",
  "email": "ahmed@example.com"
}
```
Requests that the existing user representation be replaced with the supplied data.

**PATCH**
```http
PATCH /users/10
Content-Type: application/json

{
  "name": "Ahmed Ali"
}
```
Requests an update to only the specified field.

**DELETE**
```http
DELETE /users/10
```
Requests the deletion of the user with ID `10`.

> Note: The exact behavior of an endpoint depends on its API contract. HTTP defines the general semantics of each method, while the server implements the corresponding operation.

---

## 2. What Is Idempotency?

Idempotency is a property of an HTTP method where making the same request multiple times has the same intended effect on the server as making it once.

In simple terms, repeating an idempotent request should not repeatedly change the final intended state of the resource.

### Example

Consider this request:

```http
DELETE /users/10
```

- First request: The user is deleted.
- Second request: The user is already deleted.
- Third request: The user is still deleted.

The final server state is the same, even if the responses differ.

This makes `DELETE` an idempotent method.

### Idempotent vs. Non-Idempotent Methods

| Method | Idempotent? | Reason |
|---|---|---|
| `GET` | Yes | Repeating the request does not change the intended server state |
| `HEAD` | Yes | Like `GET`, but without the response body |
| `OPTIONS` | Yes | Repeating it has no additional intended effect |
| `PUT` | Yes | Replacing a resource with the same representation produces the same intended state |
| `DELETE` | Yes | Deleting a resource repeatedly has the same intended effect |
| `POST` | No, by default | Repeating a request may create multiple resources or trigger an operation multiple times |
| `PATCH` | Not necessarily | Its idempotency depends on the specific operation |

---

## 3. Idempotency Examples

### Example A: Idempotent PUT

```http
PUT /users/10
Content-Type: application/json

{
  "name": "Ahmed",
  "email": "ahmed@example.com"
}
```

Sending this request three times results in the same intended user representation, assuming the request replaces that representation and no other changes intervene.

```text
Request 1 → User updated
Request 2 → Same user representation
Request 3 → Same user representation
```

### Example B: Non-Idempotent POST

```http
POST /orders
Content-Type: application/json

{
  "productId": 25,
  "quantity": 1
}
```

Sending the same request three times may create three separate orders.

```text
Request 1 → Order #101 created
Request 2 → Order #102 created
Request 3 → Order #103 created
```

Each request can produce an additional side effect.

### Example C: PATCH Can Be Idempotent

```http
PATCH /users/10
Content-Type: application/json

{
  "name": "Ahmed Ali"
}
```

If the operation sets the user's name to `"Ahmed Ali"`, repeating it leaves the name unchanged after the first successful update.

Therefore, this particular `PATCH` operation is idempotent.

However, consider this operation:

```json
{
  "incrementBalanceBy": 100
}
```

If each request increases the balance by 100, repeating it changes the balance each time. This operation is not idempotent.

**Important:** `PATCH` is neither inherently idempotent nor inherently non-idempotent. Its behavior depends on the operation.

---

## 4. Why Is Idempotency Important?

Idempotency is especially important in distributed systems and backend APIs because requests can be repeated unintentionally.

Common causes include:

- Network timeouts.
- Temporary connection failures.
- Client-side retries.
- Load balancer or proxy retries.
- Users clicking a submit button multiple times.

### Example: Payment Processing

Imagine a user submits a payment request.

```http
POST /payments
```

The server processes the payment, but the response never reaches the client because of a network failure.

The client cannot determine whether the payment succeeded.

If it sends the request again, the user might be charged twice.

Idempotency helps prevent this problem.

### Using an Idempotency Key

A client can send a unique key with the request:

```http
POST /payments
Idempotency-Key: payment-abc-123
Content-Type: application/json

{
  "amount": 500,
  "currency": "EGP"
}
```

The server stores the key and the result of the operation.

If the client retries the same operation using the same key, the server can return the previously stored result instead of processing the payment again.

```text
First request
    ↓
Process payment
    ↓
Store result with idempotency key
    ↓
Client retries with the same key
    ↓
Return the stored result
```

The exact implementation depends on the API. A robust design also needs to handle concurrent requests, key expiration, and reuse of the same key with different request data.

**Important:** An `Idempotency-Key` header does not automatically make a request idempotent. The server must implement the required behavior.

---

## 5. Idempotency vs. Safe Methods

Idempotency and safety are two different HTTP properties.

### Safe Methods

A safe method is intended to retrieve information without requesting a change to the server's resource state.

Examples:
- `GET`
- `HEAD`
- `OPTIONS`

### Idempotent Methods

An idempotent method has the same intended server-side effect when the same request is repeated.

Examples:
- `GET`
- `HEAD`
- `OPTIONS`
- `PUT`
- `DELETE`

| Method | Safe? | Idempotent? |
|---|---|---|
| `GET` | Yes | Yes |
| `HEAD` | Yes | Yes |
| `OPTIONS` | Yes | Yes |
| `PUT` | No | Yes |
| `DELETE` | No | Yes |
| `POST` | No | No, by default |
| `PATCH` | No | Depends on the operation |

A `DELETE` request is not safe because it requests a change, but it is idempotent because repeating the deletion has the same intended effect.

---

## 6. Key Takeaways

- HTTP methods describe the intended action of a request.
- `GET` retrieves data, `POST` submits or creates, `PUT` replaces, `PATCH` partially updates, and `DELETE` removes.
- Idempotency means repeating a request has the same intended effect as performing it once.
- `PUT` and `DELETE` are idempotent by definition of their HTTP semantics.
- `POST` is not idempotent by default, but an API can implement idempotency using an idempotency key.
- `PATCH` may or may not be idempotent, depending on the operation.
- HTTP responses do not have to be identical for a method to be idempotent.
- Idempotency is important when implementing retries, payment processing, and reliable distributed systems.

### Reference

- RFC 9110 — HTTP Semantics: https://www.rfc-editor.org/rfc/rfc9110
