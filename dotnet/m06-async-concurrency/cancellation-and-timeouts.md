# Cancellation Tokens and Timeouts

## Definition

`CancellationToken` is a cooperative cancellation signal: a caller creates a `CancellationTokenSource`, passes its `Token` into an async operation, and the operation periodically checks (or is told via a callback) whether cancellation was requested, stopping voluntarily if so. Nothing forces a running operation to stop — cancellation only works if the code checks for it.

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(5)); // auto-cancels after 5s
await ProcessAsync(cts.Token);
```

## Alternatives & Trade-offs

Cooperative cancellation via `CancellationToken` is the standard .NET pattern and integrates with most async APIs. Forcibly aborting a thread (`Thread.Abort`, removed in modern .NET) is unsafe and no longer available — cooperative cancellation is the only supported approach, which means every layer of an operation must actually observe the token for cancellation to have any effect.

## How It Works

### Passing and observing the token

```csharp
public async Task ProcessItemsAsync(IEnumerable<Item> items, CancellationToken cancellationToken)
{
    foreach (var item in items)
    {
        cancellationToken.ThrowIfCancellationRequested(); // cooperative check
        await ProcessAsync(item, cancellationToken);       // propagate the token downstream
    }
}
```

If `ProcessItemsAsync` never checks the token, cancelling the source has no effect on it at all — cancellation must be threaded through every level that can meaningfully stop.

### Timeout via `CancellationTokenSource`

```csharp
using var cts = new CancellationTokenSource(TimeSpan.FromSeconds(10));
try
{
    await httpClient.GetStringAsync(url, cts.Token);
}
catch (OperationCanceledException)
{
    // Either the caller cancelled, or the timeout elapsed — inspect cts.IsCancellationRequested if it matters which
}
```

### Combining multiple cancellation sources

```csharp
using var timeoutCts = new CancellationTokenSource(TimeSpan.FromSeconds(30));
using var linkedCts = CancellationTokenSource.CreateLinkedTokenSource(timeoutCts.Token, userCancellationToken);
await ProcessAsync(linkedCts.Token); // cancels if either the timeout OR the user's token fires
```

### Registering a cleanup callback

```csharp
cancellationToken.Register(() => Console.WriteLine("Cancellation requested — cleaning up"));
```

## Application

Accept a `CancellationToken` parameter on any public async method that does meaningful work, defaulting to `CancellationToken.None` if omitted. Use `CancellationTokenSource(TimeSpan)` for simple timeouts, and `CreateLinkedTokenSource` when an operation needs to respect both an external cancellation request and an internal timeout.

## Common Mistakes

- Accepting a `CancellationToken` parameter but never actually checking it or passing it to downstream calls — cancellation support that does nothing.
- Catching `OperationCanceledException` and treating it as a generic failure instead of a deliberate, expected outcome.
- Not disposing `CancellationTokenSource` instances (they implement `IDisposable`), especially ones created with a timeout.
- Assuming cancellation is immediate — a long-running synchronous block between cancellation checks will keep running until it reaches the next check.

## Common Interview Questions

### Basic
- What is a `CancellationToken`, and how is it different from forcibly stopping a thread?
- How do you implement a timeout using `CancellationTokenSource`?

### Intermediate
- What happens if a method accepts a `CancellationToken` but never checks it?
- What exception is thrown when a cancellation is honored, and how should it typically be handled?

### Advanced
- How would you distinguish between a timeout-triggered cancellation and a user-triggered cancellation when both use a linked token?
- How does `CancellationToken.Register` support cleanup logic for non-cooperative resources (e.g., unblocking a blocking call)?

### Follow-up Questions
- Is cancellation guaranteed to be immediate?
- Can a single `CancellationToken` be observed by multiple independent operations at once?

### Code Prediction
```csharp
var cts = new CancellationTokenSource();
cts.Cancel();
try
{
    await Task.Delay(1000, cts.Token);
}
catch (OperationCanceledException)
{
    Console.WriteLine("Cancelled");
}
```
Does `Task.Delay` here run for close to 1000ms, or does it stop almost immediately? Why?

## Practical Tasks

- Add proper `CancellationToken` support to a method that currently ignores it, threading it through all downstream async calls.
- Implement a combined timeout-and-user-cancellation flow using `CreateLinkedTokenSource`.
- Reproduce a scenario where cancellation appears to "not work" because a downstream call didn't accept or check the token, then fix it.

## Readiness Criteria

Explain cooperative cancellation accurately, correctly implement timeout and combined-cancellation patterns, and identify cancellation support that's present in a method signature but not actually wired through.

## References

### Microsoft Learn

- [Cancellation in managed threads](https://learn.microsoft.com/dotnet/standard/threading/cancellation-in-managed-threads)
- [CancellationTokenSource class](https://learn.microsoft.com/dotnet/api/system.threading.cancellationtokensource)
