# Headers and Content Types

## Definition

HTTP headers are key-value metadata attached to a request or response, covering content description (`Content-Type`, `Content-Length`), negotiation (`Accept`), caching (`Cache-Control`, `ETag`), and custom application data. `Content-Type` (and its negotiation counterpart, `Accept`) describes the media type of a body, telling the receiver how to parse it.

```
Content-Type: application/json
Accept: application/json
Cache-Control: no-cache
Authorization: Bearer eyJhbGciOi...
```

## Alternatives & Trade-offs

Headers keep metadata separate from the payload, letting infrastructure (proxies, caches, gateways) inspect and act on it without parsing the body. The trade-off is that headers are less structured than a body — there's no schema enforcement, and case-insensitive names plus historically inconsistent conventions (some use commas, some repeat the header) can make them easy to misuse compared to a well-typed body.

## How It Works

### Content negotiation

```
GET /orders/42 HTTP/1.1
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json
```

The client states what it can accept (`Accept`); the server responds with what it actually sent (`Content-Type`) — ideally matching, but the server may also respond `406 Not Acceptable` if it can't produce any of the requested formats.

### Common content types

```
application/json           — JSON payload
application/x-www-form-urlencoded — traditional HTML form data
multipart/form-data        — file uploads mixed with form fields
text/plain                 — plain text
application/problem+json   — structured error details (RFC 7807/9457)
```

### Caching headers

```
Cache-Control: max-age=3600, public
ETag: "33a64df551"

GET /orders/42
If-None-Match: "33a64df551"

HTTP/1.1 304 Not Modified   — the client's cached copy is still valid, no body sent
```

`ETag` lets a client ask "has this changed since I last saw it?" without re-downloading the full body if the answer is no.

### Custom headers for cross-cutting concerns

```
X-Correlation-Id: 8f14e45f-...
Idempotency-Key: 5f3c1e2a-...
```

Custom headers (often prefixed `X-` by convention, though that prefix is now discouraged by RFC 6648 in favor of clearly documented names) carry cross-cutting metadata — request tracing, idempotency — outside the body's business data.

## Application

Use `Content-Type`/`Accept` to support multiple representations of the same resource (JSON is the default for most APIs; XML or others where required). Use caching headers (`ETag`, `Cache-Control`) to avoid unnecessary data transfer for unchanged resources. Use correlation IDs to trace a single logical request across multiple services.

## Common Mistakes

- Setting `Content-Type` incorrectly (e.g., sending JSON with `Content-Type: text/plain`), causing clients or frameworks to fail to parse the body correctly.
- Ignoring the `Accept` header entirely and always returning the same format, even when a client explicitly requests something else.
- Treating headers as unlimited free-form storage for data that actually belongs in the body, making the API contract harder to discover and version.
- Forgetting that header names are case-insensitive — code that does an exact case-sensitive string match against a header name can silently fail against a technically valid request.

## Common Interview Questions

### Basic
- What is the difference between `Content-Type` and `Accept`?
- What does `application/json` mean as a `Content-Type` value?

### Intermediate
- How does `ETag`-based caching avoid unnecessary data transfer?
- What should a server do if it can't produce any format the client's `Accept` header requests?

### Advanced
- How would you design content negotiation for an API that needs to support both JSON and CSV export?
- What's the purpose of a correlation ID header, and how does it help in a distributed system?

### Follow-up Questions
- Is `X-` still the recommended prefix for custom headers?
- Can a single response include multiple `Content-Type`-like negotiations (e.g., language and format together)?

### Code Prediction
A client sends `GET /reports/5` with `Accept: application/xml`, but the server only supports JSON. What status code should a well-behaved server return, and why is silently returning JSON anyway a worse choice?

## Practical Tasks

- Implement `ETag`-based conditional requests for a read endpoint and verify a `304 Not Modified` response when nothing has changed.
- Design content negotiation for an endpoint that must support both JSON and CSV output.
- Add a correlation ID header to a request and trace how it would flow through the response and any downstream calls.

## Readiness Criteria

Explain content negotiation and caching headers precisely, correctly reason about `Content-Type` vs. `Accept`, and design a header-based mechanism for a cross-cutting concern like tracing or idempotency.

## References

### Other

- [MDN: HTTP headers](https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers)
- [RFC 9110: HTTP Semantics — Content Negotiation](https://www.rfc-editor.org/rfc/rfc9110#name-content-negotiation)
- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457)
