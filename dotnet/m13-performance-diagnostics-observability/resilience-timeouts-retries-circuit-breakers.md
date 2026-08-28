# Basic Resilience: Timeouts, Retries, and Circuit Breakers

## Definition

Resilience patterns let a system tolerate a dependency being slow or unavailable without cascading that failure into a larger outage. A **timeout** bounds how long to wait for a response. A **retry** re-attempts a failed operation, typically with backoff. A **circuit breaker** stops sending requests to a dependency that's clearly failing, giving it time to recover instead of continuing to hammer it — and protecting the caller from piling up waiting requests.

```csharp
builder.Services.AddHttpClient<PaymentGatewayClient>()
    .AddResilienceHandler("payments", pipeline =>
    {
        pipeline.AddTimeout(TimeSpan.FromSeconds(5));
        pipeline.AddRetry(new HttpRetryStrategyOptions { MaxRetryAttempts = 3, BackoffType = DelayBackoffType.Exponential });
        pipeline.AddCircuitBreaker(new HttpCircuitBreakerStrategyOptions());
    });
```

## Alternatives & Trade-offs

Making a call with no timeout risks a slow dependency holding resources indefinitely, eventually exhausting threads/connections and taking down the caller too (Module 6's thread-pool-starvation risk, and this module's connection-pool topic). Adding a timeout without a retry means a single transient blip fails outright, when a quick retry might have succeeded. Adding retries without a circuit breaker means a genuinely down dependency gets hammered with repeated retries from every caller, potentially prolonging its outage rather than letting it recover.

## How It Works

### Timeouts — the first, non-negotiable layer

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5));
var response = await httpClient.GetAsync(url, cts.Token); // never wait indefinitely for any external call
```

Every call to an external dependency should have a bounded maximum wait — this is the direct application of Module 6's cancellation/timeout patterns specifically as a resilience concern, not just a cooperative-cancellation nicety.

### Retries with backoff — for genuinely transient failures

```csharp
pipeline.AddRetry(new HttpRetryStrategyOptions
{
    MaxRetryAttempts = 3,
    BackoffType = DelayBackoffType.Exponential, // wait longer between each successive retry
    ShouldHandle = args => ValueTask.FromResult(args.Outcome.Result?.StatusCode == HttpStatusCode.ServiceUnavailable)
});
```

Retrying only for genuinely retryable failures (`503`, network timeouts) — not for `400`-class errors, where the request itself is wrong and retrying identically will just fail again (the same distinction covered in Module 7's status-codes topic) — and backing off exponentially avoids hammering an already-struggling dependency with immediate repeated attempts.

### Circuit breakers — stopping the bleeding when a dependency is clearly down

```
Closed:    normal operation, requests pass through
Open:      failure threshold exceeded — requests fail FAST without even attempting the call,
           giving the dependency room to recover instead of continuing to be hammered
Half-Open: after a cooldown, a limited number of test requests are allowed through to check
           if the dependency has recovered, before fully closing the circuit again
```

```csharp
pipeline.AddCircuitBreaker(new HttpCircuitBreakerStrategyOptions
{
    FailureRatio = 0.5,             // open the circuit if 50% of recent requests failed
    SamplingDuration = TimeSpan.FromSeconds(30),
    MinimumThroughput = 10,
    BreakDuration = TimeSpan.FromSeconds(15)
});
```

Once open, calls fail immediately (without even attempting the network call) — which is faster for the caller than waiting out a timeout on every single request, and reduces load on an already-struggling dependency.

### Combining all three — and idempotency's role (Module 7) in making retries safe

```
A retry re-executes the same logical operation. If that operation isn't idempotent (Module 7),
a retry after a request that actually succeeded — but whose response was lost due to a network
blip — can cause duplicate side effects (a double charge). This is exactly why idempotency keys
matter for making retries safe on non-idempotent operations, not just a nice-to-have.
```

## Application

Set explicit timeouts on every call to an external dependency. Add retries with backoff specifically for transient, retryable failure types, and pair retries on non-idempotent operations with idempotency keys (Module 7). Add a circuit breaker for dependencies where sustained failure is possible, to fail fast and give the dependency room to recover rather than continuing to pile on load.

## Common Mistakes

- Making external calls with no timeout at all, risking resource exhaustion if the dependency hangs.
- Retrying non-idempotent operations without an idempotency key, risking duplicate side effects from a retry after a response was lost but the original request actually succeeded.
- Retrying `4xx` client errors, which will simply fail again identically, wasting time and load instead of failing fast.
- Not implementing a circuit breaker for a dependency prone to extended outages, continuing to send load to (and potentially prolong the outage of) a system that's clearly down.

## Common Interview Questions

### Basic
- What do timeouts, retries, and circuit breakers each protect against?
- What are the three states of a circuit breaker?

### Intermediate
- Why is it risky to retry a non-idempotent operation without an idempotency key?
- Why should retries generally not be applied to `4xx` client errors?

### Advanced
- How would you design a resilience pipeline combining timeouts, retries, and a circuit breaker for a call to a genuinely unreliable external dependency?
- How does a circuit breaker's "fail fast" behavior in the Open state actually help the failing dependency recover, compared to continuing to send (and time out on) every request?

### Follow-up Questions
- Does exponential backoff between retries eliminate the risk of overwhelming a struggling dependency?
- Can a circuit breaker be scoped per-dependency, or does it apply globally to all outbound calls?

### Code Prediction
A payment API call has a 3-retry policy with no idempotency key, and the call actually succeeds on the server side but the response is lost due to a network blip before the client receives it. What happens when the client's resilience pipeline retries?

## Practical Tasks

- Configure a resilience pipeline with timeout, retry-with-backoff, and circuit breaker for a typed `HttpClient`, and verify each layer's behavior independently.
- Combine a retry policy with an idempotency key for a non-idempotent payment-style operation.
- Simulate a dependency outage and observe a configured circuit breaker transition through Closed, Open, and Half-Open states.

## Readiness Criteria

Configure timeouts, retries, and circuit breakers correctly and in combination, pair retries with idempotency where needed, and explain how each layer protects against a distinct failure mode.

## References

### Microsoft Learn

- [Build resilient HTTP apps](https://learn.microsoft.com/dotnet/core/resilience/http-resilience)
- [Polly resilience strategies](https://learn.microsoft.com/dotnet/core/resilience/)
