# `Task.WhenAll`, `Task.WhenAny`, and Sequential vs. Concurrent Execution

## Definition

`Task.WhenAll` returns a task that completes when every supplied task completes; `Task.WhenAny` returns a task that completes when the first supplied task completes. Both let you shape whether independent async operations run one after another (sequential) or all at once (concurrent).

```csharp
Task<string> a = client.GetStringAsync(url1);
Task<string> b = client.GetStringAsync(url2);
string[] results = await Task.WhenAll(a, b); // both requests were in flight concurrently
```

## Alternatives & Trade-offs

Awaiting tasks one at a time is simpler to read and sufficient when operations depend on each other's results. Running them concurrently with `Task.WhenAll` reduces total wall-clock time when operations are independent, at the cost of needing to think about combined failure/cancellation behavior and, if unbounded, potentially overwhelming a downstream resource (see `bounded-concurrency.md`).

## How It Works

### Sequential — accidental and often slower than necessary

```csharp
var a = await client.GetStringAsync(url1); // waits fully before starting the next request
var b = await client.GetStringAsync(url2);
// Total time ≈ time(a) + time(b)
```

### Concurrent — start both, then await together

```csharp
var taskA = client.GetStringAsync(url1); // started, not yet awaited
var taskB = client.GetStringAsync(url2); // started, not yet awaited
var (a, b) = (await taskA, await taskB);
// Total time ≈ max(time(a), time(b))
```

The key detail: calling the async method *starts* the operation; `await` only *waits for* it. Awaiting immediately after each call forces sequential execution even though nothing about the operations required it.

### `Task.WhenAll` — wait for all, aggregate exceptions

```csharp
try
{
    var results = await Task.WhenAll(taskA, taskB, taskC);
}
catch (Exception)
{
    // Only the first exception is rethrown by await Task.WhenAll(...);
    // inspect each task's .Exception to see all failures if more than one task failed.
}
```

### `Task.WhenAny` — race, take the first

```csharp
Task<string> winner = await Task.WhenAny(primaryTask, fallbackTask);
string result = await winner; // re-await to observe its result or exception
```

A common use: implementing a timeout by racing the real operation against `Task.Delay`.

```csharp
var completed = await Task.WhenAny(operationTask, Task.Delay(TimeSpan.FromSeconds(5)));
if (completed != operationTask) throw new TimeoutException();
```

## Application

Use `Task.WhenAll` when you have several independent async operations and need all of their results before proceeding (e.g., fetching several unrelated resources to build one response). Use `Task.WhenAny` for racing, soft timeouts, or "whichever finishes first" logic.

## Common Mistakes

- Awaiting each task immediately after starting it, accidentally making independent operations run sequentially.
- Assuming `Task.WhenAll` throws all exceptions — `await Task.WhenAll(...)` only rethrows the *first* faulted task's exception; the others are still available on the `Task` objects themselves via `.Exception`.
- Using `Task.WhenAny` for a timeout pattern without cancelling the losing operation, leaving it running in the background consuming resources.
- Firing off an unbounded number of concurrent tasks with `Task.WhenAll` against a rate-limited or resource-constrained dependency, overwhelming it.

## Common Interview Questions

### Basic
- What's the difference between `Task.WhenAll` and `Task.WhenAny`?
- How do you run two independent async operations concurrently instead of sequentially?

### Intermediate
- Why does calling an async method without immediately awaiting it start the operation?
- What happens to exceptions when multiple tasks passed to `Task.WhenAll` fail?

### Advanced
- How would you implement a timeout for an operation that doesn't natively support `CancellationToken`, using `Task.WhenAny`?
- What resource-exhaustion risk does unbounded `Task.WhenAll` introduce, and how would you mitigate it?

### Follow-up Questions
- Does the losing task in a `Task.WhenAny` race get cancelled automatically?
- How do you inspect all exceptions from a `Task.WhenAll` call, not just the first?

### Code Prediction
```csharp
var taskA = SlowOperationAsync(); // takes 3 seconds
var taskB = await FastOperationAsync(); // takes 1 second, awaited immediately
var resultA = await taskA;
```
Is `SlowOperationAsync` running concurrently with `FastOperationAsync`, or does awaiting `FastOperationAsync` immediately change that? What's the total elapsed time?

## Practical Tasks

- Refactor a sequence of three independently-awaited HTTP calls into a concurrent `Task.WhenAll`-based version and measure the time difference.
- Implement a timeout wrapper for an arbitrary `Task` using `Task.WhenAny` and `Task.Delay`, including proper cancellation of the losing branch.
- Write code that correctly surfaces every exception from a `Task.WhenAll` call with multiple failures, not just the first.

## Readiness Criteria

Correctly reason about whether a piece of async code runs sequentially or concurrently by tracing when each task is started versus awaited, and use `Task.WhenAll`/`Task.WhenAny` correctly including their exception and cancellation subtleties.

## References

### Microsoft Learn

- [Task.WhenAll method](https://learn.microsoft.com/dotnet/api/system.threading.tasks.task.whenall)
- [Task.WhenAny method](https://learn.microsoft.com/dotnet/api/system.threading.tasks.task.whenany)
