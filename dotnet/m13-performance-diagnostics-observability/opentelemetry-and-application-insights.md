# OpenTelemetry and Application Insights Awareness

## Definition

**OpenTelemetry (OTel)** is a vendor-neutral standard and set of libraries for collecting metrics, traces, and logs, letting an application instrument itself once and send data to whichever backend a team chooses (Jaeger, Prometheus, Application Insights, and others), rather than being locked into one vendor's SDK. **Application Insights** is Azure's specific observability product — a backend (and historically, its own SDK) for collecting and visualizing exactly this kind of telemetry, now itself commonly fed via OpenTelemetry rather than its older proprietary SDK.

```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(tracing => tracing.AddAspNetCoreInstrumentation().AddHttpClientInstrumentation())
    .WithMetrics(metrics => metrics.AddAspNetCoreInstrumentation())
    .UseAzureMonitor(); // sends OTel-collected data to Application Insights specifically
```

## Alternatives & Trade-offs

Instrumenting directly against a vendor-specific SDK (the older Application Insights SDK, for example) is simple to set up for that one vendor, but locks the instrumentation code to that backend — switching observability platforms later means re-instrumenting. Instrumenting via OpenTelemetry keeps the application-level code vendor-neutral; only the *exporter* configuration (which backend receives the data) changes if the team switches platforms, at the cost of OTel being a broader, sometimes more complex standard to configure initially.

## How It Works

### OpenTelemetry's three signal types, unified under one instrumentation approach

```csharp
builder.Services.AddOpenTelemetry()
    .WithTracing(t => t.AddSource("MyApp").AddAspNetCoreInstrumentation())
    .WithMetrics(m => m.AddMeter("MyApp").AddAspNetCoreInstrumentation())
    .WithLogging(); // logs, traces, and metrics all instrumented through the same overall OTel setup
```

Automatic instrumentation packages (`AddAspNetCoreInstrumentation`, `AddHttpClientInstrumentation`, EF Core instrumentation) provide baseline tracing/metrics for common .NET components with minimal manual code, while custom `ActivitySource`/`Meter` usage (as in the previous topic) covers application-specific detail.

### Exporting to different backends without changing instrumentation code

```csharp
// Exporting to Application Insights
.UseAzureMonitor();

// Exporting to a self-hosted Jaeger/Prometheus stack instead — the instrumentation code above is unchanged
.WithTracing(t => t.AddOtlpExporter());
```

This is the concrete payoff of the vendor-neutral approach: switching where telemetry is sent is a configuration change, not an instrumentation rewrite.

### Application Insights as a specific backend — what it adds on top of raw OTel data

```
Application Insights provides: a hosted backend, a query language (KQL) for exploring collected
data, pre-built dashboards, alerting, and integration with the broader Azure Monitor ecosystem —
value distinct from OpenTelemetry itself, which is just the collection/instrumentation layer.
```

### Sampling configuration — controlling volume/cost at the OTel layer

```csharp
.WithTracing(t => t.SetSampler(new TraceIdRatioBasedSampler(0.1))) // sample 10% of traces
```

## Application

Instrument new applications with OpenTelemetry by default, to avoid vendor lock-in at the instrumentation layer, and choose a backend (Application Insights, or a self-hosted alternative) as an exporter configuration rather than baking vendor-specific SDK calls throughout application code. Use Application Insights (or an equivalent) specifically for the dashboarding, querying, and alerting capabilities it adds on top of raw collected telemetry.

## Common Mistakes

- Instrumenting directly against a vendor-specific SDK for new code, creating lock-in that a vendor-neutral OpenTelemetry approach would have avoided.
- Confusing OpenTelemetry (the instrumentation/collection standard) with Application Insights (a specific backend/product) — they operate at different layers and aren't mutually exclusive.
- Not configuring sampling for a high-volume production system, incurring unnecessary telemetry storage and processing cost.
- Assuming automatic instrumentation packages cover everything needed, without adding custom tracing/metrics for application-specific logic that the automatic packages have no way to know about.

## Common Interview Questions

### Basic
- What is OpenTelemetry, and what problem does it solve compared to a vendor-specific SDK?
- What is Application Insights, and how does it relate to OpenTelemetry?

### Intermediate
- How would you switch an OpenTelemetry-instrumented application from one observability backend to another?
- What does automatic instrumentation (e.g., `AddAspNetCoreInstrumentation`) provide, and what does it not cover?

### Advanced
- How would you design an observability strategy that avoids vendor lock-in while still taking advantage of a specific platform's (e.g., Application Insights') dashboarding and alerting features?
- How would you configure sampling to balance telemetry completeness against cost for a very high-traffic service?

### Follow-up Questions
- Does using OpenTelemetry mean you can't use Application Insights at all?
- Is custom instrumentation (manual `ActivitySource`/`Meter` usage) ever necessary alongside automatic instrumentation packages?

### Code Prediction
A team instruments their application using the older, Application-Insights-specific SDK directly throughout their codebase. A year later, they want to evaluate a different observability platform. What does this decision cost them, compared to if they had instrumented with OpenTelemetry and only configured an Application Insights exporter?

## Practical Tasks

- Configure OpenTelemetry with automatic ASP.NET Core and `HttpClient` instrumentation, exporting to a chosen backend.
- Add custom tracing spans for an application-specific operation that automatic instrumentation wouldn't capture on its own.
- Configure trace sampling for a simulated high-throughput scenario and explain the cost/completeness trade-off chosen.

## Readiness Criteria

Explain the relationship between OpenTelemetry and Application Insights (or any specific backend) at the right layer, instrument applications in a vendor-neutral way, and configure sampling appropriately for volume/cost.

## References

### Microsoft Learn

- [.NET observability with OpenTelemetry](https://learn.microsoft.com/dotnet/core/diagnostics/observability-with-otel)
- [Application Insights overview](https://learn.microsoft.com/azure/azure-monitor/app/app-insights-overview)

### Other

- [OpenTelemetry official documentation](https://opentelemetry.io/docs/)
