# `IAsyncEnumerable<T>` and Async Streams

## Definition

`IAsyncEnumerable<T>` represents a sequence whose elements are produced asynchronously, one at a time, consumed with `await foreach`. It's the asynchronous counterpart to `IEnumerable<T>` — useful when each element requires an async operation to produce (a paged API call, rows streamed from a database) rather than materializing the whole sequence up front.

```csharp
public async IAsyncEnumerable<Order> GetOrdersAsync()
{
    await foreach (var page in FetchPagesAsync())
        foreach (var order in page)
            yield return order;
}

await foreach (var order in GetOrdersAsync())
{
    Process(order);
}
```

## Alternatives & Trade-offs

Compared to `Task<List<T>>` (load everything, then iterate), `IAsyncEnumerable<T>` starts producing usable results before the entire sequence is available and avoids holding the whole result set in memory at once. The trade-off is a slightly less familiar programming model and the need for `await foreach` at every consumption site, plus careful handling if you need to materialize the whole sequence anyway (`ToListAsync` from an async LINQ provider, or manual accumulation).

## How It Works

### Producing an async stream

```csharp
public async IAsyncEnumerable<int> CountToAsync(int max, [EnumeratorCancellation] CancellationToken cancellationToken = default)
{
    for (int i = 1; i <= max; i++)
    {
        cancellationToken.ThrowIfCancellationRequested();
        await Task.Delay(100, cancellationToken); // simulate async work per element
        yield return i;
    }
}
```

The `[EnumeratorCancellation]` attribute lets a `CancellationToken` passed to `WithCancellation` flow correctly into the generated enumerator.

### Consuming with cancellation

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(1));
await foreach (var value in CountToAsync(100).WithCancellation(cts.Token))
{
    Console.WriteLine(value);
}
```

### Streaming avoids loading everything into memory

```csharp
// Loads the entire result set before returning anything
public async Task<List<Order>> GetAllOrdersAsync() => await LoadAllFromDatabaseAsync();

// Starts yielding orders as they arrive, without waiting for the full set or holding it all in memory
public async IAsyncEnumerable<Order> StreamOrdersAsync()
{
    await foreach (var order in _database.QueryAsync<Order>("SELECT * FROM Orders"))
        yield return order;
}
```

## Application

Use `IAsyncEnumerable<T>` for paged API consumption, streaming large database result sets, or any scenario where producing the next element requires an async operation and you want to start processing before the whole sequence is ready.

## Common Mistakes

- Materializing an `IAsyncEnumerable<T>` into a `List<T>` immediately after producing it, defeating the point of streaming.
- Forgetting `[EnumeratorCancellation]`, so a `CancellationToken` passed via `WithCancellation` never actually reaches the generator method's internal checks.
- Assuming `await foreach` behaves identically to `foreach` performance-wise — each iteration involves an async continuation, which has overhead compared to synchronous iteration.
- Using `IAsyncEnumerable<T>` for a small, fully in-memory sequence where a simple `Task<List<T>>` would be simpler and no less efficient.

## Common Interview Questions

### Basic
- What is `IAsyncEnumerable<T>`, and how do you consume it?
- How does `IAsyncEnumerable<T>` differ from `IEnumerable<T>`?

### Intermediate
- Why would you choose `IAsyncEnumerable<T>` over returning `Task<List<T>>`?
- What does `[EnumeratorCancellation]` do, and why is it needed?

### Advanced
- How does `IAsyncEnumerable<T>` help with memory usage when streaming large result sets, compared to full materialization?
- How would you implement cancellation-aware paging over a remote API using `IAsyncEnumerable<T>`?

### Follow-up Questions
- Can you use LINQ operators directly on an `IAsyncEnumerable<T>`?
- Does `await foreach` block the calling thread between elements?

### Code Prediction
```csharp
async IAsyncEnumerable<int> GenerateAsync()
{
    for (int i = 0; i < 3; i++)
    {
        Console.WriteLine($"Producing {i}");
        await Task.Delay(100);
        yield return i;
    }
}

await foreach (var value in GenerateAsync())
{
    Console.WriteLine($"Consuming {value}");
}
```
What is the interleaving of "Producing" and "Consuming" lines? Is every value produced before any is consumed, or are they interleaved?

## Practical Tasks

- Convert a method that loads an entire paged API result into a `List<T>` before returning it into an `IAsyncEnumerable<T>`-based streaming version.
- Add proper cancellation support to an async stream generator using `[EnumeratorCancellation]`.
- Compare memory usage between a fully-materialized large result set and a streamed `IAsyncEnumerable<T>` version.

## Readiness Criteria

Explain when `IAsyncEnumerable<T>` is preferable to `Task<List<T>>`, implement cancellation-aware async streams correctly, and reason about the interleaving of production and consumption in `await foreach`.

## References

### Microsoft Learn

- [Generate and consume async streams](https://learn.microsoft.com/dotnet/csharp/asynchronous-programming/generate-consume-asynchronous-stream)
- [IAsyncEnumerable<T> interface](https://learn.microsoft.com/dotnet/api/system.collections.generic.iasyncenumerable-1)
