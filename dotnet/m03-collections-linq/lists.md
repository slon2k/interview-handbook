# `List<T>`

## Definition

`List<T>` is a resizable, zero-based collection backed by an array.

```csharp
var names = new List<string> { "Ada", "Grace" };
names.Add("Lin");
```

## Alternatives & Trade-offs

`List<T>` is a strong default for ordered data with frequent indexing and appends. Use `LinkedList<T>` only for specific node-insertion workloads, and use a set or dictionary when uniqueness or key lookup is the primary requirement.

## How It Works

A list maintains a `Count` and an internal capacity. When capacity is exceeded, it allocates a larger array and copies elements. Indexing is $O(1)$; appending is amortized $O(1)$; inserting or removing in the middle is $O(n)$.

`CollectionsMarshal.AsSpan` exposes the list's internal storage without copying. The span must not be used after an operation that can resize the list, and it should be reserved for carefully reviewed performance-sensitive code.

## Application

Use lists for ordered results, buffers that grow over time, and APIs returning materialized collections.

## Common Mistakes

- Removing items while iterating with `foreach`.
- Assuming `Capacity` equals `Count`.
- Repeatedly inserting at index zero.
- Returning a mutable list when callers should not mutate the collection.
- Using `Contains` for repeated membership checks instead of a set.

## Common Interview Questions

### Basic

- What is `List<T>`?
- How does it differ from an array?
- What do `Count` and `Capacity` mean?

### Intermediate

- What is the complexity of add, index, insert, and remove?
- What happens when capacity is exceeded?
- How can you expose a read-only view?

### Advanced

- Why is append amortized constant time?
- How do capacity growth and allocation affect performance?
- When is `CollectionsMarshal.AsSpan` appropriate?
- How can list mutation invalidate enumeration?
- How would you choose initial capacity?

### Follow-up Questions

- Does `List<T>` guarantee ordering?
- Is `List<T>` thread-safe?
- What is the difference between `AsReadOnly` and copying?

### Code Prediction

What is the final count?

```csharp
var values = new List<int> { 1, 2, 3 };
values.Remove(2);
values.Add(4);
Console.WriteLine(values.Count);
```

## Practical Tasks

- Implement a stable filter without modifying the source during enumeration.
- Measure the effect of preallocating capacity.
- Design an API that exposes read-only results.

## Readiness Criteria

Explain list capacity, operation complexity, mutation rules, ordering, and when a list should be replaced by a set, dictionary, or array.

## References

### Microsoft Learn

- [List<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.list-1)
- [How to initialize a collection](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/how-to-initialize-objects-by-using-an-object-initializer)
