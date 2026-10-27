# Cookies & Session Management

A practical guide to HTTP cookies, session management, authentication state, and security best practices.

---

## 1. Core Mechanics: How State Works in HTTP

HTTP is stateless, meaning each request is independent by default. Cookies and sessions allow applications to maintain authentication state across multiple requests.

### Session-Based Authentication Flow

```text
[ Browser / Client ]                [ Web Server ]
        |                                  |
        |--- POST /login ----------------->|
        |                                  | Validate credentials
        |                                  | Generate Session ID
        |                                  | Store session in Redis/DB
        |<-- Set-Cookie: sid=xyz ----------|
        |                                  |
        |--- GET /profile ---------------->|
        |    Cookie: sid=xyz               | Retrieve session
        |                                  | Identify user
        |<-- Profile Response -------------|
```

The browser stores the cookie and automatically sends it with matching requests. The server uses the Session ID to retrieve the associated session data.

---

## 2. Essential Cookie Directives

Cookies are created through the `Set-Cookie` HTTP response header.

```http
Set-Cookie: sid=abc123xyz; Path=/; Max-Age=86400; Secure; HttpOnly; SameSite=Lax
```

| Directive | Purpose |
|---|---|
| `HttpOnly` | Prevents client-side JavaScript from reading the cookie. |
| `Secure` | Sends the cookie over HTTPS only. |
| `SameSite` | Controls cookie transmission in cross-site contexts. |
| `Path` | Restricts the URL paths for which the cookie is sent. |
| `Domain` | Defines the hosts eligible to receive the cookie. |
| `Max-Age` | Specifies the cookie lifetime in seconds. |
| `Expires` | Specifies the cookie's expiration date. |

### SameSite Options

- **`Strict`** — Restricts cookies to same-site requests, including most navigations originating from other sites.
- **`Lax`** — Allows cookies on same-site requests and certain cross-site top-level navigations, typically using safe methods such as GET.
- **`None`** — Allows cross-site cookie transmission when other browser policies permit it. Requires `Secure`.

`SameSite=Lax` is a common starting point for authentication cookies. However, SameSite is not a complete replacement for CSRF protection.

### Cookie Lifetime

- If neither `Max-Age` nor `Expires` is specified, the cookie is generally a session cookie.
- `Max-Age` defines a relative lifetime and takes precedence over `Expires` when both are present.
- Cookie expiration and server-side session expiration are separate concerns.

---

## 3. Cookie Security Prefixes

Modern browsers enforce additional restrictions on cookies using reserved prefixes.

### `__Secure-`

Requires the cookie to be set from a secure origin over HTTPS and include the `Secure` attribute.

### `__Host-`

Provides stricter host-level restrictions:

- Requires HTTPS and `Secure`.
- Requires `Path=/`.
- Must not include a `Domain` attribute.

Example:

```http
Set-Cookie: __Host-sid=xyz123; Path=/; Secure; HttpOnly; SameSite=Lax
```

The `__Host-` prefix helps prevent a cookie from being shared across subdomains.

---

## 4. Session Architectures: Stateful vs. Stateless

### Stateful Authentication

The client stores an opaque Session ID. The actual session data is stored on the server, usually in a session store such as Redis or a database.

```text
Client
  |
  | Cookie: sid=xyz
  v
Server
  |
  | Lookup session
  v
Redis / Database
  |
  v
User Session Data
```

### Stateless Authentication

The client holds a signed token, such as a JWT. The server validates the token without necessarily looking up a session on every request.

```text
Client
  |
  | Authentication Token
  v
Server
  |
  | Validate signature, issuer,
  | audience, and expiration
  v
Authorized Request
```

A JWT may be stored in an HttpOnly cookie or sent using the `Authorization: Bearer` header. These are different transport choices, not different JWT formats.

### Comparison

| Aspect | Stateful Session | Stateless JWT |
|---|---|---|
| Client stores | Session ID | Signed token |
| Authentication state | Server-side store | Token claims and validation rules |
| Immediate revocation | Delete or invalidate the session | Requires an additional revocation strategy for already-issued tokens |
| Server storage | Shared session store when required | No per-request session lookup required by default |
| Multiple servers | Requires shared session storage or another coordination mechanism | Servers can independently validate tokens when configured with trusted keys and matching validation rules |
| Request size | Usually smaller identifier | Usually larger token |

Neither approach is universally better. The appropriate choice depends on the application's authentication requirements and deployment architecture.

---

## 5. Security Threats and Countermeasures

### Cross-Site Scripting (XSS)

Malicious JavaScript may attempt to steal credentials or perform actions as the user.

**Mitigations:**
- Set `HttpOnly` on authentication cookies.
- Use a restrictive Content Security Policy (CSP).
- Validate and safely encode untrusted output.
- Avoid exposing authentication credentials to JavaScript unnecessarily.

`HttpOnly` limits direct cookie access but does not prevent malicious scripts from making authenticated requests.

### Cross-Site Request Forgery (CSRF)

An attacker may attempt to make a victim's browser send an authenticated request to another site.

**Mitigations:**
- Use appropriate `SameSite` settings.
- Implement antiforgery tokens for state-changing requests where required.
- Validate request origins when appropriate.
- Configure CORS with explicit trusted origins.

CORS alone is not a complete CSRF defense.

### Session Fixation

An attacker may attempt to make a victim authenticate using a session identifier already known to the attacker.

**Mitigation:** Regenerate the session identifier after successful authentication and relevant privilege changes.

### Session Hijacking

An attacker who obtains a valid session identifier or authentication token may impersonate its owner.

**Mitigations:**
- Enforce HTTPS.
- Set `Secure` and `HttpOnly` on authentication cookies.
- Use appropriate expiration and revocation policies.
- Enable HTTP Strict Transport Security (HSTS) for HTTPS deployments.
- Never log session identifiers, cookies, or authentication tokens.

---

## 6. Session Lifecycle and Invalidation

A secure session-management strategy should define the following policies.

### Idle Timeout

Invalidate a session after a defined period of inactivity, such as 15–30 minutes when appropriate for the application's risk level.

### Absolute Timeout

Set a maximum session lifetime, regardless of user activity. For example, a system may require reauthentication after a defined workday.

Actual durations should follow the application's security requirements.

### Logout and Invalidation

A complete logout process should:

1. Invalidate the server-side session or authentication credential when supported by the chosen architecture.
2. Expire the browser cookie using the same cookie name, path, and domain scope as the original cookie.
3. Ensure subsequent protected requests are rejected when the relevant credential has been invalidated.

Example cookie-expiration header:

```http
Set-Cookie: __Host-sid=; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=0
```

Clearing a cookie in the browser does not automatically invalidate a server-side session or a previously issued stateless JWT.

---

## 7. Production Checklist

- [ ] Use HTTPS for production authentication.
- [ ] Configure `Secure` and `HttpOnly` on authentication cookies.
- [ ] Select an appropriate `SameSite` policy.
- [ ] Define idle and absolute expiration policies where required.
- [ ] Regenerate session identifiers after authentication.
- [ ] Implement CSRF protection when applicable.
- [ ] Use shared session storage when stateful sessions span multiple servers.
- [ ] Define token revocation and refresh policies when using JWTs.
- [ ] Invalidate sessions and clear cookies during logout.
- [ ] Never expose credentials or session identifiers in application logs.

---

## 8. Key Takeaway

Cookies are a browser mechanism for storing and sending values. A Session ID identifies server-side session state, while a JWT carries signed claims that the server can validate.

Choose the authentication architecture according to the application's needs:

- **Session-Based Authentication:** When centralized session management and straightforward revocation are important.
- **JWT Authentication:** When bearer-token authentication and independent token validation across services or clients are useful.
# Cookies & Session Management

A practical guide to HTTP cookies, session management, authentication state, and security best practices.

---

## 1. Core Mechanics: How State Works in HTTP

HTTP is stateless, meaning each request is independent by default. Cookies and sessions allow applications to maintain authentication state across multiple requests.

### Session-Based Authentication Flow

```text
[ Browser / Client ]                [ Web Server ]
        |                                  |
        |--- POST /login ----------------->|
        |                                  | Validate credentials
        |                                  | Generate Session ID
        |                                  | Store session in Redis/DB
        |<-- Set-Cookie: sid=xyz ----------|
        |                                  |
        |--- GET /profile ---------------->|
        |    Cookie: sid=xyz               | Retrieve session
        |                                  | Identify user
        |<-- Profile Response -------------|
```

The browser stores the cookie and automatically sends it with matching requests. The server uses the Session ID to retrieve the associated session data.

---

## 2. Essential Cookie Directives

Cookies are created through the `Set-Cookie` HTTP response header.

```http
Set-Cookie: sid=abc123xyz; Path=/; Max-Age=86400; Secure; HttpOnly; SameSite=Lax
```

| Directive | Purpose |
|---|---|
| `HttpOnly` | Prevents client-side JavaScript from reading the cookie. |
| `Secure` | Sends the cookie over HTTPS only. |
| `SameSite` | Controls cookie transmission in cross-site contexts. |
| `Path` | Restricts the URL paths for which the cookie is sent. |
| `Domain` | Defines the hosts eligible to receive the cookie. |
| `Max-Age` | Specifies the cookie lifetime in seconds. |
| `Expires` | Specifies the cookie's expiration date. |

### SameSite Options

- **`Strict`** — Restricts cookies to same-site requests, including most navigations originating from other sites.
- **`Lax`** — Allows cookies on same-site requests and certain cross-site top-level navigations, typically using safe methods such as GET.
- **`None`** — Allows cross-site cookie transmission when other browser policies permit it. Requires `Secure`.

`SameSite=Lax` is a common starting point for authentication cookies. However, SameSite is not a complete replacement for CSRF protection.

### Cookie Lifetime

- If neither `Max-Age` nor `Expires` is specified, the cookie is generally a session cookie.
- `Max-Age` defines a relative lifetime and takes precedence over `Expires` when both are present.
- Cookie expiration and server-side session expiration are separate concerns.

---

## 3. Cookie Security Prefixes

Modern browsers enforce additional restrictions on cookies using reserved prefixes.

### `__Secure-`

Requires the cookie to be set from a secure origin over HTTPS and include the `Secure` attribute.

### `__Host-`

Provides stricter host-level restrictions:

- Requires HTTPS and `Secure`.
- Requires `Path=/`.
- Must not include a `Domain` attribute.

Example:

```http
Set-Cookie: __Host-sid=xyz123; Path=/; Secure; HttpOnly; SameSite=Lax
```

The `__Host-` prefix helps prevent a cookie from being shared across subdomains.

---

## 4. Session Architectures: Stateful vs. Stateless

### Stateful Authentication

The client stores an opaque Session ID. The actual session data is stored on the server, usually in a session store such as Redis or a database.

```text
Client
  |
  | Cookie: sid=xyz
  v
Server
  |
  | Lookup session
  v
Redis / Database
  |
  v
User Session Data
```

### Stateless Authentication

The client holds a signed token, such as a JWT. The server validates the token without necessarily looking up a session on every request.

```text
Client
  |
  | Authentication Token
  v
Server
  |
  | Validate signature, issuer,
  | audience, and expiration
  v
Authorized Request
```

A JWT may be stored in an HttpOnly cookie or sent using the `Authorization: Bearer` header. These are different transport choices, not different JWT formats.

### Comparison

| Aspect | Stateful Session | Stateless JWT |
|---|---|---|
| Client stores | Session ID | Signed token |
| Authentication state | Server-side store | Token claims and validation rules |
| Immediate revocation | Delete or invalidate the session | Requires an additional revocation strategy for already-issued tokens |
| Server storage | Shared session store when required | No per-request session lookup required by default |
| Multiple servers | Requires shared session storage or another coordination mechanism | Servers can independently validate tokens when configured with trusted keys and matching validation rules |
| Request size | Usually smaller identifier | Usually larger token |

Neither approach is universally better. The appropriate choice depends on the application's authentication requirements and deployment architecture.

---

## 5. Security Threats and Countermeasures

### Cross-Site Scripting (XSS)

Malicious JavaScript may attempt to steal credentials or perform actions as the user.

**Mitigations:**
- Set `HttpOnly` on authentication cookies.
- Use a restrictive Content Security Policy (CSP).
- Validate and safely encode untrusted output.
- Avoid exposing authentication credentials to JavaScript unnecessarily.

`HttpOnly` limits direct cookie access but does not prevent malicious scripts from making authenticated requests.

### Cross-Site Request Forgery (CSRF)

An attacker may attempt to make a victim's browser send an authenticated request to another site.

**Mitigations:**
- Use appropriate `SameSite` settings.
- Implement antiforgery tokens for state-changing requests where required.
- Validate request origins when appropriate.
- Configure CORS with explicit trusted origins.

CORS alone is not a complete CSRF defense.

### Session Fixation

An attacker may attempt to make a victim authenticate using a session identifier already known to the attacker.

**Mitigation:** Regenerate the session identifier after successful authentication and relevant privilege changes.

### Session Hijacking

An attacker who obtains a valid session identifier or authentication token may impersonate its owner.

**Mitigations:**
- Enforce HTTPS.
- Set `Secure` and `HttpOnly` on authentication cookies.
- Use appropriate expiration and revocation policies.
- Enable HTTP Strict Transport Security (HSTS) for HTTPS deployments.
- Never log session identifiers, cookies, or authentication tokens.

---

## 6. Session Lifecycle and Invalidation

A secure session-management strategy should define the following policies.

### Idle Timeout

Invalidate a session after a defined period of inactivity, such as 15–30 minutes when appropriate for the application's risk level.

### Absolute Timeout

Set a maximum session lifetime, regardless of user activity. For example, a system may require reauthentication after a defined workday.

Actual durations should follow the application's security requirements.

### Logout and Invalidation

A complete logout process should:

1. Invalidate the server-side session or authentication credential when supported by the chosen architecture.
2. Expire the browser cookie using the same cookie name, path, and domain scope as the original cookie.
3. Ensure subsequent protected requests are rejected when the relevant credential has been invalidated.

Example cookie-expiration header:

```http
Set-Cookie: __Host-sid=; Path=/; Secure; HttpOnly; SameSite=Lax; Max-Age=0
```

Clearing a cookie in the browser does not automatically invalidate a server-side session or a previously issued stateless JWT.

---

## 7. Production Checklist

- [ ] Use HTTPS for production authentication.
- [ ] Configure `Secure` and `HttpOnly` on authentication cookies.
- [ ] Select an appropriate `SameSite` policy.
- [ ] Define idle and absolute expiration policies where required.
- [ ] Regenerate session identifiers after authentication.
- [ ] Implement CSRF protection when applicable.
- [ ] Use shared session storage when stateful sessions span multiple servers.
- [ ] Define token revocation and refresh policies when using JWTs.
- [ ] Invalidate sessions and clear cookies during logout.
- [ ] Never expose credentials or session identifiers in application logs.

---

## 8. Key Takeaway

Cookies are a browser mechanism for storing and sending values. A Session ID identifies server-side session state, while a JWT carries signed claims that the server can validate.

Choose the authentication architecture according to the application's needs:

- **Session-Based Authentication:** When centralized session management and straightforward revocation are important.
- **JWT Authentication:** When bearer-token authentication and independent token validation across services or clients are useful.
- **Cookie Authentication:** When a web application needs browser-based authentication without necessarily maintaining a separate application session.

Keep authentication, session storage, cookie configuration, and authorization responsibilities clearly defined.