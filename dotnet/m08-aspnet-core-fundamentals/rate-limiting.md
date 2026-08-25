# Rate Limiting

## Definition

Rate limiting restricts how many requests a client can make within a given time window, protecting a service from being overwhelmed (accidentally or maliciously) and giving fair capacity across many clients. ASP.NET Core has built-in rate-limiting middleware (`Microsoft.AspNetCore.RateLimiting`, .NET 7+) supporting several algorithms out of the box.

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.AddFixedWindowLimiter("fixed", limiterOptions =>
    {
        limiterOptions.PermitLimit = 100;
        limiterOptions.Window = TimeSpan.FromMinutes(1);
    });
});

app.UseRateLimiter();
app.MapGet("/orders", () => Results.Ok()).RequireRateLimiting("fixed");
```

## Alternatives & Trade-offs

Rate limiting at the application layer (as here) is simple to configure per-endpoint and can key limits by user/API key rather than just IP, but adds load to the app itself to track and enforce limits. Rate limiting at the infrastructure layer (API gateway, reverse proxy, CDN) offloads that work before it reaches the app, but is often coarser-grained and less aware of application-specific concepts like "which user" or "which subscription tier." Many production systems use both — a coarse infrastructure-level limit as a first line of defense, and finer-grained application-level limits for specific expensive or abusable endpoints.

## How It Works

### The built-in algorithms

```
Fixed window   — N requests per fixed time window (e.g., 100/minute); simple, but allows bursts at window boundaries
Sliding window — smooths out the fixed-window boundary-burst problem by tracking sub-windows
Token bucket   — tokens refill at a steady rate; allows short bursts up to the bucket size, then throttles
Concurrency    — limits how many requests can be in flight at once, rather than counting over time
```

```csharp
options.AddTokenBucketLimiter("uploads", limiterOptions =>
{
    limiterOptions.TokenLimit = 10;
    limiterOptions.TokensPerPeriod = 2;
    limiterOptions.ReplenishmentPeriod = TimeSpan.FromSeconds(1);
});
```

### Partitioning limits per client, not globally

```csharp
options.AddPolicy("per-user", context =>
    RateLimitPartition.GetFixedWindowLimiter(
        partitionKey: context.User.Identity?.Name ?? context.Connection.RemoteIpAddress?.ToString() ?? "anonymous",
        factory: _ => new FixedWindowRateLimiterOptions { PermitLimit = 50, Window = TimeSpan.FromMinutes(1) }));
```

A global limit applies to *all* traffic combined; a partitioned limit (keyed by user ID, API key, or IP) gives each client their own independent quota — almost always what's actually intended.

### Returning a proper `429` with `Retry-After`

```csharp
builder.Services.AddRateLimiter(options =>
{
    options.OnRejected = async (context, cancellationToken) =>
    {
        context.HttpContext.Response.StatusCode = StatusCodes.Status429TooManyRequests;
        if (context.Lease.TryGetMetadata(MetadataName.RetryAfter, out var retryAfter))
            context.HttpContext.Response.Headers.RetryAfter = ((int)retryAfter.TotalSeconds).ToString();
    };
});
```

## Application

Apply rate limiting to public APIs, authentication endpoints (to slow brute-force attempts), and any expensive or abusable operation (file uploads, search, report generation). Partition limits by client identity rather than applying one global limit, and always return `429` with a `Retry-After` header so well-behaved clients can back off correctly.

## Common Mistakes

- Applying a single global rate limit instead of partitioning by client, letting one abusive client's traffic exhaust the quota for everyone else.
- Choosing fixed-window limiting for a scenario sensitive to boundary bursts (a client sending double the intended rate right at the window edge) without considering sliding window or token bucket instead.
- Not returning a `Retry-After` header with `429` responses, leaving well-behaved clients to guess how long to wait before retrying.
- Rate-limiting only at the infrastructure layer with no application-level awareness of user identity or subscription tier, when different clients genuinely need different limits.

## Common Interview Questions

### Basic
- What is rate limiting, and what status code should a rate-limited request return?
- What rate-limiting algorithms does ASP.NET Core support out of the box?

### Intermediate
- Why does a fixed-window limiter allow bursts at window boundaries, and how does a sliding window or token bucket address that?
- Why should rate limits typically be partitioned per client rather than applied globally?

### Advanced
- How would you design rate limiting that applies different limits based on a user's subscription tier?
- How would you combine infrastructure-level and application-level rate limiting in one system, and what does each layer protect against that the other doesn't?

### Follow-up Questions
- Does rate limiting protect against distributed attacks from many different IPs?
- What should a well-behaved client do when it receives a `429` with a `Retry-After` header?

### Code Prediction
A fixed-window limiter allows 100 requests per minute, with no partitioning by client. If one client sends 100 requests in the first second of the window, what happens to every other client's requests for the rest of that minute?

## Practical Tasks

- Configure a partitioned, per-user rate limiter and verify one user's traffic doesn't affect another's quota.
- Compare fixed-window and token-bucket behavior under a bursty traffic pattern.
- Implement a proper `429` response with a `Retry-After` header for a rejected request.

## Readiness Criteria

Explain the trade-offs between rate-limiting algorithms, correctly partition limits by client, and implement a compliant `429`/`Retry-After` rejection response.

## References

### Microsoft Learn

- [Rate limiting middleware in ASP.NET Core](https://learn.microsoft.com/aspnet/core/performance/rate-limit)
