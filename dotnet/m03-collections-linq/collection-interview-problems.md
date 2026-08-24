# Collection and String Interview Problems

## Definition

Collection interview problems test data-structure selection, invariants, complexity, and the ability to explain trade-offs while implementing a correct solution.

## Alternatives & Trade-offs

Start with a straightforward solution, state its complexity, then improve it only when the constraints require improvement. A hash set or dictionary often trades memory for faster lookup; sorting can trade preprocessing time for ordered or binary-search access.

## How It Works

Common patterns include:

- Frequency counting with `Dictionary<TKey, TValue>`.
- Duplicate detection with `HashSet<T>`.
- Two pointers for ordered or paired data.
- Sliding windows for contiguous ranges.
- Prefix sums for repeated range totals.
- Stack-based matching for nested delimiters.
- Queue-based BFS for shortest unweighted paths.

## Application

Use these patterns for string frequency, duplicate, grouping, interval, traversal, and filtering problems. Always state input assumptions such as null handling, case sensitivity, ordering, and expected size.

## Common Mistakes

- Coding before clarifying constraints.
- Choosing a data structure without stating why.
- Ignoring duplicate, empty, or null inputs.
- Returning the right result with an unnecessarily high complexity.
- Over-optimizing before producing a clear baseline solution.

## Common Interview Questions

### Basic

- How do you find duplicates in an array?
- How do you count character frequencies?
- How do you check whether two strings are anagrams?

### Intermediate

- How does a sliding window reduce repeated work?
- When do two pointers apply?
- How do you find the first non-repeating character?

### Advanced

- How do you handle memory limits for frequency counting?
- How would you process a stream where the full input cannot be stored?
- How do comparer and normalization choices affect string problems?
- How do you prove a sliding-window invariant?
- How would you adapt a solution for concurrent producers?

### Follow-up Questions

- What changes if the input is already sorted?
- What if values do not fit in memory?
- What is the time and space complexity?
- How should case and Unicode be handled?

### Code Prediction

What is the complexity of duplicate detection?

```csharp
var seen = new HashSet<int>();
foreach (int value in values)
{
    if (!seen.Add(value)) return true;
}
return false;
```

## Practical Tasks

- Solve duplicate detection with a set and with sorting.
- Implement an anagram check with an explicit string comparison policy.
- Solve a longest-substring-without-repetition problem using a sliding window.
- Explain correctness and complexity before optimizing.

## Readiness Criteria

Clarify constraints, select an appropriate pattern and collection, implement edge cases, explain correctness, and state realistic time and space complexity.

## References

### Microsoft Learn

- [Dictionary<TKey, TValue> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.dictionary-2)
- [HashSet<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.hashset-1)
- [String comparison](https://learn.microsoft.com/dotnet/standard/base-types/best-practices-strings)
