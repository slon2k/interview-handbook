# Async Disposal

## Definition

`IAsyncDisposable` provides an asynchronous cleanup path (`DisposeAsync`) for resources whose teardown involves I/O — flushing a stream, closing a network connection — so that cleanup doesn't block a thread the way synchronous `Dispose` would. `await using` is the asynchronous counterpart to `using`.

```csharp
public sealed class RemoteConnection : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        await FlushAsync();
        await CloseSocketAsync();
    }
}

await using var connection = new RemoteConnection();
```

## Alternatives & Trade-offs

Synchronous `Dispose` is simpler and sufficient when cleanup is fast and CPU-only (releasing a handle, clearing a buffer). `IAsyncDisposable` is worth the extra interface when cleanup genuinely does I/O — otherwise it just adds ceremony without meaningfully avoiding any thread-blocking, since there's nothing slow to make asynchronous.

## How It Works

### Implementing both, when relevant

```csharp
public sealed class HybridResource : IDisposable, IAsyncDisposable
{
    public void Dispose() => DisposeAsync().AsTask().GetAwaiter().GetResult(); // fallback, not ideal but sometimes necessary
    public async ValueTask DisposeAsync()
    {
        await FlushAsync();
        GC.SuppressFinalize(this);
    }
}
```

Most types only need to implement one or the other — implement both only if callers might reasonably use either pattern and can't be migrated to `await using`.

### `await using` at the end of an async method

```csharp
public async Task ProcessAsync()
{
    await using var connection = await OpenConnectionAsync();
    await connection.SendAsync(data);
} // connection.DisposeAsync() is awaited automatically here, even if an exception is thrown above
```

### Async disposal in a collection of resources

```csharp
await using var resource1 = new RemoteConnection();
await using var resource2 = new RemoteConnection();
// disposed in reverse order automatically, each one awaited
```

## Application

Implement `IAsyncDisposable` for types whose cleanup involves network calls, flushing buffered async writes, or any other I/O-bound teardown step. Plain, fast, synchronous cleanup should stick with `IDisposable`.

## Common Mistakes

- Implementing `IAsyncDisposable` for a type whose cleanup is actually synchronous and fast, adding unnecessary API surface.
- Calling `.Dispose()` on a type that only implements `IAsyncDisposable`, or blocking on `DisposeAsync().AsTask().Wait()` synchronously, defeating the purpose of the async cleanup path.
- Forgetting `await` before `using` (`using var x = ...` instead of `await using var x = ...`), silently falling back to synchronous disposal semantics if the type also implements `IDisposable`.
- Not handling exceptions during `DisposeAsync` itself, which can mask the original exception that triggered disposal.

## Common Interview Questions

### Basic
- What problem does `IAsyncDisposable` solve that `IDisposable` doesn't?
- What is the syntax for asynchronous disposal?

### Intermediate
- When would a type implement both `IDisposable` and `IAsyncDisposable`?
- What happens if you use `using` (not `await using`) on a type that only implements `IAsyncDisposable`?

### Advanced
- How would you safely provide a synchronous `Dispose` fallback for a type whose primary cleanup path is asynchronous?
- In what order are multiple `await using` resources disposed, and does an exception during disposal affect that order?

### Follow-up Questions
- Is `DisposeAsync` guaranteed to run if an exception is thrown inside the `await using` block?
- Should `DisposeAsync` itself ever throw?

### Code Prediction
```csharp
public sealed class Resource : IAsyncDisposable
{
    public async ValueTask DisposeAsync()
    {
        await Task.Delay(100);
        Console.WriteLine("Disposed");
    }
}

async Task RunAsync()
{
    await using var resource = new Resource();
    throw new InvalidOperationException();
}
```
Does "Disposed" get printed before the `InvalidOperationException` propagates out of `RunAsync`?

## Practical Tasks

- Implement `IAsyncDisposable` for a type that closes a simulated network connection, and use it with `await using`.
- Demonstrate disposal ordering for multiple `await using` declarations in one method.
- Identify a type in a hypothetical codebase whose synchronous `Dispose` does blocking I/O, and convert it to `IAsyncDisposable`.

## Readiness Criteria

Explain when `IAsyncDisposable` is worth implementing over `IDisposable`, use `await using` correctly, and reason about disposal ordering and exception interaction during async cleanup.

## References

### Microsoft Learn

- [Implement a DisposeAsync method](https://learn.microsoft.com/dotnet/standard/garbage-collection/implementing-disposeasync)
- [IAsyncDisposable interface](https://learn.microsoft.com/dotnet/api/system.iasyncdisposable)
