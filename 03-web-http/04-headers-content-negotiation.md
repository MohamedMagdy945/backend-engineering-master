# Headers & Content Negotiation

## 1. What Are HTTP Headers?

HTTP Headers are additional pieces of information sent between a client and a server as part of an HTTP request or response.

They help both sides understand details about the message, such as the data format, language, compression method, and authentication information.

### Example

```http
GET /api/users HTTP/1.1
Host: example.com
Accept: application/json
Authorization: Bearer token123
```

- `Host`: Specifies the target host.
- `Accept`: Specifies the response media types the client prefers.
- `Authorization`: Provides credentials for accessing a protected resource.

## 2. What Is Content Negotiation?

Content Negotiation is the process of selecting the most appropriate representation of a resource based on the client's preferences and the server's capabilities.

The same resource can be available in different formats, languages, or compression methods.

For example:

- JSON or XML.
- English or French.
- Compressed or uncompressed content.

The client communicates its preferences through HTTP request headers, and the server selects an appropriate representation when possible.

## 3. Important Headers

### Accept

The `Accept` header specifies the media types that the client prefers to receive in the response.

```http
Accept: application/json
```

Meaning: The client prefers to receive JSON data.

Another example:

```http
Accept: application/json, application/xml
```

Meaning: The client accepts JSON or XML representations, with preferences determined by the header's quality values and applicable negotiation rules.

### Content-Type

The `Content-Type` header identifies the media type of the message body.

Example in a request:

```http
POST /api/users HTTP/1.1
Content-Type: application/json

{
  "name": "Ahmed",
  "age": 26
}
```

Meaning: The client is sending JSON data.

Example in a response:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 101,
  "name": "Ahmed"
}
```

Meaning: The server is returning JSON data.

**The key difference:**

- `Accept` = What response format does the client prefer?
- `Content-Type` = What format does the message body actually contain?

### Accept-Language

The `Accept-Language` header specifies the languages the client prefers.

```http
Accept-Language: en, ar;q=0.8
```

Meaning:

- English has the highest preference.
- Arabic is also acceptable, with a lower preference.

The `q` value represents a preference weight between 0 and 1. A higher value indicates a stronger preference.

### Accept-Encoding

The `Accept-Encoding` header specifies the content-coding methods the client can handle.

```http
Accept-Encoding: gzip, br
```

Meaning: The client supports compressed responses using `gzip` or `br` (Brotli).

### Content-Encoding

The `Content-Encoding` header identifies the content coding applied to the message body.

```http
Content-Encoding: gzip
```

Meaning: The message body is compressed using `gzip`.

## 4. How Content Negotiation Works

Suppose an API provides user information in JSON and XML formats.

The client sends this request:

```http
GET /api/users/1 HTTP/1.1
Host: example.com
Accept: application/json
```

The server checks the requested media type and determines whether it can provide JSON.

If JSON is available, the server might respond:

```http
HTTP/1.1 200 OK
Content-Type: application/json

{
  "id": 1,
  "name": "Ahmed"
}
```

The process works as follows:

1. The client sends an HTTP request with an `Accept` header.
2. The server checks the client's preferences and its supported representations.
3. The server selects an appropriate representation.
4. The server identifies the response body's media type using `Content-Type`.
5. The client reads and processes the response.

## 5. What Happens When the Requested Format Is Unavailable?

Suppose the client requests XML:

```http
Accept: application/xml
```

However, the server can only provide JSON.

Depending on its implementation and negotiation rules, the server might respond with:

```http
HTTP/1.1 406 Not Acceptable
```

`406 Not Acceptable` indicates that the server cannot provide a response representation acceptable under the request's negotiation preferences.

Another important status code is `415 Unsupported Media Type`.

It commonly occurs when the server refuses the media type or content coding of the request body.

For example, an API might reject an XML request body if it only accepts JSON.

## 6. Accept vs. Content-Type vs. Content-Encoding

| Header             | Purpose                                            |
| ------------------ | -------------------------------------------------- |
| `Accept`           | Specifies preferred response media types.          |
| `Content-Type`     | Identifies the media type of the message body.     |
| `Accept-Encoding`  | Specifies supported content-coding methods.        |
| `Content-Encoding` | Identifies the content coding applied to the body. |
| `Accept-Language`  | Specifies preferred response languages.            |

## 7. Why Are These Headers Important?

- **Interoperability:** Help different systems exchange data in compatible formats.
- **Flexibility:** Allow resources to be represented in different formats or languages.
- **Efficiency:** Enable compression to reduce the amount of data transferred.
- **Clear communication:** Help clients and servers understand how message bodies should be interpreted.
- **User experience:** Allow servers to provide content in a language or representation suited to the client.

## 8. Important Notes

- The `Accept` header expresses preferences; it does not guarantee that the server will provide the requested format.
- `Content-Type` and `Accept` have different purposes.
- `Content-Encoding` describes content coding, such as compression. It is different from `Content-Type`.
- JSON is commonly used in REST APIs, but HTTP supports many other media types.
- Content negotiation can also involve other headers and server-specific rules.

## Summary

**HTTP Headers** provide additional information about HTTP requests and responses.

**Content Negotiation** allows a client and server to select an appropriate representation of a resource based on client preferences and server capabilities.

Remember:

- `Accept` → What response format do I prefer?
- `Content-Type` → What type of data does the body contain?
- `Accept-Encoding` → Which content-coding methods can I handle?
- `Content-Encoding` → Which content coding is applied to the body?
- `Accept-Language` → Which language do I prefer?

### Further Reading

- [MDN: Content Negotiation](https://developer.mozilla.org/en-US/docs/Web/HTTP/Guides/Content_negotiation)
- [MDN: HTTP Headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Reference/Headers)
