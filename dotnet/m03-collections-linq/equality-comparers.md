# Equality Comparers in Collections

## Definition

An equality comparer defines how values are compared and how their hash codes are produced. Sets and dictionaries use comparers to determine uniqueness and key identity.

```csharp
var users = new HashSet<string>(StringComparer.OrdinalIgnoreCase);
users.Add("Ada");
users.Add("ada");
```

## Alternatives & Trade-offs

Use a built-in comparer when it expresses the domain rule. Use a custom `IEqualityComparer<T>` for value-based identity that differs from the type's default equality. Avoid changing equality rules unexpectedly for an existing type. For the hashing side of the same contract, see [Hashing and Dictionary Keys](hashing-and-dictionary-keys.md).

## How It Works

A comparer must obey the contract that equal values produce equal hash codes. Comparisons should be stable while a key or element is stored. Dictionaries and sets use the comparer supplied at construction, not necessarily the type's default comparer.

## Application

- Use `StringComparer.OrdinalIgnoreCase` for case-insensitive identifiers.
- Use `StringComparer.Ordinal` for case-sensitive machine keys.
- Supply a domain comparer for composite keys or normalized values.
- Use `EqualityComparer<T>.Default` when the type's normal equality is intended.

## Common Mistakes

- Using `ToLower()` as a substitute for an explicit comparer; it also introduces culture-dependent behavior unless a culture is specified.
- Mixing culture-sensitive and ordinal comparison rules.
- Creating a comparer that disagrees with `Equals` and `GetHashCode`.
- Mutating fields used by equality after insertion.
- Assuming a comparer changes the stored values themselves.

## Common Interview Questions

### Basic

- What is an equality comparer?
- Why do dictionaries need hash codes?
- What does `StringComparer.OrdinalIgnoreCase` do?

### Intermediate

- What methods must a custom comparer implement?
- Why must equal objects have equal hash codes?
- How do you compare composite keys?

### Advanced

- How should comparer consistency be maintained across a process boundary?
- What are the risks of culture-sensitive key comparison?
- How can a comparer affect dictionary performance?
- When should equality be defined by the type versus supplied by the collection?
- How would you test a custom comparer for symmetry, transitivity, and hash consistency?

### Follow-up Questions

- Can unequal values share a hash code?
- What happens if a key changes after insertion?
- Which comparer should be used for protocol tokens?

### Code Prediction

What is the set count?

```csharp
var values = new HashSet<string>(StringComparer.OrdinalIgnoreCase)
{
    "A",
    "a"
};
Console.WriteLine(values.Count);
```

## Practical Tasks

- Implement a comparer for a composite key.
- Compare ordinal and culture-aware string behavior.
- Test a comparer against equality and hash-code invariants.

## Readiness Criteria

You should be able to select or implement a comparer, explain its equality and hashing contract, and identify culture, mutation, and performance risks.

## References

### Microsoft Learn

- [IEqualityComparer<T> interface](https://learn.microsoft.com/dotnet/api/system.collections.generic.iequalitycomparer-1)
- [StringComparer class](https://learn.microsoft.com/dotnet/api/system.stringcomparer)
- [EqualityComparer<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.equalitycomparer-1)
