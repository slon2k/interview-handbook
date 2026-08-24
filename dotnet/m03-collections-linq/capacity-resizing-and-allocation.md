# Capacity, Resizing, and Allocation Awareness

## Definition

Many collections maintain capacity separately from the number of stored elements. Capacity is the allocated room; count is the number of elements currently present.

```csharp
var values = new List<int>(capacity: 100);
Console.WriteLine(values.Count);    // 0
Console.WriteLine(values.Capacity); // 100
```

## Alternatives & Trade-offs

Preallocating capacity can reduce resizing and copying when the approximate size is known. Overallocating wastes memory. For unknown or changing workloads, allow the collection to grow and measure before tuning.

## How It Works

When an array-backed collection reaches capacity, it allocates a larger buffer and copies elements. This makes individual growth operations potentially expensive, but the total cost of repeated appends is usually amortized linear.

Removing elements often reduces count without immediately returning capacity to the runtime. `TrimExcess` can reduce spare capacity, but it may allocate and copy, so it should not be used indiscriminately.

## Application

- Set initial capacity for large, predictable batches.
- Materialize once when a sequence will be enumerated repeatedly.
- Avoid repeated conversions between collection types.
- Monitor allocation rate and GC pressure in hot paths.
- Consider arrays, spans, pooling, or specialized collections only after measurement.

## Common Mistakes

- Confusing `Count` with `Capacity`.
- Calling `TrimExcess` after every removal.
- Assuming a capacity hint guarantees exact allocation.
- Repeatedly growing a collection inside a high-volume loop without planning.
- Optimizing allocations before measuring the workload.

## Common Interview Questions

### Basic

- What is the difference between count and capacity?
- Why do resizable collections copy elements?
- What does `TrimExcess` do?

### Intermediate

- Why is repeated append amortized efficient?
- When should you provide an initial capacity?
- Does removing items always reduce allocated memory?

### Advanced

- How do resizing and GC pressure interact?
- What are the trade-offs of pooling collection buffers?
- How can capacity choices affect cache behavior?
- When is `ArrayPool<T>` appropriate?
- How would you measure whether preallocation helps?

### Follow-up Questions

- Does `List<T>.Clear()` release its backing array?
- Can capacity shrink automatically?
- What risks come with retaining a large list after its count drops?

### Code Prediction

What are the count and capacity after construction?

```csharp
var values = new List<int>(capacity: 10);
values.Add(1);
Console.WriteLine(values.Count);
Console.WriteLine(values.Capacity);
```

## Practical Tasks

- Measure list growth with and without an initial capacity.
- Compare memory behavior after `Clear` and `TrimExcess`.
- Design a batch processor that avoids unnecessary intermediate allocations.

## Readiness Criteria

Explain capacity, resizing, amortized growth, spare memory, and allocation measurement, and choose when a capacity hint or pooling strategy is justified.

## References

### Microsoft Learn

- [List<T>.Capacity property](https://learn.microsoft.com/dotnet/api/system.collections.generic.list-1.capacity)
- [ArrayPool<T> class](https://learn.microsoft.com/dotnet/api/system.buffers.arraypool-1)
- [Performance best practices](https://learn.microsoft.com/dotnet/framework/performance/)
