# Concurrent Collections

## Definition

The `System.Collections.Concurrent` namespace provides collection types designed for safe access from multiple threads without external locking — `ConcurrentDictionary<TKey, TValue>`, `ConcurrentQueue<T>`, `ConcurrentBag<T>`, `ConcurrentStack<T>`, and `BlockingCollection<T>`.

```csharp
private readonly ConcurrentDictionary<string, int> _cache = new();

public int GetOrAdd(string key) => _cache.GetOrAdd(key, k => ComputeExpensive(k));
```

## Alternatives & Trade-offs

Concurrent collections avoid the need for manual `lock` statements around a plain `Dictionary<TKey,TValue>` or `List<T>`, and are generally more efficient than a coarse-grained lock for high-contention scenarios, since many use fine-grained internal locking or lock-free algorithms. They are not automatically faster for low-contention, single-threaded-mostly access — a plain collection guarded by a simple lock (or no lock at all, if truly single-threaded) can outperform them there.

## How It Works

### `ConcurrentDictionary` compound operations

```csharp
_cache.GetOrAdd("key", k => ComputeExpensive(k)); // atomic: no race between checking and adding
_cache.AddOrUpdate("key", 1, (k, existing) => existing + 1); // atomic add-or-update
```

A plain `Dictionary<TKey, TValue>` requires you to manually lock around a check-then-act sequence (`ContainsKey` then `Add`) to avoid a race; `ConcurrentDictionary` provides that atomicity built in.

### The `GetOrAdd` factory can run more than once

```csharp
_cache.GetOrAdd("key", k =>
{
    Console.WriteLine("Computing...");
    return ComputeExpensive(k);
});
```

Under concurrent calls for the same missing key, the factory delegate may execute on more than one thread before one result "wins" and is stored — a common surprise if the factory has side effects.

### `BlockingCollection<T>` as a producer/consumer queue

```csharp
var queue = new BlockingCollection<int>(boundedCapacity: 100);

// Producer
queue.Add(item);
queue.CompleteAdding();

// Consumer
foreach (var item in queue.GetConsumingEnumerable())
{
    Process(item);
}
```

### `ConcurrentBag<T>` — unordered, thread-local optimized

```csharp
var bag = new ConcurrentBag<int>();
Parallel.For(0, 1000, i => bag.Add(i)); // fast for scenarios where each thread mostly adds/removes its own items
```

## Application

Use `ConcurrentDictionary` for shared caches and lookups accessed from multiple threads. Use `BlockingCollection` for classic bounded producer/consumer pipelines. Use `ConcurrentQueue`/`ConcurrentBag`/`ConcurrentStack` when multiple threads need to add/remove items without external locking. For a collection only ever touched by one thread at a time, a concurrent collection adds overhead with no benefit — use the plain version.

## Common Mistakes

- Reaching for a concurrent collection reflexively "to be safe," even in code that's never actually accessed from more than one thread.
- Assuming `ConcurrentDictionary.GetOrAdd`'s factory delegate runs exactly once per key — it can run multiple times concurrently for the same missing key.
- Iterating a concurrent collection while assuming a consistent snapshot — enumeration is thread-safe (won't throw) but may reflect a mix of before/after states as other threads mutate it concurrently.
- Using `ConcurrentBag<T>` for a scenario needing ordering — it has no ordering guarantee at all.

## Common Interview Questions

### Basic
- What's the difference between `Dictionary<TKey, TValue>` and `ConcurrentDictionary<TKey, TValue>`?
- Why might you use `BlockingCollection<T>`?

### Intermediate
- Why can `ConcurrentDictionary.GetOrAdd`'s factory run more than once for the same key?
- Is enumerating a `ConcurrentDictionary` while another thread modifies it safe? What does "safe" mean here specifically?

### Advanced
- When would a plain `Dictionary<TKey, TValue>` protected by a single `lock` outperform a `ConcurrentDictionary`?
- How would you design a bounded producer/consumer pipeline using `BlockingCollection<T>`, including graceful shutdown?

### Follow-up Questions
- Does `ConcurrentQueue<T>` guarantee FIFO ordering under concurrent access?
- What happens if you call `Add` on a `BlockingCollection` after `CompleteAdding`?

### Code Prediction
```csharp
var cache = new ConcurrentDictionary<string, int>();
Parallel.For(0, 10, _ => cache.GetOrAdd("key", k => { Console.WriteLine("computing"); return 42; }));
```
Is "computing" guaranteed to print exactly once? Why or why not?

## Practical Tasks

- Replace a manually-locked `Dictionary<TKey, TValue>` cache with `ConcurrentDictionary` and simplify the surrounding code.
- Build a small bounded producer/consumer pipeline using `BlockingCollection<T>` with multiple producers and one consumer.
- Reproduce and explain the multiple-factory-execution behavior of `ConcurrentDictionary.GetOrAdd` under concurrent load.

## Readiness Criteria

Choose the correct concurrent collection for a given access pattern, understand the atomicity guarantees (and gaps) of compound operations like `GetOrAdd`, and judge when a concurrent collection isn't actually needed.

## References

### Microsoft Learn

- [Thread-safe collections](https://learn.microsoft.com/dotnet/standard/collections/thread-safe/)
- [ConcurrentDictionary<TKey,TValue> class](https://learn.microsoft.com/dotnet/api/system.collections.concurrent.concurrentdictionary-2)
- [BlockingCollection<T> class](https://learn.microsoft.com/dotnet/api/system.collections.concurrent.blockingcollection-1)
