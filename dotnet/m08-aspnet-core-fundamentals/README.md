# Module 8 - ASP.NET Core Fundamentals

**Status:** Complete  
**Priority:** Critical  
**Prerequisites:** Module 7 - HTTP, REST, and API Design  

## Scope

This is the main framework module for a backend or full-stack .NET role: how ASP.NET Core actually hosts and processes a request, from Kestrel through the middleware pipeline, routing, model binding, dependency injection, and the cross-cutting concerns (logging, exception handling, caching, rate limiting) every production API needs. It builds directly on Module 7's protocol-level vocabulary — HTTP methods, status codes, and REST concepts are assumed knowledge here, not re-explained.

## Learning Outcomes

By the end of this module, you should be able to:

- Explain the hosting model, Kestrel's role, and why a reverse proxy is typically used in production.
- Reason precisely about middleware ordering, including why authentication must precede authorization.
- Distinguish middleware from filters and choose the right layer for a given cross-cutting concern.
- Choose and correctly implement controllers or minimal APIs for a given API's size and needs.
- Use the DI container correctly, including keyed services, and diagnose captive-dependency lifetime bugs.
- Implement global exception handling producing consistent `ProblemDetails` responses.
- Implement background services, outbound HTTP calls, file handling, CORS, rate limiting, and health checks correctly and safely.

## Topics

### 1. Hosting and the Request Pipeline

- [ASP.NET Core hosting model and Kestrel](hosting-model-and-kestrel.md)
- [The request pipeline and middleware](request-pipeline-and-middleware.md)
- [Middleware vs. filters](middleware-versus-filters.md)
- [Routing](routing.md)

### 2. Building Endpoints

- [Controllers vs. minimal APIs](controllers-versus-minimal-apis.md)
- [Model binding](model-binding.md)
- [Validation](validation.md)

### 3. Dependency Injection and Configuration

- [The dependency-injection container](dependency-injection-container.md)
- [Service lifetimes: transient, scoped, and singleton](service-lifetimes.md)
- [Configuration and the options pattern](configuration-and-options-pattern.md)

### 4. Cross-Cutting Concerns

- [Logging](logging.md)
- [Global exception handling and ProblemDetails](global-exception-handling-and-problemdetails.md)
- [CORS](cors.md)
- [Rate limiting](rate-limiting.md)
- [Health checks](health-checks.md)
- [Basic caching (framework mechanics)](basic-caching.md)

### 5. Outbound and Background Work

- [File uploads and downloads](file-uploads-and-downloads.md)
- [Background services](background-services.md)
- [HttpClientFactory](httpclientfactory.md)
- [Cancellation of HTTP requests](cancellation-of-http-requests.md)

## Scope Boundaries

- HTTP protocol fundamentals, status codes, REST design, and API contracts belong in Module 7 - HTTP, REST, and API Design.
- Authentication and authorization mechanics (JWTs, cookies vs. bearer tokens, claims, policies) belong in Module 12 - Application Security (not yet published).
- Caching *strategy* (invalidation patterns, in-memory vs. distributed trade-offs at scale) belongs in Module 13 - Performance, Diagnostics, and Observability (not yet published); this module covers only the framework mechanics (`IMemoryCache`, Output Caching).
- General C# async/await mechanics, cancellation tokens, and threading belong in [Module 6 - Asynchronous Programming and Concurrency](../m06-async-concurrency/README.md); this module covers their ASP.NET Core-specific application (`RequestAborted`, hosted services).
- Dependency injection as a design *technique* (constructor injection, DIP) belongs in [Module 4 - Object-Oriented Design and Maintainable Code](../m04-oop-design/README.md); this module covers the ASP.NET Core container's specific mechanics (lifetimes, keyed services, validation).

## Suggested Learning Sequence

1. Hosting model, Kestrel, and the request pipeline/middleware.
2. Middleware vs. filters, and routing.
3. Controllers vs. minimal APIs, model binding, and validation.
4. The DI container, service lifetimes, and configuration/options.
5. Logging, global exception handling, CORS, rate limiting, health checks, and caching.
6. File handling, background services, `HttpClientFactory`, and request cancellation.

## Practical Deliverables

- Build a small API (controllers or minimal APIs) with correct middleware ordering, routing, and model validation.
- Reproduce and fix a captive-dependency bug caused by a singleton holding a scoped service.
- Implement global exception handling that maps at least three custom exception types to appropriate `ProblemDetails` responses.
- Implement a background service that safely resolves a scoped dependency via `IServiceScopeFactory`.
- Configure a typed `HttpClientFactory` client with a retry-and-timeout resilience policy.
- Implement partitioned rate limiting and separate liveness/readiness health checks.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and API familiarity.
- Intermediate questions involving common usage and configuration trade-offs.
- Advanced questions involving pipeline ordering, lifetime bugs, and production failure modes.
- Follow-up questions that test precise understanding rather than memorized definitions.
- Code-prediction questions grounded in concrete request/response or DI-registration examples.

## References

### Microsoft Learn

- [ASP.NET Core fundamentals overview](https://learn.microsoft.com/aspnet/core/fundamentals/)
- [ASP.NET Core Middleware](https://learn.microsoft.com/aspnet/core/fundamentals/middleware/)
- [Dependency injection in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/dependency-injection)
- [Handle errors in ASP.NET Core](https://learn.microsoft.com/aspnet/core/web-api/handle-errors)
