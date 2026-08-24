# Race Conditions, Deadlocks, and Shared Mutable State

## Definition

A **race condition** occurs when the correctness of a result depends on the unpredictable timing of concurrent operations accessing shared mutable state. A **deadlock** occurs when two or more operations wait on each other indefinitely, none able to proceed. Both stem from the same root cause: multiple threads touching shared mutable state without adequate coordination.

```csharp
private int _counter = 0;
public void Increment() => _counter++; // not atomic — a race condition under concurrent calls
```

## Alternatives & Trade-offs

Sharing mutable state across threads is fast (no copying) but requires careful synchronization to stay correct. Avoiding shared mutable state entirely (immutable data, message-passing, or thread-local state) sidesteps the problem at the cost of some design flexibility or extra copying. Synchronization primitives (`lock`, `Interlocked`) fix correctness but can introduce contention and, if misused, deadlocks of their own.

## How It Works

### Race condition

```csharp
private int _counter = 0;

public void IncrementManyTimes()
{
    Parallel.For(0, 100_000, _ => _counter++); // reads, adds 1, writes — not atomic
}
// Final _counter is typically less than 100,000 due to lost updates between concurrent read-modify-write sequences
```

`_counter++` is actually three steps (read, add, write); two threads can both read the same value before either writes back, losing one of the increments.

### Fixing it

```csharp
private int _counter = 0;
public void IncrementManyTimes()
{
    Parallel.For(0, 100_000, _ => Interlocked.Increment(ref _counter)); // atomic
}
```

### Deadlock — classic two-lock scenario

```csharp
private readonly object _lockA = new();
private readonly object _lockB = new();

void MethodOne()
{
    lock (_lockA) { lock (_lockB) { /* work */ } }
}

void MethodTwo()
{
    lock (_lockB) { lock (_lockA) { /* work */ } } // acquires in the opposite order
}
```

If `MethodOne` and `MethodTwo` run concurrently, each can acquire its first lock and then block forever waiting for the other's lock — a deadlock. The fix is consistent lock ordering across all code paths.

### A common async-specific deadlock

```csharp
// In a context with a synchronization context (e.g., older ASP.NET, WPF, WinForms):
public void ButtonClick()
{
    var result = GetDataAsync().Result; // blocks the UI thread waiting for a continuation
                                          // that needs the same UI thread to resume — deadlock
}
```

This is covered in depth in `synchronization-context-and-configureawait.md`; it's mentioned here because it's the most common deadlock pattern async code introduces.

## Application

Watch for shared mutable state anywhere multiple threads or concurrent tasks can touch the same field, collection, or resource — counters, caches, in-memory state in singletons. Use atomic operations (`Interlocked`), locks, or immutability to eliminate races; use consistent lock ordering (or avoid nested locks altogether) to eliminate deadlocks.

## Common Mistakes

- Assuming a simple increment or read-then-write on a field is atomic — it almost never is without `Interlocked` or a lock.
- Acquiring multiple locks in inconsistent order across different code paths, creating a latent deadlock that only appears under concurrent load.
- Believing a race condition will always be caught by tests — races are often intermittent and load-dependent, so they can pass in dev and fail intermittently in production.
- Adding a lock reactively around symptoms without identifying every place the shared state is touched, leaving other unprotected access paths.

## Common Interview Questions

### Basic
- What is a race condition? What is a deadlock?
- Why isn't `counter++` thread-safe?

### Intermediate
- How does `Interlocked.Increment` avoid the race condition in the counter example?
- What causes the classic two-lock deadlock, and how do you prevent it?

### Advanced
- Why are race conditions often hard to reproduce in tests, and how would you approach diagnosing one reported only in production under load?
- How does blocking on `.Result`/`.Wait()` inside a `SynchronizationContext`-bound method cause a deadlock specifically?

### Follow-up Questions
- Does using a `lock` always prevent a deadlock?
- What tools or techniques can help detect race conditions before they reach production?

### Code Prediction
Given the `_counter++` example under `Parallel.For(0, 100_000, ...)`, is the final value of `_counter` guaranteed to be 100,000? Why or why not?

## Practical Tasks

- Reproduce the lost-update race condition with `Parallel.For` and a plain `_counter++`, then fix it with `Interlocked.Increment` and verify the count is correct.
- Construct the two-lock deadlock example above, observe it hang, then fix it with consistent lock ordering.
- Reproduce the `.Result`-in-a-synchronization-context deadlock in a small WinForms or ASP.NET (classic) example, if available, or explain it precisely if not.

## Readiness Criteria

Identify race conditions and deadlock risks in code involving shared mutable state, fix races with atomic operations or locks, and prevent deadlocks through consistent lock ordering or avoiding blocking calls in synchronization-context-bound code.

## References

### Microsoft Learn

- [Interlocked class](https://learn.microsoft.com/dotnet/api/system.threading.interlocked)
- [Thread synchronization](https://learn.microsoft.com/dotnet/standard/threading/overview-of-synchronization-primitives)
