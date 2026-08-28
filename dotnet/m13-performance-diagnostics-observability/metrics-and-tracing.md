# Metrics and Tracing

## Definition

**Metrics** are numeric measurements aggregated over time — request counts, error rates, latency percentiles, queue depths — answering "how is the system behaving, in aggregate?" **Tracing** follows one specific request's journey through a system, including across service boundaries, answering "what exactly happened for this one request, and where did the time go?" Together with logging (Module 8), these are the three pillars of observability.

```
Metric example:  http_requests_total{status="500"} increased by 40 in the last minute
Trace example:   Request abc123 -> OrderService.CreateOrder (12ms) -> InventoryService.Reserve (340ms) -> ...
```

## Alternatives & Trade-offs

Metrics are cheap to collect and store (pre-aggregated numbers, not full detail) and great for dashboards and alerting on trends, but a metric alone can't tell you *why* a specific request was slow — only that latency went up. Tracing captures the detailed, per-request causal chain needed to answer "why," but is more expensive to collect and store in full for every request, which is why sampling (tracing only a fraction of requests) is common in high-volume systems.

## How It Works

### Metrics — aggregated, cheap, good for "what" and "how much"

```csharp
// .NET's built-in System.Diagnostics.Metrics API
var counter = meter.CreateCounter<int>("orders.created");
counter.Add(1, new KeyValuePair<string, object?>("status", "success"));

var histogram = meter.CreateHistogram<double>("orders.processing_duration_ms");
histogram.Record(elapsedMs);
```

A dashboard built on these metrics can show "error rate spiked from 0.1% to 5% at 14:32" — a clear, actionable signal that something changed, without needing per-request detail to notice it.

### Tracing — per-request, detailed, good for "why" and "where"

```csharp
using var activity = _activitySource.StartActivity("CreateOrder");
activity?.SetTag("customer.id", customerId);
// ... work happens, potentially calling another service which continues the SAME trace ...
```

```
Trace abc123:
  CreateOrder (total: 850ms)
    ├── ValidateOrder (5ms)
    ├── ReserveInventory (340ms)     <- this is where the time actually went
    └── SaveOrder (15ms)
```

A trace reconstructs the full causal chain for one specific slow request — exactly the kind of detail a metric's aggregate number can't provide, and exactly what's needed to answer "why was *this* request slow" rather than just "requests are slower on average."

### Correlation IDs — the thread connecting logs, metrics, and traces for one request

```csharp
using (_logger.BeginScope(new Dictionary<string, object> { ["CorrelationId"] = correlationId }))
{
    _logger.LogInformation("Processing order"); // this log entry, and the trace, and any related metric
                                                    // dimension, can all be tied back to the SAME correlationId
}
```

A correlation ID (often the trace ID itself) lets an engineer pivot from "I see an error in the logs" to "let me see the full trace for that exact request" to "let me check if this correlates with a metric spike" — without a correlation ID, these three signals are disconnected.

### Percentiles matter more than averages for latency

```
Average latency: 120ms  <- can hide a bimodal distribution (most requests fast, some very slow)
p50: 80ms, p95: 450ms, p99: 2100ms  <- reveals that 1% of requests are dramatically slower than typical
```

A single average can look perfectly healthy while a meaningful fraction of real users experience a much worse actual latency — percentiles (especially p95/p99) surface this in a way an average cannot.

## Application

Instrument metrics for aggregate trend visibility and alerting (error rates, latency percentiles, throughput). Instrument tracing for the detailed, per-request "why" investigation, using sampling to control volume/cost at scale. Propagate a correlation/trace ID through every log entry, metric dimension, and service boundary so all three signals can be tied back to the same originating request.

## Common Mistakes

- Relying only on average latency, missing a meaningful tail of slow requests that percentiles would reveal.
- Instrumenting metrics but not tracing (or vice versa), losing either the aggregate trend view or the per-request causal detail — they answer different questions and neither substitutes for the other.
- Not propagating a correlation ID across service boundaries, making it impossible to reconstruct the full picture of a single request that touched multiple services.
- Tracing every single request at full detail in a very high-volume system without sampling, incurring unnecessary storage and processing cost.

## Common Interview Questions

### Basic
- What's the difference between a metric and a trace?
- Why do latency percentiles matter more than the average for diagnosing user-experienced performance?

### Intermediate
- What is a correlation ID, and what problem does it solve across logs, metrics, and traces?
- Why might a system sample traces rather than capturing every single request in full detail?

### Advanced
- How would you use a metric spike as the starting point for an investigation, and what would the corresponding trace add that the metric alone couldn't tell you?
- How would you design correlation ID propagation across a multi-service system, including through message queues (Module 14) as well as synchronous HTTP calls?

### Follow-up Questions
- Can a single correlation ID span both a synchronous HTTP call chain and an asynchronous message-queue-based flow?
- Does tracing replace the need for structured logging?

### Code Prediction
A dashboard shows p50 latency at 80ms and p99 latency at 3000ms for the same endpoint. What does this distribution suggest about the nature of the performance problem, compared to a hypothetical scenario where p50 and p99 were both around 200ms?

## Practical Tasks

- Instrument a simple counter and histogram metric for an endpoint, and build a basic dashboard view of error rate and latency percentiles.
- Add distributed tracing to a multi-step operation and inspect the resulting trace to identify which step dominates total latency.
- Implement correlation ID propagation across two services communicating over HTTP, verifying logs from both services share the same ID for a single request.

## Readiness Criteria

Explain what metrics and tracing each answer that the other doesn't, use percentiles rather than averages for latency analysis, and propagate correlation IDs across service and communication-mode boundaries.

## References

### Microsoft Learn

- [.NET observability with OpenTelemetry](https://learn.microsoft.com/dotnet/core/diagnostics/observability-with-otel)
- [Metrics in .NET](https://learn.microsoft.com/dotnet/core/diagnostics/metrics)
