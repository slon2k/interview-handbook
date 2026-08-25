# Global Exception Handling and ProblemDetails

## Definition

Global exception handling catches unhandled exceptions at the middleware level so they produce a consistent, safe error response instead of leaking a stack trace or crashing the pipeline for that request. **ProblemDetails** (RFC 9457) is a standard JSON shape for error responses, giving clients a predictable, machine-readable error contract instead of an ad hoc error format per endpoint.

```json
{
  "type": "https://example.com/errors/insufficient-stock",
  "title": "Insufficient stock",
  "status": 409,
  "detail": "Only 2 units of SKU-123 remain in stock",
  "instance": "/orders/42"
}
```

## Alternatives & Trade-offs

A global exception handler centralizes error formatting and logging in one place instead of wrapping every action in a try/catch. `ProblemDetails` standardizes the *shape* of that error response so client-side error handling doesn't need per-endpoint special cases. The trade-off of centralizing is that a global handler can't always add context as precise as a local try/catch closer to the failure — the two are complementary: catch specific, expected exceptions locally where you can respond precisely, and let a global handler catch everything else safely.

## How It Works

### Global exception-handling middleware

```csharp
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var feature = context.Features.Get<IExceptionHandlerFeature>();
        var exception = feature?.Error;

        var problem = new ProblemDetails
        {
            Status = StatusCodes.Status500InternalServerError,
            Title = "An unexpected error occurred",
            // deliberately not exposing exception.Message or stack trace to the client
        };

        context.Response.StatusCode = problem.Status.Value;
        await context.Response.WriteAsJsonAsync(problem);
    });
});
```

### Mapping specific exceptions to specific status codes

```csharp
app.UseExceptionHandler(errorApp =>
{
    errorApp.Run(async context =>
    {
        var exception = context.Features.Get<IExceptionHandlerFeature>()?.Error;
        var (status, title) = exception switch
        {
            InsufficientStockException => (StatusCodes.Status409Conflict, "Insufficient stock"),
            NotFoundException => (StatusCodes.Status404NotFound, "Resource not found"),
            _ => (StatusCodes.Status500InternalServerError, "An unexpected error occurred")
        };

        context.Response.StatusCode = status;
        await context.Response.WriteAsJsonAsync(new ProblemDetails { Status = status, Title = title });
    });
});
```

### `IExceptionHandler` (.NET 8+) — a more structured alternative

```csharp
public class InsufficientStockExceptionHandler : IExceptionHandler
{
    public async ValueTask<bool> TryHandleAsync(HttpContext context, Exception exception, CancellationToken ct)
    {
        if (exception is not InsufficientStockException ex) return false; // let the next handler try
        context.Response.StatusCode = StatusCodes.Status409Conflict;
        await context.Response.WriteAsJsonAsync(new ProblemDetails { Status = 409, Title = "Insufficient stock", Detail = ex.Message }, ct);
        return true; // handled
    }
}

builder.Services.AddExceptionHandler<InsufficientStockExceptionHandler>();
builder.Services.AddProblemDetails(); // fallback ProblemDetails for anything not handled above
```

### Never leak internal details to the client

```csharp
// WRONG: exposes internal implementation details (stack trace, SQL, file paths) to callers
new ProblemDetails { Detail = exception.ToString() }

// RIGHT: log the full exception server-side, return only safe, intentional detail to the client
_logger.LogError(exception, "Unhandled exception processing order");
new ProblemDetails { Detail = "An unexpected error occurred while processing your request." }
```

## Application

Register a global exception handler for every ASP.NET Core API, mapping known application exceptions to appropriate status codes and `ProblemDetails` responses, with a safe generic fallback for anything unexpected. Log the full exception detail server-side always; only return client-safe detail in the response body.

## Common Mistakes

- Leaking exception messages, stack traces, or internal details (SQL, file paths, connection strings) into the client-facing response.
- Having no global exception handler at all, letting an unhandled exception either crash the process or leak a framework-default error page in production.
- Using generic `500` for every exception instead of mapping known exception types to more specific, useful status codes (`404`, `409`, `422`).
- Not logging the exception server-side just because it's being handled and turned into a clean client response — losing the diagnostic information needed to actually fix the underlying issue.

## Common Interview Questions

### Basic
- What does `ProblemDetails` standardize, and why does that matter for API consumers?
- Why shouldn't a global exception handler expose the raw exception message to clients?

### Intermediate
- How would you map different application exception types to different HTTP status codes in a global handler?
- What's the difference between `UseExceptionHandler` middleware and the `IExceptionHandler` interface (.NET 8+)?

### Advanced
- How do you balance centralizing error handling globally versus catching specific, expected exceptions locally where more context is available?
- How would you ensure every unhandled exception is both logged with full detail and returned to the client with only safe detail?

### Follow-up Questions
- Does a global exception handler replace the need for any local try/catch blocks?
- Is `ProblemDetails` specific to ASP.NET Core, or a general HTTP standard?

### Code Prediction
An `InsufficientStockException` is thrown deep in a service call with no local try/catch, and a global exception handler maps that exception type to `409 Conflict`. What status code and body does the client receive, and what would the client receive instead if the exception type mapping were missing from the handler?

## Practical Tasks

- Implement a global exception handler mapping at least three custom exception types to appropriate `ProblemDetails` responses.
- Verify that internal exception details never reach the client response while still being fully logged server-side.
- Migrate a `UseExceptionHandler`-based approach to the `IExceptionHandler` interface and compare the two.

## Readiness Criteria

Implement a global exception handler producing consistent `ProblemDetails` responses, correctly map known exception types to specific status codes, and never leak internal details to clients while still logging them fully.

## References

### Microsoft Learn

- [Handle errors in ASP.NET Core](https://learn.microsoft.com/aspnet/core/web-api/handle-errors)
- [ProblemDetails class](https://learn.microsoft.com/dotnet/api/microsoft.aspnetcore.mvc.problemdetails)

### Other

- [RFC 9457: Problem Details for HTTP APIs](https://www.rfc-editor.org/rfc/rfc9457)
