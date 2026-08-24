# Searching and Sorting

## Definition

Searching finds an item or position in a collection. Sorting arranges items according to a comparer so that later operations, such as binary search, can be faster or results can be presented predictably.

```csharp
int index = Array.IndexOf(values, target);
Array.Sort(values);
```

## Alternatives & Trade-offs

Linear search works on unsorted data and costs $O(n)$. Sorting costs additional time but can support repeated binary searches in $O(\log n)$ each. Hash-based lookup is often better when membership or key lookup is the primary requirement.

Use the standard library unless the interview specifically asks for an implementation. It is usually better tested and may use optimized algorithms.

## How It Works

Common approaches:

- Linear search scans elements until a match is found.
- Hash lookup uses a key and comparer for average $O(1)$ access.
- Comparison sorting commonly costs $O(n \log n)$ on average.
- Stable sorting preserves the relative order of equal elements.
- Selection, insertion, and bubble sort are useful for learning but usually not production defaults.

## Application

- Search unsorted, small, or one-off sequences linearly.
- Sort when ordered output or repeated binary search is valuable.
- Use dictionaries or sets for repeated membership and key access.
- Provide an explicit comparer when domain ordering is not the default ordering.

## Common Mistakes

- Sorting for a single lookup when a set or dictionary is more appropriate.
- Assuming every sort is stable.
- Sorting a collection in place when callers expect the original order preserved.
- Forgetting null and comparer behavior.
- Using a linear search repeatedly inside a loop and creating quadratic work.

## Common Interview Questions

### Basic

- What is linear search?
- What is the complexity of sorting?
- When is a dictionary better than searching a list?

### Intermediate

- What makes a sorting algorithm stable?
- When is binary search applicable?
- What is the difference between in-place and out-of-place sorting?

### Advanced

- How do comparer cost and data movement affect sorting performance?
- How would you choose between sorting once and building an index?
- Why can a linear scan outperform a hash lookup for small collections?
- How do partial-selection algorithms avoid fully sorting data?
- How should a sort handle a comparer that is inconsistent with equality?

### Follow-up Questions

- Does `List<T>.Sort` preserve equal-element order?
- Can a sorted collection be searched in constant time?
- What happens when a target occurs more than once?

### Code Prediction

What is the complexity of the repeated search?

```csharp
foreach (string name in names)
{
    if (blockedNames.Contains(name)) { }
}
```

## Practical Tasks

- Compare repeated list search with a prebuilt `HashSet<T>`.
- Implement stable sorting for objects with equal primary keys.
- Choose an approach for top-k results without fully sorting all input.

## Readiness Criteria

Explain linear search, sorting cost and stability, hash-based alternatives, comparer behavior, and how repeated operations affect total complexity.

## References

### Microsoft Learn

- [Array.Sort](https://learn.microsoft.com/dotnet/api/system.array.sort)
- [List<T>.Sort](https://learn.microsoft.com/dotnet/api/system.collections.generic.list-1.sort)
- [Comparer<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.comparer-1)
