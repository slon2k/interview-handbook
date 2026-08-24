# Selecting the Correct Collection

## Definition

Collection selection is the process of choosing a data structure based on access patterns, ordering, uniqueness, mutation, concurrency, and memory constraints.

| Requirement | Starting point |
| --- | --- |
| Fixed-size indexed data | Array |
| Ordered resizable data | `List<T>` |
| Unique values and membership checks | `HashSet<T>` |
| Key-based lookup | `Dictionary<TKey, TValue>` |
| FIFO processing | `Queue<T>` |
| LIFO processing | `Stack<T>` |
| Sorted unique values | `SortedSet<T>` |
| Sorted key/value data | `SortedDictionary<TKey, TValue>` or `SortedList<TKey, TValue>` |
| Read-only shared snapshot | Immutable collection |
| Shared thread-safe state | Concurrent collection |

## Alternatives & Trade-offs

Start with the simplest collection that matches the dominant operation. Do not choose a linked list, concurrent collection, or sorted structure based only on theoretical advantages; workload, locality, allocation, and API ownership matter.

## How It Works

The selection should account for:

- Lookup by index, key, or value
- Duplicate and ordering requirements
- Insertions and removals near the beginning, middle, or end
- Number of reads versus writes
- Ownership and mutability
- Threading and coordination requirements
- Expected size and memory pressure

## Application

Describe the access pattern before selecting a type. For example, use a dictionary for repeated key lookup, a set for membership, and a list for ordered results that are mostly appended.

## Common Mistakes

- Choosing `List<T>` for repeated membership checks.
- Choosing `Dictionary<TKey, TValue>` when duplicate keys are meaningful.
- Assuming sorted collections are free to update.
- Using concurrent collections without a concurrency requirement.
- Returning mutable implementation types from public APIs unnecessarily.

## Common Interview Questions

### Basic

- How would you choose between a list, set, and dictionary?
- Which collection models FIFO behavior?
- Which collection guarantees unique values?

### Intermediate

- What questions should you ask before choosing a collection?
- When is a sorted collection worth its update cost?
- Why might an array outperform a linked list?

### Advanced

- How do cache locality and allocation shape collection choice?
- How would you select a collection for a read-heavy workload?
- When is a snapshot better than synchronized shared state?
- How do API ownership and mutability affect the collection interface you expose?
- How would you validate a collection choice with measurements?

### Follow-up Questions

- What if order and uniqueness are both required?
- What if the collection must support concurrent producers?
- What if keys can change after insertion?

### Code Prediction

Which collection is the best fit for counting word frequencies, and why?

```csharp
var counts = new Dictionary<string, int>(StringComparer.OrdinalIgnoreCase);
```

## Practical Tasks

- Build a decision table for five application access patterns.
- Refactor a list-based membership lookup to a set.
- Compare two candidate collections using representative benchmarks.

## Readiness Criteria

You should be able to state the dominant operations and constraints, select a suitable collection, explain alternatives, and defend the choice with complexity and workload reasoning.

## References

### Microsoft Learn

- [Selecting a collection class](https://learn.microsoft.com/dotnet/standard/collections/selecting-a-collection-class)
- [Collections and data structures](https://learn.microsoft.com/dotnet/standard/collections/)
