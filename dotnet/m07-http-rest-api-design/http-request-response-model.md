# The HTTP Request/Response Model

## Definition

HTTP is a stateless, text-based (though often binary-framed in HTTP/2+) request/response protocol: a client sends a request (method, URL, headers, optional body) to a server, and the server replies with a single response (status code, headers, optional body). Each request is independent — the server retains no memory of previous requests unless the application explicitly builds state on top (sessions, tokens).

```
GET /orders/42 HTTP/1.1
Host: api.example.com
Accept: application/json

HTTP/1.1 200 OK
Content-Type: application/json
Content-Length: 57

{"id": 42, "customerId": 7, "total": 149.99}
```

## Alternatives & Trade-offs

HTTP's statelessness makes servers easy to scale horizontally (any server can handle any request, since none holds client-specific state) at the cost of needing an explicit mechanism — cookies, tokens, a session store — for anything that needs to remember something between requests. Stateful protocols (raw TCP sessions, WebSockets) avoid re-sending context on every message but require sticky connections and more complex server-side lifecycle management.

## How It Works

### Anatomy of a request

```
POST /orders HTTP/1.1          <- request line: method, path, HTTP version
Host: api.example.com          <- headers
Content-Type: application/json
Content-Length: 42

{"customerId": 7, "items": []} <- body (optional)
```

### Anatomy of a response

```
HTTP/1.1 201 Created           <- status line: version, status code, reason phrase
Location: /orders/43           <- headers
Content-Type: application/json

{"id": 43, "customerId": 7}    <- body (optional)
```

### Statelessness in practice

```csharp
using var client = new HttpClient();
var response1 = await client.GetAsync("https://api.example.com/orders/42");
var response2 = await client.GetAsync("https://api.example.com/orders/42");
// Nothing about response1 influences how the server handles response2 —
// each request carries everything the server needs (including auth headers, if required)
```

Any apparent "memory" between requests (a logged-in user, a shopping cart) is application state carried explicitly in a header, cookie, or token on each subsequent request — not something HTTP itself provides.

### Connections vs. requests (HTTP/1.1 keep-alive)

A single TCP connection can carry multiple sequential requests (`Connection: keep-alive`), which avoids the cost of a new TCP/TLS handshake per request — but each request/response pair on that connection is still logically independent and stateless at the HTTP level.

## Application

Understanding this model explains why REST APIs are naturally stateless (a core REST constraint — see `rest-principles-and-trade-offs.md`), why authentication must be re-asserted on every request (a bearer token in a header, not a "logged in" flag on the server), and why load balancers can freely route requests to any backend instance without needing "sticky sessions" unless the application specifically introduces server-side session state.

## Common Mistakes

- Assuming the server "remembers" a previous request without an explicit mechanism (cookie, token, session ID) carrying that context forward.
- Conflating a TCP connection (or an HTTP/2 stream) with a stateful session — keep-alive reuses the underlying connection for efficiency, but each request/response exchange is still independently processed.
- Forgetting that a request/response pair is atomic — there's no built-in way for a server to "push" a second response for one request without additional protocols (WebSockets, Server-Sent Events, HTTP/2 Server Push).
- Assuming headers are guaranteed to arrive in a specific order or case — header names are case-insensitive and clients/servers may reorder them.

## Common Interview Questions

### Basic
- What does it mean for HTTP to be stateless?
- What are the main parts of an HTTP request and an HTTP response?

### Intermediate
- If HTTP is stateless, how does a web application "remember" that a user is logged in?
- What's the difference between a TCP connection and an HTTP request/response exchange?

### Advanced
- How does statelessness influence horizontal scalability and load-balancer design?
- What are the trade-offs of introducing server-side session state versus keeping all state client-side (e.g., in a token)?

### Follow-up Questions
- Does HTTP/2 multiplexing change the request/response model itself, or just how requests are transported?
- Is a cookie part of HTTP's core statelessness model, or a mechanism built on top of it?

### Code Prediction
Given two sequential `HttpClient.GetAsync` calls to the same endpoint with no cookies or auth headers set, does the server have any way to know they came from the "same" client, beyond the IP address it observes at the network level?

## Practical Tasks

- Use a tool like `curl -v` or Fiddler to inspect the raw request and response for a simple HTTP call, identifying each part described above.
- Explain, in your own words, why a REST API described as "stateless" can still support "logged in" behavior.
- Diagram how a load balancer can route two requests from the same user to different backend servers without breaking a stateless API.

## Readiness Criteria

Explain the request/response model and HTTP's statelessness precisely, and correctly reason about what statelessness does and doesn't imply for scaling and authentication.

## References

### Microsoft Learn

- [HttpClient class](https://learn.microsoft.com/dotnet/api/system.net.http.httpclient)

### Other

- [MDN: HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
