# Background Services

## Definition

A background service is long-running work hosted alongside the web application, managed by the generic host's lifecycle rather than triggered by an incoming HTTP request. `IHostedService` is the base interface; `BackgroundService` is an abstract base class implementing it with a simpler `ExecuteAsync` method for the common continuous-loop case.

```csharp
public class OrderCleanupService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            await CleanupStaleOrdersAsync(stoppingToken);
            await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
        }
    }
}

builder.Services.AddHostedService<OrderCleanupService>();
```

## Alternatives & Trade-offs

A hosted `BackgroundService` runs in-process with the web app — simple to deploy and share code/DI with the rest of the app, but tied to the web app's own lifecycle and scaling (if you scale out to five instances, you now have five copies of that background service running, which may or may not be intended). A separate worker service or scheduled job (e.g., Azure Functions timer trigger, Hangfire, Quartz.NET) runs independently, letting you scale the background work separately from the web app, at the cost of a separate deployable and possibly a separate way to share code.

## How It Works

### `BackgroundService` and cooperative shutdown

```csharp
public class OrderCleanupService : BackgroundService
{
    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            try
            {
                await CleanupStaleOrdersAsync(stoppingToken);
            }
            catch (Exception ex) when (ex is not OperationCanceledException)
            {
                // an unhandled exception here would stop the service permanently — catch and log instead
            }
            await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
        }
    }
}
```

`stoppingToken` is signaled when the host begins shutting down, giving the loop a chance to exit cleanly rather than being killed mid-operation.

### Using scoped services from a singleton-hosted background service

```csharp
public class OrderCleanupService : BackgroundService
{
    private readonly IServiceScopeFactory _scopeFactory;
    public OrderCleanupService(IServiceScopeFactory scopeFactory) => _scopeFactory = scopeFactory;

    protected override async Task ExecuteAsync(CancellationToken stoppingToken)
    {
        while (!stoppingToken.IsCancellationRequested)
        {
            using var scope = _scopeFactory.CreateScope();
            var repository = scope.ServiceProvider.GetRequiredService<IOrderRepository>();
            await repository.DeleteStaleOrdersAsync(stoppingToken);
            await Task.Delay(TimeSpan.FromMinutes(5), stoppingToken);
        }
    }
}
```

`BackgroundService` instances are registered as singletons by the host, so they can't directly constructor-inject a scoped `DbContext`-backed repository — this is the same captive-dependency issue covered in `service-lifetimes.md`, resolved the same way with `IServiceScopeFactory`.

### `IHostedService` directly, for start/stop hooks rather than a continuous loop

```csharp
public class WarmupService : IHostedService
{
    public Task StartAsync(CancellationToken cancellationToken) { /* run once at startup */ return Task.CompletedTask; }
    public Task StopAsync(CancellationToken cancellationToken) { /* run once at shutdown */ return Task.CompletedTask; }
}
```

## Application

Use `BackgroundService` for continuous, in-process background work sharing the web app's deployment (periodic cleanup, queue consumers, cache warmers). Use a separate worker/scheduler when the work needs independent scaling, different resource requirements, or must run exactly-once even when the web app is scaled to multiple instances.

## Common Mistakes

- Letting an unhandled exception escape `ExecuteAsync`'s loop, silently stopping the background service permanently with no automatic restart.
- Constructor-injecting a scoped service directly into a `BackgroundService` (which is registered as a singleton), causing a captive-dependency bug.
- Not honoring `stoppingToken`, causing the host to have to forcibly kill the service on shutdown instead of allowing a clean stop.
- Running a `BackgroundService` that assumes it's the only instance running, when the web app is actually scaled to multiple instances — each instance runs its own copy unless something (a distributed lock, a separate singleton worker) prevents duplicate execution.

## Common Interview Questions

### Basic
- What is `BackgroundService`, and how does it relate to `IHostedService`?
- Why can't a `BackgroundService` directly constructor-inject a scoped service?

### Intermediate
- What does the `stoppingToken` passed to `ExecuteAsync` represent, and why should it be honored?
- What happens if an unhandled exception escapes a `BackgroundService`'s `ExecuteAsync` loop?

### Advanced
- How would you prevent a periodic background task from running duplicated across multiple scaled-out instances of the same web app?
- When would you choose a separate worker service or scheduler over an in-process `BackgroundService`?

### Follow-up Questions
- Does scaling a web app to multiple instances also scale its hosted `BackgroundService` instances?
- Can a `BackgroundService` be manually stopped and restarted at runtime?

### Code Prediction
A `BackgroundService`'s `ExecuteAsync` throws an unhandled `NullReferenceException` on its first loop iteration, with no try/catch around the work. What happens to the periodic cleanup this service was supposed to perform, for the rest of the application's lifetime?

## Practical Tasks

- Implement a `BackgroundService` performing periodic cleanup work, correctly resolving a scoped dependency via `IServiceScopeFactory`.
- Add proper exception handling inside the loop so a single failed iteration doesn't stop the service permanently.
- Discuss how you would prevent duplicate execution of a scheduled task across multiple scaled-out instances.

## Readiness Criteria

Implement `BackgroundService` correctly including scoped-dependency resolution and cooperative cancellation, and reason about the implications of scaling for in-process background work.

## References

### Microsoft Learn

- [Background tasks with hosted services](https://learn.microsoft.com/aspnet/core/fundamentals/host/hosted-services)
