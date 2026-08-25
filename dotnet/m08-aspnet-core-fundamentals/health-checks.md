# Health Checks

## Definition

A health check is an endpoint reporting whether an application (and its critical dependencies — database, cache, downstream services) is able to function, used by orchestrators (Kubernetes), load balancers, and monitoring systems to decide whether to route traffic to an instance or restart it.

```csharp
builder.Services.AddHealthChecks()
    .AddDbContextCheck<AppDbContext>()
    .AddCheck<RedisHealthCheck>("redis");

app.MapHealthChecks("/health");
```

## Alternatives & Trade-offs

A trivial health check ("is the process running and able to respond at all") is cheap and fast but doesn't catch the common failure mode of "process is up but its database connection is broken." A deep health check (verifying the database, cache, and downstream dependencies) catches more real failures but costs more per check and, if a downstream dependency is flaky, can cause an otherwise-healthy instance to be marked unhealthy and removed from rotation unnecessarily — the check needs to distinguish "I can't serve any traffic" from "one non-critical dependency is degraded."

## How It Works

### Liveness vs. readiness — two different questions

```
Liveness  — "is the process alive and responsive at all?" Used to decide whether to restart the instance.
Readiness — "is the process ready to receive traffic?" Used to decide whether to route traffic to it.
```

```csharp
app.MapHealthChecks("/health/live", new HealthCheckOptions { Predicate = _ => false }); // no dependency checks, just "is the process up"
app.MapHealthChecks("/health/ready", new HealthCheckOptions { Predicate = check => check.Tags.Contains("ready") }); // full dependency check
```

Conflating these is a common mistake: if liveness includes a database check, a temporary database blip can cause a perfectly healthy process to be needlessly restarted by the orchestrator — restarting doesn't fix a database outage, it just adds churn.

### A custom health check

```csharp
public class RedisHealthCheck : IHealthCheck
{
    private readonly IConnectionMultiplexer _redis;
    public RedisHealthCheck(IConnectionMultiplexer redis) => _redis = redis;

    public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken)
    {
        try
        {
            await _redis.GetDatabase().PingAsync();
            return HealthCheckResult.Healthy();
        }
        catch (Exception ex)
        {
            return HealthCheckResult.Unhealthy("Redis unreachable", ex);
        }
    }
}
```

### Distinguishing "degraded" from "unhealthy"

```csharp
public async Task<HealthCheckResult> CheckHealthAsync(HealthCheckContext context, CancellationToken cancellationToken)
{
    if (await IsCacheReachableAsync())
        return HealthCheckResult.Healthy();
    return HealthCheckResult.Degraded("Cache unreachable — serving without cache"); // not critical enough to stop routing traffic
}
```

A cache being down might mean "slower, but still functional" (`Degraded`) rather than "cannot serve requests at all" (`Unhealthy`) — collapsing this distinction into a single healthy/unhealthy check can cause unnecessary traffic removal for a non-fatal issue.

## Application

Expose separate liveness and readiness endpoints. Keep liveness minimal (process responsiveness only) so transient dependency issues don't trigger unnecessary restarts. Make readiness check the dependencies genuinely required to serve traffic, distinguishing `Degraded` (still functional, but impaired) from `Unhealthy` (cannot serve requests) where that distinction is meaningful.

## Common Mistakes

- Using one combined health check for both liveness and readiness, causing a transient dependency failure to trigger an unnecessary restart instead of just temporarily removing the instance from load-balancer rotation.
- Making a health check too deep/expensive (running full business queries) so the check itself adds meaningful load or latency risk.
- Treating every dependency failure as `Unhealthy` when some (a non-critical cache, a secondary reporting service) should only be `Degraded`.
- Not securing the health-check endpoint at all when it exposes sensitive dependency details in its response, in environments where that endpoint is reachable from outside a trusted network.

## Common Interview Questions

### Basic
- What is a health check, and who typically consumes it?
- What's the difference between liveness and readiness checks?

### Intermediate
- Why is it risky to include a database check in a liveness endpoint?
- What's the difference between `Degraded` and `Unhealthy` health check results?

### Advanced
- How would you design health checks for a service with several dependencies of varying criticality?
- How would you prevent a health-check endpoint from becoming a source of unnecessary load or a security information leak?

### Follow-up Questions
- Should a health-check endpoint require authentication?
- What happens in Kubernetes if a liveness probe fails repeatedly?

### Code Prediction
A liveness probe includes a database connectivity check. The database has a brief 10-second network blip, but the application process itself is otherwise fully healthy. What does Kubernetes do to this pod during those 10 seconds, and how does separating liveness from readiness avoid that outcome?

## Practical Tasks

- Implement separate liveness and readiness endpoints, with readiness checking at least one real dependency.
- Implement a custom health check that distinguishes `Degraded` from `Unhealthy` for a non-critical dependency.
- Explain, for a given set of dependencies (primary database, cache, third-party API), which belong in liveness, readiness, both, or neither.

## Readiness Criteria

Explain the distinction between liveness and readiness precisely, implement custom health checks with appropriate severity levels, and avoid the common mistake of conflating the two check types.

## References

### Microsoft Learn

- [Health checks in ASP.NET Core](https://learn.microsoft.com/aspnet/core/host-and-deploy/health-checks)
