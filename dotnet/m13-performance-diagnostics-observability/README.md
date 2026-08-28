# Module 13 - Performance, Diagnostics, and Observability

**Status:** Complete  
**Priority:** Medium to High  
**Prerequisites:** [Asynchronous Programming and Concurrency](../m06-async-concurrency/README.md), [ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md), [Entity Framework Core](../m10-entity-framework-core/README.md)

## Scope

This module's goal is not advanced optimization — it's demonstrating that you can investigate performance systematically. Every topic here builds on mechanics already covered elsewhere (async/CPU-vs-I/O from Module 6, GC from Module 5, collections/LINQ from Module 3, database performance from Modules 9-10, caching/logging/health-check mechanics from Module 8) and asks the operational question on top: how do you actually *measure*, *diagnose*, and *observe* these things in a running system, rather than just implementing them correctly once.

## Learning Outcomes

By the end of this module, you should be able to:

- Measure before optimizing, using profilers, benchmarks, or production telemetry rather than intuition.
- Correctly diagnose whether a bottleneck is CPU-bound or I/O-bound, and apply the matching fix.
- Identify and reduce GC pressure, collection/LINQ performance issues, and N+1 queries using real measurement.
- Choose an appropriate caching strategy and design reliable invalidation.
- Instrument metrics, tracing, structured logging, and correlation IDs as complementary observability signals.
- Use OpenTelemetry for vendor-neutral instrumentation, and .NET's diagnostic tools for detailed investigation.
- Design and run load and stress tests that reveal real capacity limits and failure modes.
- Apply timeouts, retries, and circuit breakers correctly, including their interaction with idempotency.

## Topics

### 1. Investigation Fundamentals

- [Measuring before optimizing](measure-before-optimizing.md)
- [Diagnosing CPU-bound vs. I/O-bound bottlenecks](cpu-vs-io-bound-bottlenecks.md)

### 2. Memory and Data Access Performance

- [Allocations and GC pressure](allocations-and-gc-pressure.md)
- [Common collection and LINQ performance issues](collection-and-linq-performance.md)
- [Database query performance and N+1 problems](database-query-performance-and-n-plus-one.md)
- [Caching strategies: in-memory vs. distributed](caching-strategies.md)

### 3. Observability

- [Metrics and tracing](metrics-and-tracing.md)
- [Structured logging and correlation IDs, revisited for observability](structured-logging-and-correlation-ids.md)
- [OpenTelemetry and Application Insights awareness](opentelemetry-and-application-insights.md)
- [Health checks in a monitoring and observability context](health-checks-and-monitoring.md)

### 4. Diagnosis and Validation

- [Profilers and diagnostic tools](profilers-and-diagnostic-tools.md)
- [Load and stress testing](load-and-stress-testing.md)
- [Basic resilience: timeouts, retries, and circuit breakers](resilience-timeouts-retries-circuit-breakers.md)

## Scope Boundaries

- GC and memory-model fundamentals belong in [Module 5 - Exceptions, Resources, and Memory Management](../m05-exceptions-resources-memory/README.md); this module covers diagnosing and reducing allocation pressure in practice.
- Async/await mechanics and CPU-vs-I/O-bound classification fundamentals belong in [Module 6 - Asynchronous Programming and Concurrency](../m06-async-concurrency/README.md); this module covers diagnosing which category a real bottleneck falls into.
- Collection choice and LINQ deferred execution fundamentals belong in [Module 3 - Collections, LINQ, and Basic Algorithms](../m03-collections-linq/README.md); this module covers the recurring performance mistakes in practice.
- Database indexing and execution plans belong in [Module 9 - Relational Databases and SQL](../m09-relational-databases-and-sql/README.md); N+1 queries and EF Core-specific performance belong in [Module 10 - Entity Framework Core](../m10-entity-framework-core/README.md); this module covers detecting these in a running system.
- Caching, logging, and health-check *mechanics* (how to implement them in ASP.NET Core) belong in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md); this module covers the strategic and operational layer on top.
- Rate limiting as a security control belongs in [Module 12 - Application Security](../m12-application-security/README.md); this module's resilience topic covers timeouts/retries/circuit breakers as an availability concern.

## Suggested Learning Sequence

1. Measuring before optimizing, and diagnosing CPU-bound vs. I/O-bound bottlenecks.
2. GC pressure, collection/LINQ performance, database performance and N+1, caching strategies.
3. Metrics, tracing, structured logging/correlation IDs, OpenTelemetry, and health checks as observability signals.
4. Profilers and diagnostic tools, load/stress testing, and resilience patterns.

## Practical Deliverables

- Diagnose a simulated slow endpoint by measuring rather than guessing, correctly classifying it as CPU-bound or I/O-bound and applying the matching fix.
- Reduce GC pressure in a benchmarked hot path using `[MemoryDiagnoser]` to prove the improvement.
- Detect an N+1 query problem via query-count instrumentation, and design a cache-invalidation strategy for a read-heavy endpoint.
- Instrument an endpoint with metrics, tracing, and correlation-ID-propagated structured logging, and correlate all three during a simulated incident.
- Use `dotnet-trace` or `dotnet-dump` to diagnose a deliberately introduced CPU or memory problem.
- Design and run a load test and a stress test for the same endpoint, documenting the difference in what each reveals.
- Configure a resilience pipeline (timeout, retry, circuit breaker) for an unreliable dependency, paired with an idempotency key where needed.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and tool familiarity.
- Intermediate questions involving diagnostic technique and configuration trade-offs.
- Advanced questions involving systematic investigation design and production-scale reasoning.
- Follow-up questions that test precise understanding rather than memorized definitions.
- Code-prediction questions grounded in concrete measurement scenarios, since this module is fundamentally about demonstrating investigative process, not memorized facts.

## References

### Microsoft Learn

- [.NET observability with OpenTelemetry](https://learn.microsoft.com/dotnet/core/diagnostics/observability-with-otel)
- [Performance profiling overview](https://learn.microsoft.com/visualstudio/profiling/)
- [dotnet-trace, dotnet-counters, dotnet-dump](https://learn.microsoft.com/dotnet/core/diagnostics/)
- [Build resilient HTTP apps](https://learn.microsoft.com/dotnet/core/resilience/http-resilience)

### Other

- [BenchmarkDotNet documentation](https://benchmarkdotnet.org/)
- [OpenTelemetry official documentation](https://opentelemetry.io/docs/)
