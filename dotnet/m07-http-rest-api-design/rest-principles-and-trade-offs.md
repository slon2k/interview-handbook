# REST Principles and Practical Trade-offs

## Definition

REST (Representational State Transfer) is an architectural style for networked APIs built on a small set of constraints: statelessness, a uniform resource-oriented interface, client-server separation, cacheability, and a layered system. In practice, "RESTful" API design usually means: resources identified by URLs, standard HTTP methods expressing intent, and representations (typically JSON) exchanged between client and server.

```
GET    /customers/7/orders     — list a customer's orders (resource-oriented URL)
POST   /customers/7/orders     — create a new order for that customer
GET    /orders/42              — a single order, addressable by its own URL
```

## Alternatives & Trade-offs

Strict REST (including HATEOAS — responses containing links to related actions/resources) gives maximum discoverability and decoupling between client and server evolution, at the cost of more complex responses and client logic that few real APIs actually implement fully. Most production APIs are "pragmatically RESTful" — resource-oriented URLs and correct HTTP method/status usage, without full HATEOAS — trading some of REST's theoretical benefits for simplicity. RPC-style or gRPC APIs (see `api-styles-rest-vs-rpc-vs-grpc-vs-graphql.md`) trade resource-orientation for direct action-oriented calls, which can be a better fit when the domain is naturally built around operations rather than resources.

## How It Works

### Resource orientation vs. action orientation

```
RESTful (resource-oriented):
POST /orders/42/cancel        — treats "cancel" as an action on a resource

RPC-style (action-oriented):
POST /cancelOrder?orderId=42  — treats "cancel" as a remote procedure call
```

Both are common in practice; the REST-purist view treats the resource-oriented version as more correct, but many real-world "RESTful" APIs mix in action-style endpoints for operations that don't map cleanly onto CRUD (cancel, approve, refund).

### The Richardson Maturity Model (awareness level)

```
Level 0 — a single URL, everything via POST (essentially RPC over HTTP)
Level 1 — multiple resource URLs, but not using HTTP methods meaningfully
Level 2 — resource URLs + correct HTTP methods and status codes (where most real APIs sit)
Level 3 — HATEOAS: responses include links describing available next actions
```

Most production REST APIs deliberately stop at Level 2 — Level 3's added complexity rarely pays for itself outside specific hypermedia-driven use cases.

### Statelessness constraint in practice

```
Each request must carry everything needed to process it (auth token, filters, pagination cursor) —
the server does not remember anything about the client between requests.
```

### When REST is a poor fit

```
- High-performance internal service-to-service calls where binary serialization and streaming matter more
  than human-readable JSON and browser-friendliness → gRPC often fits better.
- Clients needing to fetch deeply nested, client-specific shapes of data in one round trip → GraphQL often fits better.
- A small number of well-defined remote actions with no natural "resource" model → plain RPC-style calls may
  be simpler than forcing a resource abstraction.
```

## Application

Default to Level 2 REST for public and most internal HTTP APIs: resource-oriented URLs, correct HTTP methods and status codes, JSON representations. Reach for RPC-style endpoints for actions that don't map naturally to CRUD, and consider gRPC or GraphQL specifically when their trade-offs (see the next topic) genuinely fit the problem better than plain REST.

## Common Mistakes

- Treating full HATEOAS as mandatory for "real" REST, and either over-investing in it for an API that doesn't need it, or feeling guilty about a pragmatic Level 2 API that is, in practice, what almost all production REST APIs actually are.
- Modeling every operation as CRUD on a noun, forcing awkward URLs for actions that are naturally verbs (`PUT /orders/42/status` with a body of `{"status": "cancelled"}` vs. the arguably clearer `POST /orders/42/cancel`).
- Confusing "REST" with "JSON over HTTP" — JSON is just the most common representation format; REST is about the architectural constraints, not the serialization format.
- Ignoring statelessness by storing per-client context in server memory instead of requiring the client to send it, which breaks horizontal scalability.

## Common Interview Questions

### Basic
- What are the core constraints of REST?
- What does "resource-oriented" mean in the context of API design?

### Intermediate
- What is HATEOAS, and why do most real-world APIs not fully implement it?
- How would you design a URL for an action like "cancel an order" in a resource-oriented API?

### Advanced
- What is the Richardson Maturity Model, and where do most production APIs sit on it?
- When would you choose an RPC-style or gRPC API over REST for a given problem?

### Follow-up Questions
- Is JSON a requirement for an API to be considered RESTful?
- Does statelessness mean an API can't have "sessions" in any sense?

### Code Prediction
Given the two design choices `PUT /orders/42 {"status": "cancelled"}` and `POST /orders/42/cancel`, what's the practical difference in how a client uses each, and which better preserves the "resource" abstraction if cancellation has side effects beyond just changing a status field (e.g., triggering a refund)?

## Practical Tasks

- Design resource-oriented URLs and correct HTTP methods for a small e-commerce API (customers, orders, order items, payments).
- Identify an operation in a hypothetical API that doesn't map naturally to CRUD, and design both a resource-oriented and an RPC-style solution for it.
- Explain, for a specific scenario, why gRPC or GraphQL might be a better fit than REST.

## Readiness Criteria

Explain REST's core constraints and the Richardson Maturity Model, design resource-oriented APIs pragmatically (Level 2, not full HATEOAS), and reason about when REST isn't the best fit for a given problem.

## References

### Other

- [MDN: An overview of HTTP — REST concepts](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [Richardson Maturity Model (Martin Fowler)](https://martinfowler.com/articles/richardsonMaturityModel.html)
