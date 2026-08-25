# Cancellation of HTTP Requests

## Definition

`HttpContext.RequestAborted` is a `CancellationToken` that's signaled when the client disconnects or cancels the request before the server finishes processing it. Accepting this token (or a `CancellationToken` parameter, which ASP.NET Core binds to `RequestAborted` automatically) lets a handler stop expensive work early instead of continuing to compute a response nobody will receive.

```csharp
app.MapGet("/reports/{id:int}", async (int id, CancellationToken cancellationToken) =>
{
    var report = await _reportService.GenerateAsync(id, cancellationToken); // stops if the client disconnects
    return Results.Ok(report);
});
```

## Alternatives & Trade-offs

Ignoring request cancellation means the server keeps doing work (database queries, external calls, CPU-bound processing) for a response that will simply be discarded — wasted resources under load, and in extreme cases a real availability risk if abandoned requests pile up faster than they're cancelled. Honoring cancellation costs only a parameter and threading it through downstream calls (the same cooperative-cancellation model from `cancellation-and-timeouts.md` in Module 6), for a real resource-efficiency benefit.

## How It Works

### Binding `RequestAborted` automatically

```csharp
[HttpGet("{id:int}")]
public async Task<IActionResult> Get(int id, CancellationToken cancellationToken)
{
    // ASP.NET Core automatically supplies HttpContext.RequestAborted here — no manual wiring needed
    var order = await _repository.GetByIdAsync(id, cancellationToken);
    return order is null ? NotFound() : Ok(order);
}
```

A `CancellationToken` parameter on a controller action or minimal API handler is automatically bound to `HttpContext.RequestAborted` by the framework — nothing extra needs to be registered.

### Propagating the token through every downstream call

```csharp
public async Task<Report> GenerateAsync(int id, CancellationToken cancellationToken)
{
    var data = await _repository.GetDataAsync(id, cancellationToken);       // must accept and check it too
    var enriched = await _externalApi.EnrichAsync(data, cancellationToken); // same
    return BuildReport(enriched);
}
```

If any layer in the call chain drops the token (doesn't accept it, or accepts but ignores it), cancellation stops propagating past that point — the request keeps running underneath even though the client already disconnected.

### Combining request cancellation with an additional timeout

```csharp
app.MapGet("/reports/{id:int}", async (int id, CancellationToken requestAborted) =>
{
    using var timeoutCts = new CancellationTokenSource(TimeSpan.FromSeconds(30));
    using var combined = CancellationTokenSource.CreateLinkedTokenSource(requestAborted, timeoutCts.Token);
    var report = await _reportService.GenerateAsync(id, combined.Token); // stops on either signal
    return Results.Ok(report);
});
```

### What happens if cancellation isn't honored

```csharp
public async Task<Report> GenerateAsync(int id, CancellationToken cancellationToken)
{
    // cancellationToken accepted but never passed to the database call or checked —
    // the expensive query keeps running to completion even after the client disconnects
    var data = await _repository.GetDataAsync(id);
    return BuildReport(data);
}
```

## Application

Accept and propagate `CancellationToken` through every layer of a request-handling call chain — controller/handler, service, repository, external API calls — the same discipline as Module 6's general cancellation guidance, but specifically valuable here because HTTP requests are cancelled by client disconnects constantly under real-world network conditions (mobile clients losing signal, users navigating away, client-side timeouts).

## Common Mistakes

- Accepting a `CancellationToken` parameter in a handler but not threading it through to downstream repository/service/external calls, so cancellation silently stops propagating.
- Assuming request cancellation is a rare edge case, when in practice client disconnects and client-side timeouts happen constantly at any real scale.
- Not combining request cancellation with an additional server-side timeout, leaving a request that the client is still connected to (but that's taking pathologically long) with no bound on how long it can run.
- Catching `OperationCanceledException` and logging it as an error, when it's typically an expected, unremarkable outcome (the client just disconnected) rather than a bug.

## Common Interview Questions

### Basic
- What does `HttpContext.RequestAborted` represent?
- How does a `CancellationToken` parameter on a controller action get populated automatically?

### Intermediate
- Why does failing to propagate the cancellation token through downstream calls make accepting it in the handler pointless?
- Why does honoring request cancellation matter for server resource usage under load?

### Advanced
- How would you combine request-abort cancellation with an independent server-side timeout for a single request?
- How would you audit a codebase to find handlers that accept a `CancellationToken` but silently fail to propagate it?

### Follow-up Questions
- Should `OperationCanceledException` from an aborted request typically be logged as an error?
- Does honoring cancellation guarantee an in-progress database write is rolled back?

### Code Prediction
A client sends a request to an endpoint performing a 10-second database query, then disconnects after 2 seconds. If the handler's `CancellationToken` parameter is never passed into the repository call, does the database query stop after 2 seconds or continue running for the full 10?

## Practical Tasks

- Add `CancellationToken` propagation through a full call chain (handler → service → repository) and verify a simulated client disconnect actually stops the underlying work.
- Combine request-abort cancellation with a server-side timeout using a linked token source.
- Audit a small codebase for handlers that accept but don't propagate a `CancellationToken`.

## Readiness Criteria

Explain `RequestAborted` and its automatic binding, correctly propagate cancellation through a full call chain, and combine request cancellation with independent timeouts where appropriate.

## References

### Microsoft Learn

- [Cancel incomplete asynchronous work](https://learn.microsoft.com/aspnet/core/fundamentals/request-cancellation)
