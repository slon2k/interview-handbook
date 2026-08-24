# `HashSet<T>`

## Definition

`HashSet<T>` stores unique elements and is optimized for membership tests, set operations, insertion, and removal.

```csharp
var tags = new HashSet<string>(StringComparer.OrdinalIgnoreCase)
{
    "dotnet"
};
```

## Alternatives & Trade-offs

Use a hash set when uniqueness and membership are central. Use `List<T>` when order and duplicates matter, or `SortedSet<T>` when sorted uniqueness is required. Use `FrozenSet<T>` for .NET 8+ build-once, read-heavy membership data.

## How It Works

A set uses hashing and an equality comparer. `Add` returns `false` when an equivalent element already exists. Average membership, insertion, and removal are $O(1)$ when hashing distributes elements well; enumeration is proportional to the number of stored elements.

## Application

Use sets for deduplication, visited-node tracking, permissions, feature flags, and fast membership checks.

## Common Mistakes

- Expecting duplicate values to be retained.
- Choosing the wrong comparer for strings.
- Mutating an element so its hash changes while stored.
- Using a set when stable order is required.
- Assuming set operations preserve input order.

## Common Interview Questions

### Basic

- What is a `HashSet<T>`?
- How does it differ from a list?
- What does `Add` return?

### Intermediate

- How do equality and hashing determine uniqueness?
- What are union, intersection, and difference?
- When would you use `SortedSet<T>`?

### Advanced

- How do collision patterns affect membership performance?
- Why must stored elements have stable equality behavior?
- How would you implement a set with domain-specific equality?
- How do set operations allocate and scale?
- How can a set support graph traversal efficiently?

### Follow-up Questions

- Does a set guarantee insertion order?
- Can a set contain null?
- Is `HashSet<T>` thread-safe?

### Code Prediction

What is printed?

```csharp
var values = new HashSet<int> { 1, 1, 2 };
Console.WriteLine(values.Count);
```

## Practical Tasks

- Deduplicate input while selecting an explicit comparer.
- Implement common set operations and explain their complexity.
- Use a set to prevent revisiting nodes in a traversal.

## Readiness Criteria

Explain uniqueness, hashing, comparer choice, set operations, complexity, and the consequences of mutating stored values.

## References

### Microsoft Learn

- [HashSet<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.hashset-1)
- [Set collections](https://learn.microsoft.com/dotnet/standard/collections/)
