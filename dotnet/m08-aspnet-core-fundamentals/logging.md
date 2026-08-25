# Logging

## Definition

ASP.NET Core's logging abstraction (`ILogger<T>`) decouples application code from specific logging backends (console, files, Application Insights, Seq) via **logging providers**, and supports **structured logging** — capturing log message parameters as named fields rather than baking them into a plain string, so they can be queried later.

```csharp
public class OrderService
{
    private readonly ILogger<OrderService> _logger;
    public OrderService(ILogger<OrderService> logger) => _logger = logger;

    public void Process(Order order)
    {
        _logger.LogInformation("Processing order {OrderId} for customer {CustomerId}", order.Id, order.CustomerId);
    }
}
```

## Alternatives & Trade-offs

Structured logging (named placeholders like `{OrderId}`) costs almost nothing over plain string interpolation but makes logs queryable by field in any log-aggregation backend that supports it (Seq, Application Insights, Elasticsearch) — you can filter "all logs where OrderId = 42" instead of grepping text. Plain string-interpolated logs (`$"Processing order {order.Id}"`) lose that structure entirely, flattening everything into unsearchable text.

## How It Works

### Structured logging — never string-interpolate the message template

```csharp
// WRONG: loses structure, and defeats log-provider optimizations like deferred formatting
_logger.LogInformation($"Processing order {order.Id}");

// RIGHT: OrderId becomes a queryable structured field in providers that support it
_logger.LogInformation("Processing order {OrderId}", order.Id);
```

With string interpolation, the message is fully formatted immediately regardless of whether the log level is even enabled; with the templated form, some providers can skip formatting entirely if the level is filtered out, and all providers can extract `OrderId` as a distinct field.

### Log levels and filtering

```csharp
_logger.LogTrace(...);       // extremely verbose, rarely enabled outside deep debugging
_logger.LogDebug(...);       // development diagnostics
_logger.LogInformation(...); // normal operational events
_logger.LogWarning(...);     // unexpected but recoverable
_logger.LogError(...);       // failure of an operation
_logger.LogCritical(...);    // application-wide failure
```

```json
// appsettings.json — configure minimum level per category
{ "Logging": { "LogLevel": { "Default": "Information", "MyApp.OrderService": "Debug" } } }
```

### Scopes — attaching context to a group of log entries

```csharp
using (_logger.BeginScope("OrderId: {OrderId}", order.Id))
{
    _logger.LogInformation("Validating order");
    _logger.LogInformation("Charging payment");
    // both log entries automatically include the OrderId scope value, without repeating it in each call
}
```

### Avoiding logging sensitive data

```csharp
_logger.LogInformation("User {UserId} logged in", user.Id);       // fine
_logger.LogInformation("User {Password} logged in", user.Password); // never log secrets/PII directly
```

## Application

Use `ILogger<T>` injected per class for automatic category naming. Always use structured message templates, not string interpolation. Use scopes to attach shared context (a correlation ID, an order ID) across a group of related log entries without repeating it manually. Configure log levels per category/environment so production doesn't drown in `Debug`-level noise.

## Common Mistakes

- String-interpolating log messages, losing structured-field querying and any level-based formatting optimization.
- Logging at `Information` or higher for extremely high-frequency events, flooding logs and increasing cost in hosted logging backends.
- Logging sensitive data (passwords, tokens, full credit card numbers) directly into log messages.
- Not using scopes for related log entries, forcing correlation to be reconstructed manually from timestamps or repeated IDs in every message.

## Common Interview Questions

### Basic
- What is `ILogger<T>`, and what does the generic parameter control?
- What is structured logging, and how does it differ from string interpolation?

### Intermediate
- What are logging scopes, and what problem do they solve?
- How would you configure different log levels for different parts of the application?

### Advanced
- Why can structured logging avoid unnecessary formatting work compared to string interpolation, depending on the configured log level?
- How would you design logging so a correlation ID automatically appears on every log entry for a given request?

### Follow-up Questions
- Should log messages ever include personally identifiable information?
- Can multiple logging providers be active simultaneously?

### Code Prediction
Given `_logger.LogDebug($"Processing order {order.Id}")` in a production app configured with a minimum log level of `Information`, does the string interpolation still execute even though the log entry is filtered out? What's the cost of that, and how does the templated form avoid it?

## Practical Tasks

- Convert a set of string-interpolated log statements to structured templates.
- Add a logging scope carrying a correlation ID across a multi-step operation and verify it appears on every related log entry.
- Configure per-category log levels so a noisy component logs at `Debug` while the rest of the app stays at `Information`.

## Readiness Criteria

Use structured logging correctly and explain why it matters, use scopes to attach shared context, and configure log levels appropriately per environment and category.

## References

### Microsoft Learn

- [Logging in .NET Core and ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/logging/)
- [High-performance logging with LoggerMessage](https://learn.microsoft.com/dotnet/core/extensions/high-performance-logging)
