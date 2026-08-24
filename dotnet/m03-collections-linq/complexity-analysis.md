# Collection Complexity

## Definition

Complexity describes how time or memory use grows as input size increases. Big-O notation expresses an upper growth pattern while ignoring constant factors.

Common classes include $O(1)$, $O(\log n)$, $O(n)$, and $O(n \log n)$.

## Alternatives & Trade-offs

Asymptotic complexity helps compare scalable approaches, but it is not the only performance measure. Constant factors, cache locality, allocations, branch behavior, input distribution, and actual workload can change the best choice.

## How It Works

Typical average-case operation costs:

| Collection | Access or operation | Typical complexity |
| --- | --- | --- |
| Array | Index | $O(1)$ |
| `List<T>` | Index | $O(1)$ |
| `List<T>` | Append | Amortized $O(1)$ |
| `List<T>` | Middle insert/remove | $O(n)$ |
| `Dictionary<TKey, TValue>` | Lookup | Average $O(1)$ |
| `HashSet<T>` | Membership | Average $O(1)$ |
| `SortedSet<T>` | Membership/update | $O(\log n)$ |
| `SortedDictionary<TKey, TValue>` | Lookup/update | $O(\log n)$ |
| `Queue<T>` or `Stack<T>` | Add/remove | Amortized $O(1)$ |
| `LinkedList<T>` | Known-node insert/remove | $O(1)$ |
| `LinkedList<T>` | Search | $O(n)$ |

Average hash-table costs depend on comparer quality and distribution. Always state assumptions when explaining complexity.

## Application

Use complexity to explain collection choices, identify repeated scans, estimate scaling behavior, and decide what to measure. Analyze both the outer algorithm and the collection operations it invokes.

## Common Mistakes

- Calling every dictionary operation guaranteed $O(1)$.
- Ignoring sorting or materialization hidden inside a query.
- Treating Big-O as a benchmark result.
- Forgetting auxiliary memory and allocation costs.
- Missing nested loops or repeated enumeration.

## Common Interview Questions

### Basic

- What does $O(1)$ mean?
- What is the complexity of array indexing?
- What is the average complexity of dictionary lookup?

### Intermediate

- Why is list append amortized constant time?
- Why is inserting in the middle of a list linear?
- How do space complexity and time complexity differ?

### Advanced

- How do cache locality and constant factors change practical performance?
- How would you analyze a LINQ query's complexity?
- How can repeated enumeration turn a linear operation into quadratic behavior?
- How do worst-case hash collisions affect complexity?
- How would you benchmark an asymptotically better algorithm?

### Follow-up Questions

- What is amortized complexity?
- What is auxiliary space?
- Why can an $O(n)$ algorithm beat an $O(\log n)$ algorithm in practice?

### Code Prediction

What is the complexity of this operation?

```csharp
var result = values.Where(value => value > 0).ToList();
```

## Practical Tasks

- Annotate common collection operations with time and space complexity.
- Find a hidden repeated enumeration in a method.
- Benchmark two implementations and compare results with their complexity claims.

## Readiness Criteria

You should be able to state realistic complexity assumptions, include auxiliary memory, identify hidden work, and distinguish asymptotic analysis from measurement.

## References

### Microsoft Learn

- [Selecting a collection class](https://learn.microsoft.com/dotnet/standard/collections/selecting-a-collection-class)
- [Big O notation](https://learn.microsoft.com/dotnet/standard/collections/)
