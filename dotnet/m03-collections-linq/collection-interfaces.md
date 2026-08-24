# Read-Only and Mutable Collection Interfaces

## Definition

Collection interfaces communicate the operations an API permits without exposing its concrete storage. Common interfaces include `IEnumerable<T>`, `IReadOnlyCollection<T>`, `IReadOnlyList<T>`, `ICollection<T>`, `IList<T>`, and `ISet<T>`. See [Selecting the Correct Collection](selecting-collections.md) when choosing the underlying collection.

```csharp
public IReadOnlyList<string> GetNames() => names;
public void AddNames(ICollection<string> destination) { }
```

## Alternatives & Trade-offs

Expose the narrowest interface that satisfies the caller. `IEnumerable<T>` communicates enumeration only; `IReadOnlyList<T>` adds count and indexing; mutable interfaces permit callers to change the collection.

Returning an interface does not automatically make the underlying collection immutable. A caller may still mutate the original collection through another reference.

## How It Works

Interfaces define capabilities rather than storage. A concrete collection can implement several interfaces, and adapters such as `AsReadOnly` can restrict one access path while retaining the original mutable owner.

`IEnumerable<T>` may represent a deferred sequence rather than materialized data, so enumeration can execute code or observe later mutations.

## Application

- Return `IReadOnlyList<T>` for stable, indexed results.
- Return `IReadOnlyCollection<T>` when count matters but indexing does not.
- Accept `IEnumerable<T>` when streaming or deferred enumeration is acceptable.
- Accept `ICollection<T>` or `ISet<T>` only when mutation is part of the contract.
- Use immutable interfaces or copies when ownership must be isolated.

## Common Mistakes

- Returning `List<T>` from every public method.
- Assuming `IReadOnlyList<T>` guarantees deep immutability.
- Enumerating an `IEnumerable<T>` multiple times without considering cost or side effects.
- Accepting `IEnumerable<T>` when random access is required.
- Exposing a mutable collection through a read-only wrapper while retaining unclear ownership.

## Common Interview Questions

### Basic

- What is the difference between `IEnumerable<T>` and `ICollection<T>`?
- What does `IReadOnlyList<T>` add?
- Why should APIs expose interfaces?

### Intermediate

- Does `IReadOnlyCollection<T>` make the underlying collection immutable?
- When should a method accept `IEnumerable<T>` versus `IReadOnlyList<T>`?
- What is the difference between `IList<T>` and `IReadOnlyList<T>`?

### Advanced

- How do interface choices affect deferred execution and ownership?
- When should an API return an immutable collection instead of a read-only interface?
- How can repeated enumeration create hidden performance or correctness problems?
- How do covariance rules affect read-only collection interfaces?
- How would you design an API that supports streaming and materialized callers?

### Follow-up Questions

- Can an `IEnumerable<T>` be enumerated more than once?
- Is `IReadOnlyList<T>` covariant?
- How do you prevent callers from modifying returned data?

### Code Prediction

Can the caller mutate the original list through this return type?

```csharp
IReadOnlyList<int> values = mutableList;
```

## Practical Tasks

- Refactor public methods to expose the narrowest useful interface.
- Demonstrate the difference between a read-only wrapper, a copy, and an immutable collection.
- Review an API that unintentionally exposes mutable state.

## Readiness Criteria

You should be able to choose collection interfaces based on capabilities, explain ownership and immutability limits, and identify deferred enumeration risks.

## References

### Microsoft Learn

- [IEnumerable<T> interface](https://learn.microsoft.com/dotnet/api/system.collections.generic.ienumerable-1)
- [IReadOnlyCollection<T> interface](https://learn.microsoft.com/dotnet/api/system.collections.generic.ireadonlycollection-1)
- [IReadOnlyList<T> interface](https://learn.microsoft.com/dotnet/api/system.collections.generic.ireadonlylist-1)
- [ICollection<T> interface](https://learn.microsoft.com/dotnet/api/system.collections.generic.icollection-1)
