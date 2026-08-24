# `Task` and `Task<T>`

## Definition

`Task` represents an asynchronous operation that produces no value; `Task<T>` represents one that produces a value of type `T` on completion. Both model a future result, exposing its completion state, result (or exception), and continuation support.

```csharp
Task saveTask = repository.SaveAsync(order);       // no return value
Task<int> countTask = repository.CountAsync();      // returns an int when done
```

## Alternatives & Trade-offs

`Task`/`Task<T>` are the standard .NET abstraction for asynchronous work and integrate directly with `async`/`await`. `ValueTask`/`ValueTask<T>` avoid an allocation for operations that frequently complete synchronously (e.g., a cache hit), at the cost of stricter usage rules (a `ValueTask` should generally be awaited exactly once and not stored). For most application code, `Task<T>` is the right default; reach for `ValueTask<T>` only in hot paths where profiling shows the allocation matters.

## How It Works

### `Task.Run` offloads work to the thread pool

```csharp
Task<int> result = Task.Run(() => ComputeExpensiveHash(buffer)); // runs on a thread-pool thread
int hash = await result;
```

`Task.Run` is for CPU-bound work you want off the current thread. It is not needed for I/O-bound async methods, which don't occupy a thread while waiting in the first place.

### A `Task` can represent already-completed or synchronous-looking work

```csharp
public Task<int> GetCachedValueAsync(string key)
{
    if (_cache.TryGetValue(key, out var value))
        return Task.FromResult(value); // no actual asynchrony — a completed Task wrapping a value
    return LoadFromDatabaseAsync(key);
}
```

### Awaiting extracts the result or rethrows the exception

```csharp
Task<int> task = ComputeAsync();
int value = await task;         // unwraps the result, or rethrows the original exception
// vs.
int value2 = task.Result;       // blocks synchronously and wraps exceptions in AggregateException — avoid this
```

### Task status and completion

```csharp
Task task = DoWorkAsync();
Console.WriteLine(task.Status);      // Created, WaitingForActivation, Running, RanToCompletion, Faulted, Canceled
await task;
Console.WriteLine(task.IsCompletedSuccessfully);
```

## Application

Use `Task<T>` as the return type for any method that does asynchronous work and returns a value; use `Task` for asynchronous methods with no return value. Use `Task.Run` specifically to offload CPU-bound work from a thread that shouldn't be blocked (e.g., a UI thread), not as a general-purpose way to "make something async."

## Common Mistakes

- Calling `.Result` or `.Wait()` on a `Task` instead of `await`ing it, which can cause deadlocks in contexts with a synchronization context (see `synchronization-context-and-configureawait.md`) and wraps exceptions in `AggregateException`.
- Using `Task.Run` inside an already-asynchronous I/O method, adding an unnecessary thread-pool hop with no benefit.
- Returning `null` instead of `Task.CompletedTask` or a completed `Task<T>` from a method with a `Task` return type, causing a `NullReferenceException` at the await site.
- Not understanding that a `Task` is "hot" — it starts running as soon as it's created (unlike `IEnumerable`'s lazy evaluation), so creating one and never awaiting it doesn't prevent it from running; it just risks losing track of its exceptions.

## Common Interview Questions

### Basic
- What is the difference between `Task` and `Task<T>`?
- What does `Task.Run` do?

### Intermediate
- Why is calling `.Result` risky compared to `await`?
- What does `Task.FromResult` do, and when would you use it?

### Advanced
- What is a "hot" task, and why does an unobserved faulted task matter?
- When is `ValueTask<T>` worth the added usage constraints compared to `Task<T>`?

### Follow-up Questions
- What state is a `Task` in immediately after being created by an `async` method?
- Does awaiting a completed `Task` still yield control back to the caller?

### Code Prediction
```csharp
Task<int> task = ComputeAsync(); // starts running immediately
await Task.Delay(100);
int result = await task;
```
Does `ComputeAsync` start running when the `Task<int>` is created, or only when it's awaited? What does that imply about the total elapsed time compared to awaiting it immediately?

## Practical Tasks

- Write a method returning `Task<int>` that sometimes completes synchronously (`Task.FromResult`) and sometimes asynchronously, and observe both paths through `await`.
- Reproduce a deadlock caused by blocking on `.Result` in a context with a synchronization context, then fix it with `await`.
- Benchmark `Task<T>` vs. `ValueTask<T>` for a method that completes synchronously most of the time.

## Readiness Criteria

Explain the difference between `Task` and `Task<T>`, explain why blocking on a task is risky, and know when `Task.Run` and `ValueTask<T>` are appropriate versus unnecessary.

## References

### Microsoft Learn

- [Task class](https://learn.microsoft.com/dotnet/api/system.threading.tasks.task)
- [Task<TResult> class](https://learn.microsoft.com/dotnet/api/system.threading.tasks.task-1)
- [ValueTask<TResult> struct](https://learn.microsoft.com/dotnet/api/system.threading.tasks.valuetask-1)
