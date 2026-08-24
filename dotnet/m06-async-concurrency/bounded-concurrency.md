# Bounded Concurrency

## Definition

Bounded concurrency means deliberately limiting how many operations run at the same time — even when more could technically start at once — to avoid overwhelming a downstream dependency (a database, an external API, a rate-limited service) or exhausting local resources (memory, connections, threads).

```csharp
var semaphore = new SemaphoreSlim(initialCount: 5); // at most 5 concurrent operations
```

## Alternatives & Trade-offs

Unbounded concurrency (`Task.WhenAll` over an unfiltered list) maximizes throughput when the downstream system can handle it, but risks overwhelming a rate-limited API, exhausting a connection pool, or triggering throttling/errors. Bounding concurrency trades some potential throughput for predictable, safe load on dependencies — the right bound is usually determined by the downstream system's actual capacity, not an arbitrary guess.

## How It Works

### Unbounded — risky against a rate-limited API

```csharp
var tasks = urls.Select(url => httpClient.GetStringAsync(url));
var results = await Task.WhenAll(tasks); // fires all 500 requests at once
```

### Bounded with `SemaphoreSlim`

```csharp
var semaphore = new SemaphoreSlim(10); // at most 10 concurrent requests

async Task<string> FetchAsync(string url)
{
    await semaphore.WaitAsync();
    try { return await httpClient.GetStringAsync(url); }
    finally { semaphore.Release(); }
}

var results = await Task.WhenAll(urls.Select(FetchAsync));
```

Every task still starts (they're all created up front), but only 10 are ever actually inside the semaphore-protected section running the real work at any given moment; the rest wait their turn.

### Bounded with `Parallel.ForEachAsync` (.NET 6+)

```csharp
await Parallel.ForEachAsync(urls, new ParallelOptions { MaxDegreeOfParallelism = 10 }, async (url, ct) =>
{
    var result = await httpClient.GetStringAsync(url, ct);
    Process(result);
});
```

This achieves the same bounding with less manual bookkeeping than the semaphore pattern, when the workload fits its shape (no need to collect individual results as a list).

### Choosing the bound

The right `MaxDegreeOfParallelism`/semaphore count depends on the downstream system: a rate-limited third-party API might cap you at a documented requests-per-second limit; a database connection pool might dictate a maximum based on pool size; pure local CPU work should generally bound to `Environment.ProcessorCount`.

## Application

Apply bounded concurrency whenever fanning out many concurrent operations against a shared, capacity-limited dependency — batch processing against an external API, parallel database queries against a connection-pooled database, or any background job processing a large queue of items.

## Common Mistakes

- Using `Task.WhenAll` over an unbounded collection without considering the downstream system's actual capacity.
- Choosing an arbitrary concurrency limit without checking the documented or empirical capacity of the dependency being called.
- Forgetting to release a `SemaphoreSlim` in a `finally` block, which permanently shrinks the effective concurrency limit after any exception.
- Confusing "bounded concurrency" with "sequential" — the goal is a controlled degree of parallelism, not none at all.

## Common Interview Questions

### Basic
- What is bounded concurrency, and why would you want it?
- How can `SemaphoreSlim` be used to limit concurrent operations?

### Intermediate
- What's the risk of firing off `Task.WhenAll` over a large, unfiltered list of async operations against an external API?
- How does `Parallel.ForEachAsync` simplify bounded concurrency compared to a manual semaphore?

### Advanced
- How would you determine the right concurrency limit for a batch job calling a third-party API with an undocumented rate limit?
- How would you design a system that adapts its concurrency limit dynamically based on observed error rates (e.g., backing off on 429 responses)?

### Follow-up Questions
- Does bounding concurrency guarantee requests happen in order?
- What's the difference between bounding concurrency and adding a delay between calls?

### Code Prediction
Given the `SemaphoreSlim(10)` example fetching 500 URLs, are all 500 `Task` objects created immediately, or only 10 at a time? What actually gets delayed by the semaphore?

## Practical Tasks

- Convert an unbounded `Task.WhenAll` batch operation into a bounded version using `SemaphoreSlim`, and verify the maximum concurrent operations empirically.
- Reimplement the same scenario using `Parallel.ForEachAsync` and compare the code.
- Design (in discussion or code) a backoff strategy for a batch job that starts receiving rate-limit errors from a downstream API.

## Readiness Criteria

Explain why unbounded concurrency can be harmful, implement bounded concurrency correctly with both `SemaphoreSlim` and `Parallel.ForEachAsync`, and reason about how to choose an appropriate concurrency limit.

## References

### Microsoft Learn

- [SemaphoreSlim class](https://learn.microsoft.com/dotnet/api/system.threading.semaphoreslim)
- [Parallel.ForEachAsync method](https://learn.microsoft.com/dotnet/api/system.threading.tasks.parallel.foreachasync)
