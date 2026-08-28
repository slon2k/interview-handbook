# Structured Logging and Correlation IDs, Revisited for Observability

## Definition

Module 8 introduced structured logging (`ILogger<T>`, message templates) at the mechanics level; this topic is about logging's role specifically as one of the three observability pillars alongside metrics and tracing (previous topic) — and correlation IDs as the connective thread across all three, and across service boundaries in a distributed system.

```csharp
// A log entry that's genuinely useful for observability carries the same correlation ID
// that the corresponding trace and any related metric dimension would carry
_logger.LogInformation("Order {OrderId} processed in {ElapsedMs}ms", order.Id, elapsedMs);
```

## Alternatives & Trade-offs

Logging every detail at high verbosity gives maximum information for any single investigation, but at real cost — storage, processing, and the practical difficulty of finding a signal in an overwhelming volume of noise. Logging deliberately, at appropriate levels, with consistent structure and correlation, trades some completeness for something actually navigable during an incident, which is what matters in practice under time pressure.

## How It Works

### Correlation IDs across service boundaries — not just within one service

```csharp
// Incoming request: read an existing correlation ID, or generate one if this is the entry point
var correlationId = context.Request.Headers["X-Correlation-Id"].FirstOrDefault() ?? Guid.NewGuid().ToString();

// Propagate it to any downstream service call
httpClient.DefaultRequestHeaders.Add("X-Correlation-Id", correlationId);

using (_logger.BeginScope(new Dictionary<string, object> { ["CorrelationId"] = correlationId }))
{
    _logger.LogInformation("Processing order"); // every log line in this scope carries the ID automatically
}
```

Without this propagation, a single logical operation spanning three microservices produces three separate, unconnected log trails — reconstructing what actually happened requires manually correlating by timestamp and guesswork, instead of a single query for one ID.

### Structured fields as the actual observability payload, not just readable text

```csharp
_logger.LogWarning("Payment failed for order {OrderId}: {ErrorCode}", order.Id, errorCode);
```

In a log-aggregation platform (Seq, Application Insights, Elasticsearch), this becomes queryable: "show me every `ErrorCode = 'INSUFFICIENT_FUNDS'` warning from the last hour, grouped by `OrderId`" — a query the equivalent plain-text log message couldn't support without fragile string parsing.

### Log levels as a triage signal during an incident

```
During an active incident, filtering to Warning/Error/Critical first, then drilling into
Information/Debug for a SPECIFIC correlation ID once the relevant timeframe/request is identified,
is far more effective than scrolling through everything at Information level for the whole system.
```

### The cost side — why "just log everything" isn't actually free

```
Volume: every log line has a real storage and ingestion cost, especially at Debug/Trace levels in production.
Signal-to-noise: an overwhelming log volume makes finding the actually relevant entries during
                  an incident harder, not easier, even with good tooling.
```

## Application

Structure every log entry with consistent, queryable fields, and propagate a correlation ID through every service boundary a request crosses — including asynchronous flows (message queues, background jobs), not just synchronous HTTP calls. Set log levels deliberately so a triage workflow (filter to warnings/errors first, then drill into detail for a specific correlation ID) is actually practical during an incident.

## Common Mistakes

- Not propagating a correlation ID across service boundaries, leaving a single logical operation's logs scattered and unconnected across services.
- Logging at `Information` or higher for high-frequency, low-value events, drowning out genuinely important entries during an incident.
- Treating log volume as free, without considering the real storage/ingestion cost and the signal-to-noise cost of an overwhelming log stream.
- Logging plain, unstructured text for data that would be far more useful as structured, queryable fields (an error code, a status, an ID).

## Common Interview Questions

### Basic
- Why does structured logging matter more in an observability context than just "readable log messages"?
- What is a correlation ID, and why does it need to be propagated across service boundaries, not just within one service?

### Intermediate
- How would you propagate a correlation ID through an asynchronous flow (a message queue), not just a synchronous HTTP call chain?
- What's the practical cost of logging everything at high verbosity in production?

### Advanced
- How would you design a logging strategy that supports fast triage during an active incident, given the volume/signal-to-noise trade-off?
- How does structured logging combined with correlation IDs let you reconstruct a full multi-service request trail without a dedicated tracing system?

### Follow-up Questions
- Does correlation ID propagation require a dedicated tracing system, or can it be done with logging alone?
- Should correlation IDs be exposed to external clients (e.g., in an error response), and if so, why?

### Code Prediction
A request flows through three microservices, each logging independently with no correlation ID propagated between them. During an incident, how would an engineer reconstruct what happened across all three services for one specific failed request, and how does that compare to having a propagated correlation ID?

## Practical Tasks

- Implement correlation ID propagation across two services communicating over HTTP, including generating one at the entry point if none is provided.
- Design a log-level strategy for a service, distinguishing what belongs at Information versus Debug versus Warning/Error.
- Propagate a correlation ID through a background job triggered by a message queue, connecting it back to the originating HTTP request's ID.

## Readiness Criteria

Propagate correlation IDs across both synchronous and asynchronous service boundaries, structure log entries for queryability rather than just readability, and design log-level strategy that supports practical incident triage.

## References

### Microsoft Learn

- [Logging in .NET Core and ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/logging/)
- [.NET observability with OpenTelemetry](https://learn.microsoft.com/dotnet/core/diagnostics/observability-with-otel)
