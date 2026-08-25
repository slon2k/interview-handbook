# Module 7 - HTTP, REST, and API Design

**Status:** Complete  
**Priority:** Critical  
**Prerequisites:** [C# Language and Type System](../m02-csharp-language/README.md)  

## Scope

This module covers the protocol and contract level of building a web API: the HTTP request/response model, methods, status codes, headers, REST design principles, and the practical concerns of shaping an API contract (pagination, versioning, documentation). It is deliberately framework-agnostic — everything here applies before any ASP.NET Core specifics, the same way relational database fundamentals precede Entity Framework Core later in the roadmap.

The goal is a solid mental model of HTTP and API design that transfers regardless of framework, before layering ASP.NET Core's specific implementation of it in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md).

## Learning Outcomes

By the end of this module, you should be able to:

- Explain the HTTP request/response model and why HTTP is stateless.
- Choose the correct HTTP method and status code for a given scenario, including commonly confused pairs (`401`/`403`, `400`/`422`).
- Explain idempotency and design safe retry behavior, including idempotency keys for non-idempotent operations.
- Design a pragmatic, resource-oriented REST API and justify when REST isn't the best fit compared to RPC, gRPC, or GraphQL.
- Maintain an OpenAPI contract and reason about breaking vs. non-breaking API changes.
- Design pagination, filtering, and sorting contracts appropriate to a dataset's size and access pattern.

## Topics

### 1. HTTP Fundamentals

- [The HTTP request/response model](http-request-response-model.md)
- [HTTP methods and idempotency](http-methods-and-idempotency.md)
- [HTTP status codes](status-codes.md)
- [Headers and content types](headers-and-content-types.md)

### 2. API Design

- [REST principles and practical trade-offs](rest-principles-and-trade-offs.md)
- [API styles: REST vs. RPC vs. gRPC vs. GraphQL](api-styles-rest-vs-rpc-vs-grpc-vs-graphql.md)
- [OpenAPI/Swagger and API contracts](openapi-and-api-contracts.md)
- [Pagination, filtering, and sorting](pagination-filtering-and-sorting.md)
- [API versioning](api-versioning.md)

## Scope Boundaries

- ASP.NET Core's specific implementation of these concepts — middleware, routing, controllers vs. minimal APIs, model binding, DI, `ProblemDetails` — belongs in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md).
- The system-level choice between synchronous request/response (of which REST/gRPC are implementations) and asynchronous messaging/events belongs in Module 14 - Architecture and System Design Fundamentals (not yet published); this module covers the shape of one synchronous HTTP API, not how services in a larger system choose to communicate.
- Authentication and authorization mechanics (JWTs, cookies vs. bearer tokens, claims) belong in Module 12 - Application Security (not yet published).
- Caching *strategy* (invalidation, in-memory vs. distributed) belongs in Module 13 - Performance, Diagnostics, and Observability (not yet published); this module only covers HTTP-level caching headers (`ETag`, `Cache-Control`).

## Suggested Learning Sequence

1. The request/response model, HTTP methods, and idempotency.
2. Status codes and headers/content types.
3. REST principles, the Richardson Maturity Model, and when REST isn't the best fit.
4. API styles: REST vs. RPC vs. gRPC vs. GraphQL.
5. OpenAPI contracts, pagination/filtering/sorting, and versioning.

## Practical Deliverables

- Classify a list of API operations by correct HTTP method, idempotency, and expected status code.
- Design resource-oriented URLs and status-code handling for a small e-commerce API.
- Design an idempotency-key mechanism for a non-idempotent payment endpoint.
- Write an OpenAPI document for a small set of endpoints and use it to detect a deliberately introduced breaking change.
- Implement and compare offset-based and keyset-based pagination for the same dataset.
- Justify, for a stated scenario, a choice between REST, gRPC, and GraphQL.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and protocol-level facts.
- Intermediate questions involving common usage and design trade-offs.
- Advanced questions involving system-level reasoning (retries, breaking changes, performance at scale).
- Follow-up questions that test precise understanding rather than memorized definitions.
- Code-prediction questions grounded in concrete request/response examples.

## References

### Other

- [MDN: HTTP overview](https://developer.mozilla.org/en-US/docs/Web/HTTP/Overview)
- [RFC 9110: HTTP Semantics](https://www.rfc-editor.org/rfc/rfc9110)
- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Richardson Maturity Model (Martin Fowler)](https://martinfowler.com/articles/richardsonMaturityModel.html)
