# HttpClientFactory

## Definition

`IHttpClientFactory` centrally creates and manages `HttpClient` instances, solving two problems that arise from managing `HttpClient` manually: **socket exhaustion** from creating too many short-lived clients, and **stale DNS** from a single long-lived client never picking up DNS changes.

```csharp
builder.Services.AddHttpClient<IPaymentGatewayClient, PaymentGatewayClient>(client =>
{
    client.BaseAddress = new Uri("https://api.payments.example.com");
    client.Timeout = TimeSpan.FromSeconds(10);
});
```

## Alternatives & Trade-offs

`new HttpClient()` per request is simple but exhausts sockets under load, since each instance holds its own connection pool that isn't released immediately when disposed (TCP `TIME_WAIT`). A single `static readonly HttpClient` avoids socket exhaustion but never refreshes DNS resolution, so it can keep sending requests to a stale IP after a DNS change (e.g., during a failover). `IHttpClientFactory` solves both: it manages an underlying pool of `HttpMessageHandler`s with a rotation policy, giving you the convenience of "just ask for a client" without either failure mode.

## How It Works

### The two problems `HttpClientFactory` solves

```csharp
// Problem 1: socket exhaustion — creating a new HttpClient per call under load
public async Task CallApiAsync()
{
    using var client = new HttpClient(); // each one holds its own handler/connection pool
    await client.GetAsync("https://api.example.com");
} // sockets can remain in TIME_WAIT, accumulating under high request volume

// Problem 2: stale DNS — one long-lived client, but DNS changes are never observed
private static readonly HttpClient _client = new(); // never refreshes DNS resolution for its connections
```

`IHttpClientFactory` manages handler lifetime internally (recycling underlying handlers periodically) so DNS changes are eventually picked up, while still pooling connections to avoid exhaustion.

### Named and typed clients

```csharp
// Named client
builder.Services.AddHttpClient("payments", client => client.BaseAddress = new Uri("https://api.payments.example.com"));
var client = httpClientFactory.CreateClient("payments");

// Typed client — a dedicated class wrapping HttpClient, resolved via DI like any other service
public class PaymentGatewayClient
{
    private readonly HttpClient _client;
    public PaymentGatewayClient(HttpClient client) => _client = client; // pre-configured by AddHttpClient<>
    public Task<PaymentResult> ChargeAsync(decimal amount) => _client.PostAsJsonAsync("/charge", new { amount }).ContinueWith(_ => new PaymentResult());
}

builder.Services.AddHttpClient<PaymentGatewayClient>(client => client.BaseAddress = new Uri("https://api.payments.example.com"));
```

Typed clients are generally preferred — they give the HTTP call a proper class and testable seam (an interface can be introduced around it), rather than a loosely-typed named string.

### Adding resilience with delegating handlers

```csharp
builder.Services.AddHttpClient<PaymentGatewayClient>()
    .AddResilienceHandler("payments-pipeline", pipeline =>
    {
        pipeline.AddRetry(new HttpRetryStrategyOptions { MaxRetryAttempts = 3 });
        pipeline.AddTimeout(TimeSpan.FromSeconds(5));
    });
```

`IHttpClientFactory` integrates cleanly with resilience pipelines (via `Microsoft.Extensions.Http.Resilience`), attaching retry/timeout/circuit-breaker policies to a specific named or typed client without touching the calling code.

## Application

Always use `IHttpClientFactory` (via named or typed clients) instead of manually managing `HttpClient` instances. Prefer typed clients for outbound calls to a specific external service, since they give a proper testable class boundary. Attach resilience policies (retry, timeout, circuit breaker) at the client-registration level rather than scattering retry loops through calling code.

## Common Mistakes

- Creating a new `HttpClient` per request/call (`using var client = new HttpClient()`), risking socket exhaustion under load.
- Using a single `static readonly HttpClient` and never revisiting DNS-staleness risk for services behind a load balancer or DNS-based failover.
- Not disposing the `HttpResponseMessage` (though `HttpClient` itself no longer needs manual disposal when created via the factory) leading to unnecessarily held resources for large responses.
- Adding retry logic manually inside every calling method instead of centralizing it as a resilience handler on the client registration.

## Common Interview Questions

### Basic
- What two problems does `IHttpClientFactory` solve?
- What's the difference between a named client and a typed client?

### Intermediate
- Why does creating a new `HttpClient` per request risk socket exhaustion?
- Why does a single long-lived `HttpClient` risk stale DNS resolution?

### Advanced
- How does `IHttpClientFactory` manage handler lifetime internally to avoid both socket exhaustion and stale DNS?
- How would you attach a retry-with-backoff and circuit-breaker policy to all calls made through a specific typed client?

### Follow-up Questions
- Does `IHttpClientFactory` eliminate the need to dispose `HttpResponseMessage` objects?
- Can a typed client be unit tested without making real HTTP calls?

### Code Prediction
A service creates `new HttpClient()` inside a hot-path method called thousands of times per minute under load. What symptom would eventually appear at the OS/network level, and how would switching to `IHttpClientFactory`'s typed client pattern resolve it?

## Practical Tasks

- Convert a service manually managing `HttpClient` into a typed client registered via `AddHttpClient<T>`.
- Add a retry-and-timeout resilience pipeline to a typed client's registration.
- Explain, for a code review, why a `static readonly HttpClient` field is risky for a service behind a load balancer with DNS-based failover.

## Readiness Criteria

Explain both failure modes `IHttpClientFactory` solves, correctly use named and typed clients, and attach resilience policies at the client-registration level rather than in calling code.

## References

### Microsoft Learn

- [Make HTTP requests using IHttpClientFactory](https://learn.microsoft.com/dotnet/core/extensions/httpclient-factory)
- [Build resilient HTTP apps](https://learn.microsoft.com/dotnet/core/resilience/http-resilience)
