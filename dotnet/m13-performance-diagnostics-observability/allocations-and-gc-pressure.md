# Allocations and GC Pressure

## Definition

Module 5 covered garbage collection mechanics; this topic is about *diagnosing and reducing* excessive allocation in practice — "GC pressure" describes a situation where a program allocates so much (especially short-lived) memory that the garbage collector runs frequently enough to noticeably affect throughput or latency, particularly in high-throughput services.

```csharp
// Allocates a new string on every call, for a hot path called millions of times per minute
public string FormatOrderId(int id) => $"ORDER-{id:D8}"; // each call: a new string allocation
```

## Alternatives & Trade-offs

Allocating freely and letting the GC handle cleanup is simpler to write and correct in almost all code — premature allocation-reduction effort on cold paths wastes time for no measurable benefit. Reducing allocations matters specifically on genuinely hot paths (called very frequently, under real load) where the cumulative GC pressure becomes measurable — this is squarely a "measure before optimizing" situation (see the first topic in this module), not a blanket coding style to apply everywhere.

## How It Works

### Spotting allocation-heavy patterns in a hot path

```csharp
// Each of these allocates on every call — fine occasionally, potentially costly at high volume
public string BuildSummary(Order order) => $"Order {order.Id}: {order.Total}"; // string allocation
public List<int> GetIds(IEnumerable<Order> orders) => orders.Select(o => o.Id).ToList(); // list + enumerator allocations
```

### Reducing allocations with `Span<T>` and `stackalloc` for genuinely hot, low-level code

```csharp
// Avoids a heap allocation for a small, short-lived buffer, for code where this measurably matters
Span<char> buffer = stackalloc char[16];
id.TryFormat(buffer, out int written);
```

`Span<T>`-based APIs let performance-critical, low-level code avoid heap allocations entirely for short-lived buffers — valuable in hot paths (a high-throughput parser, a serializer) but unnecessary ceremony for ordinary application code that isn't a measured bottleneck.

### Object pooling — reusing expensive-to-allocate objects

```csharp
private static readonly ObjectPool<StringBuilder> _pool = new DefaultObjectPoolProvider().CreateStringBuilderPool();

public string BuildLargeReport()
{
    var sb = _pool.Get();
    try { /* build the report using sb */ return sb.ToString(); }
    finally { _pool.Return(sb); }
}
```

Pooling trades a small amount of bookkeeping complexity for avoiding repeated allocation/collection of objects that are expensive to construct or are allocated very frequently.

### Measuring allocations, not just guessing

```csharp
[MemoryDiagnoser] // BenchmarkDotNet attribute — reports allocated bytes per operation, not just time
public class FormattingBenchmarks
{
    [Benchmark] public string StringInterpolation() => $"ORDER-{42:D8}";
    [Benchmark] public string SpanBased() { /* ... */ }
}
```

### GC generations and why short-lived allocations matter most

```
Gen 0: short-lived objects, collected frequently and cheaply
Gen 1: intermediate — survived at least one Gen 0 collection
Gen 2: long-lived objects, collected rarely but expensively (full collection)
```

High-frequency allocation of short-lived objects (Gen 0 pressure) is usually the more actionable target — it's directly tied to allocation *rate*, which is exactly what reducing per-call allocations in a hot path addresses.

## Application

Investigate allocation-driven performance issues specifically on measured hot paths — high-frequency code paths under real production load — using a profiler's memory view or `[MemoryDiagnoser]` benchmarks to quantify allocation rate, not intuition. Apply `Span<T>`/pooling/allocation-reduction techniques where the measurement justifies it; leave ordinary, infrequently-called code alone.

## Common Mistakes

- Applying allocation-reduction techniques (`Span<T>`, pooling) to cold paths where the added complexity provides no measurable benefit.
- Assuming a slow operation is GC-related without actually measuring allocation rate or GC pause frequency via a profiler.
- Pooling objects without properly resetting their state between uses, causing state to leak from one use to the next (the same category of bug as EF Core's `DbContext` pooling from Module 10).
- Optimizing for allocation count alone without considering whether the resulting code is meaningfully harder to read/maintain for a benefit that measurement didn't actually justify.

## Common Interview Questions

### Basic
- What does "GC pressure" mean, and why does it matter for a high-throughput service specifically?
- What's the difference between Gen 0, Gen 1, and Gen 2 collections in terms of what typically drives them?

### Intermediate
- Why does reducing allocations in a rarely-called method provide little practical benefit?
- What does `[MemoryDiagnoser]` in BenchmarkDotNet measure, and why is that more useful than timing alone for this kind of investigation?

### Advanced
- How would you identify, in a running production service, whether GC activity is a meaningful contributor to observed latency spikes?
- What's the trade-off of introducing object pooling for a frequently-allocated type, and what bug class does it risk if not implemented carefully?

### Follow-up Questions
- Does `Span<T>` eliminate all heap allocation for the code that uses it?
- Is reducing allocations always worth the added code complexity?

### Code Prediction
A profiler shows a specific string-formatting method allocating 500MB total over a 10-minute load test, called 10 million times, with each individual call taking microseconds. Is this necessarily a problem worth fixing, and what would you check before deciding?

## Practical Tasks

- Use a profiler's memory view to identify the top allocation sources in a sample application under load.
- Write a `[MemoryDiagnoser]` benchmark comparing an allocation-heavy implementation against a `Span<T>`-based alternative for a hot-path formatting operation.
- Implement object pooling for a genuinely expensive-to-construct type and verify correct state reset between uses.

## Readiness Criteria

Diagnose GC pressure using actual measurement rather than intuition, apply allocation-reduction techniques only where measurement justifies the added complexity, and avoid the state-leakage risk of incorrect pooling.

## References

### Microsoft Learn

- [Garbage collection fundamentals](https://learn.microsoft.com/dotnet/standard/garbage-collection/fundamentals)
- [Span<T> and memory<T> usage guidelines](https://learn.microsoft.com/dotnet/standard/memory-and-spans/memory-t-usage-guidelines)

### Other

- [BenchmarkDotNet: MemoryDiagnoser](https://benchmarkdotnet.org/articles/configs/diagnosers.html)
