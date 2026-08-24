# Binary Search

## Definition

Binary search finds a value in a sorted sequence by repeatedly eliminating half of the remaining search interval.

```csharp
int index = Array.BinarySearch(values, target);
```

## Alternatives & Trade-offs

Binary search provides $O(\log n)$ lookup but requires sorted data and random access or an equivalent ability to divide the search space. Linear search works on unsorted data. A dictionary is often better for repeated exact key lookup.

Sorting first costs $O(n \log n)$, so binary search is most useful when the data is reused for multiple searches or arrives already sorted.

## How It Works

Compare the target with the middle element. If the target is smaller, search the lower half; if larger, search the upper half. Stop when the target is found or the interval is empty.

Use overflow-safe midpoint calculation:

```csharp
int middle = low + (high - low) / 2;
```

The iterative algorithm uses $O(1)$ extra space and $O(\log n)$ time.

## Application

- Search sorted arrays or lists.
- Find insertion points and lower or upper bounds.
- Apply the binary-search-on-answer technique when a feasibility condition is monotonic.

## Common Mistakes

- Searching unsorted data.
- Using incorrect inclusive or exclusive boundaries.
- Forgetting duplicate values may produce any matching index.
- Creating an overflow-prone midpoint.
- Failing to make progress when the middle value does not match.

## Common Interview Questions

### Basic

- What precondition does binary search require?
- What is its time complexity?
- What is its space complexity when implemented iteratively?

### Intermediate

- How do you find the first or last matching value?
- How do you calculate a safe midpoint?
- When is sorting plus binary search better than a dictionary?

### Advanced

- How do you adapt binary search to a monotonic predicate?
- How do duplicate values affect correctness and result guarantees?
- How does random access affect the choice of search algorithm?
- How would you prove termination for a boundary-based implementation?
- How do floating-point comparisons complicate binary search?

### Follow-up Questions

- Can binary search work on a linked list efficiently?
- What should be returned when the target is absent?
- Why are half-open intervals useful?

### Code Prediction

What does the return value represent if `7` is absent?

```csharp
int[] values = [1, 3, 5, 9];
int result = Array.BinarySearch(values, 7);
```

## Practical Tasks

- Implement iterative binary search with a half-open interval.
- Implement first-occurrence and insertion-point variants.
- Test empty, one-element, duplicate, and absent-target inputs.

## Readiness Criteria

Implement binary search without boundary bugs, state its preconditions and complexity, and adapt it to duplicates, insertion points, or monotonic feasibility checks.

## References

### Microsoft Learn

- [Array.BinarySearch](https://learn.microsoft.com/dotnet/api/system.array.binarysearch)
- [List<T>.BinarySearch](https://learn.microsoft.com/dotnet/api/system.collections.generic.list-1.binarysearch)
