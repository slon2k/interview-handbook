# `lock`, `Monitor`, `Interlocked`, and `SemaphoreSlim`

## Definition

These are the core .NET tools for coordinating access to shared state across threads: `lock` (syntactic sugar over `Monitor`) grants exclusive access to a block of code; `Monitor` is the underlying primitive `lock` uses, with more advanced options (`TryEnter`, `Wait`/`Pulse`); `Interlocked` performs simple atomic operations without a lock at all; `SemaphoreSlim` limits how many callers (threads or async operations) may proceed at once, and — unlike `lock` — supports asynchronous waiting.

```csharp
private readonly object _gate = new();
private int _value;

public void Update(int newValue)
{
    lock (_gate) { _value = newValue; } // only one thread inside at a time
}
```

## Alternatives & Trade-offs

`lock`/`Monitor` is simple and effective for protecting a critical section, but it's synchronous only — you cannot `await` inside a `lock` block. `Interlocked` is faster and lock-free for simple operations (increment, compare-and-swap) but only covers a narrow set of atomic operations. `SemaphoreSlim` is the right tool when you need to limit concurrency (not just enforce exclusivity) or when the protected section itself needs to be asynchronous.

## How It Works

### `lock` and its limitation with `await`

```csharp
private readonly object _gate = new();

public async Task UpdateAsync()
{
    lock (_gate)
    {
        // await httpClient.GetStringAsync(url); // compile error — cannot await inside a lock
    }
}
```

`lock` is built on `Monitor.Enter`/`Monitor.Exit`, which are tied to the calling thread; an `await` can resume on a different thread, which would break the lock's thread-ownership model. This is precisely why `SemaphoreSlim` exists for async code.

### `SemaphoreSlim` as an async-compatible lock

```csharp
private readonly SemaphoreSlim _semaphore = new(1, 1); // acts like a mutex, but awaitable

public async Task UpdateAsync()
{
    await _semaphore.WaitAsync();
    try
    {
        await httpClient.GetStringAsync(url); // awaiting inside is fine here
    }
    finally
    {
        _semaphore.Release();
    }
}
```

### `Interlocked` for simple atomic operations

```csharp
private int _counter;
public void Increment() => Interlocked.Increment(ref _counter);
public int Read() => Interlocked.CompareExchange(ref _counter, 0, 0); // safe read pattern
```

`Interlocked` avoids the overhead of a lock for operations it directly supports (increment, decrement, exchange, compare-and-swap), but doesn't generalize to protecting an arbitrary block of code.

### `Monitor` beyond `lock`

```csharp
if (Monitor.TryEnter(_gate, TimeSpan.FromMilliseconds(100)))
{
    try { /* work */ }
    finally { Monitor.Exit(_gate); }
}
else
{
    // couldn't acquire within the timeout — avoid blocking indefinitely
}
```

## Application

Use `lock` for simple, synchronous, short critical sections. Use `Interlocked` for single-variable atomic updates where a full lock would be overkill. Use `SemaphoreSlim` whenever the protected work is asynchronous, or when you need to allow more than one caller through at a time (bounded concurrency — see `bounded-concurrency.md`).

## Common Mistakes

- Trying to `await` inside a `lock` block (a compile error) instead of switching to `SemaphoreSlim`.
- Using a full `lock` where `Interlocked` would suffice, adding unnecessary contention for a simple counter.
- Forgetting to release a `SemaphoreSlim` in a `finally` block, permanently reducing available concurrency if an exception is thrown mid-operation.
- Locking on a publicly accessible object (`lock(this)` or a public field) instead of a private, dedicated lock object, allowing external code to accidentally interfere with the lock.

## Common Interview Questions

### Basic
- What is the relationship between `lock` and `Monitor`?
- Why can't you `await` inside a `lock` block?

### Intermediate
- When would you use `Interlocked` instead of `lock`?
- How does `SemaphoreSlim` differ from `lock` in terms of what it can protect?

### Advanced
- How would you implement an async-safe mutex using `SemaphoreSlim`, and why can't a plain `lock` do the same job?
- What's the risk of locking on `this` or a public object, and how do private lock objects avoid it?

### Follow-up Questions
- Can `SemaphoreSlim` allow more than one caller through simultaneously? How?
- Does `Interlocked.Increment` require any additional locking to be safe?

### Code Prediction
```csharp
private readonly SemaphoreSlim _semaphore = new(2, 2); // allows 2 concurrent callers

async Task WorkAsync(int id)
{
    await _semaphore.WaitAsync();
    try { await Task.Delay(1000); Console.WriteLine(id); }
    finally { _semaphore.Release(); }
}

await Task.WhenAll(Enumerable.Range(1, 5).Select(WorkAsync));
```
Roughly how long does this take to complete, given 5 calls but only 2 allowed to run concurrently at a time?

## Practical Tasks

- Convert a synchronous `lock`-protected method that needs to call an async API into an async-safe version using `SemaphoreSlim`.
- Replace an unnecessary `lock` around a single counter increment with `Interlocked.Increment`, and verify correctness under concurrent load.
- Implement a private lock object pattern and explain why locking on `this` would be riskier.

## Readiness Criteria

Choose the right primitive (`lock`, `Interlocked`, `SemaphoreSlim`) for a given synchronization need, explain why `lock` can't wrap `await`, and correctly release semaphores even when exceptions occur.

## References

### Microsoft Learn

- [lock statement](https://learn.microsoft.com/dotnet/csharp/language-reference/statements/lock)
- [Monitor class](https://learn.microsoft.com/dotnet/api/system.threading.monitor)
- [Interlocked class](https://learn.microsoft.com/dotnet/api/system.threading.interlocked)
- [SemaphoreSlim class](https://learn.microsoft.com/dotnet/api/system.threading.semaphoreslim)
