# ASP.NET Core Hosting Model and Kestrel

## Definition

ASP.NET Core apps run inside a generic host (`WebApplication`/`IHost`) that wires up configuration, dependency injection, and logging, and hosts a web server. **Kestrel** is the built-in, cross-platform web server that actually accepts HTTP connections; in production it's typically placed behind a **reverse proxy** (IIS, Nginx, YARP, a cloud load balancer) that handles TLS termination, static file serving, and request buffering before forwarding to Kestrel.

```csharp
var builder = WebApplication.CreateBuilder(args); // sets up host, DI, configuration, logging
builder.Services.AddControllers();

var app = builder.Build();
app.MapControllers();
app.Run(); // starts Kestrel and begins listening
```

## Alternatives & Trade-offs

Kestrel alone is fast and fully capable of serving requests directly, but a reverse proxy in front of it adds capabilities Kestrel intentionally doesn't own itself: TLS certificate management at scale, request buffering against slow clients, static file caching, and routing across multiple backend instances. Running without a reverse proxy is fine for internal services or simple deployments; production internet-facing apps almost always place one in front.

## How It Works

### The generic host

```csharp
var builder = WebApplication.CreateBuilder(args);
// builder.Services  -> DI container registration
// builder.Configuration -> layered configuration (appsettings.json, env vars, etc.)
// builder.Logging -> logging providers
var app = builder.Build(); // builds the DI container and the request pipeline
```

Everything — DI, configuration, logging, the web server itself — is unified under one host object, replacing the separate `Startup.cs`/`Program.cs` split used in older ASP.NET Core versions.

### Kestrel behind a reverse proxy

```
Internet -> [Nginx/IIS/YARP: TLS termination, static files] -> [Kestrel: application logic]
```

```csharp
// Forwarded headers middleware trusts X-Forwarded-For/Proto from a known reverse proxy,
// so the app sees the real client IP and scheme instead of the proxy's
app.UseForwardedHeaders(new ForwardedHeadersOptions
{
    ForwardedHeaders = ForwardedHeaders.XForwardedFor | ForwardedHeaders.XForwardedProto
});
```

Without this, code that reads `HttpContext.Connection.RemoteIpAddress` or checks `Request.IsHttps` would see the proxy's values, not the original client's.

### Kestrel configuration

```csharp
builder.WebHost.ConfigureKestrel(options =>
{
    options.Limits.MaxConcurrentConnections = 1000;
    options.Limits.MaxRequestBodySize = 10 * 1024 * 1024; // 10 MB
});
```

## Application

Use Kestrel directly for internal services, containerized workloads behind a cloud load balancer, or development. Add a reverse proxy for internet-facing production apps needing TLS certificate management, static file offloading, or routing across multiple instances — and always configure forwarded headers when one is present, or client IP/scheme information will be wrong throughout the app.

## Common Mistakes

- Forgetting to configure forwarded headers behind a reverse proxy, causing incorrect client IP logging, broken HTTPS redirection, or broken absolute URL generation.
- Assuming Kestrel alone provides the same production hardening (connection limits, request timeouts, TLS management at scale) that a dedicated reverse proxy is designed for.
- Not setting `MaxRequestBodySize` appropriately, leaving an endpoint vulnerable to oversized-payload abuse or blocking legitimate large uploads.

## Common Interview Questions

### Basic
- What is Kestrel, and why is a reverse proxy commonly placed in front of it?
- What does the ASP.NET Core generic host unify?

### Intermediate
- What problem does forwarded-headers middleware solve?
- What responsibilities does a reverse proxy typically take on that Kestrel doesn't handle itself?

### Advanced
- What goes wrong if forwarded headers are trusted from an untrusted source, and how do you configure `ForwardedHeadersOptions` safely?
- How would you tune Kestrel's connection and request-size limits for a specific workload?

### Follow-up Questions
- Can Kestrel serve HTTPS directly without a reverse proxy?
- Is a reverse proxy required for containerized deployments behind a cloud load balancer?

### Code Prediction
An app behind an Nginx reverse proxy checks `Request.IsHttps` to decide whether to redirect to HTTPS, but `UseForwardedHeaders` was never configured. What value does `Request.IsHttps` see if the original client connection was HTTPS but Nginx forwards to Kestrel over plain HTTP internally?

## Practical Tasks

- Configure forwarded-headers middleware for an app behind a simulated reverse proxy and verify the correct client IP is observed.
- Set Kestrel's `MaxRequestBodySize` and verify an oversized request is rejected.
- Explain, for a given deployment (bare Kestrel vs. behind Nginx vs. behind a cloud load balancer), what a reverse proxy would add.

## Readiness Criteria

Explain the generic host and Kestrel's role, correctly configure forwarded headers behind a reverse proxy, and reason about when a reverse proxy is necessary versus optional.

## References

### Microsoft Learn

- [ASP.NET Core web server implementations](https://learn.microsoft.com/aspnet/core/fundamentals/servers/)
- [Kestrel web server implementation](https://learn.microsoft.com/aspnet/core/fundamentals/servers/kestrel)
- [Configure ASP.NET Core to work with proxy servers](https://learn.microsoft.com/aspnet/core/host-and-deploy/proxy-load-balancer)
