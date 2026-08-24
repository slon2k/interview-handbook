# Hashing and Dictionary Keys

## Definition

Hashing maps a key to an integer used to locate a bucket in a hash-based collection. A dictionary key must have stable equality and hashing behavior while it is stored. See [Equality Comparers in Collections](equality-comparers.md) for comparer selection.

```csharp
var values = new Dictionary<(string Country, int Number), string>();
values[("UK", 1)] = "one";
```

## Alternatives & Trade-offs

Use immutable keys such as strings, numbers, records with stable state, or tuples. A custom key type can improve clarity and performance, but it must implement equality and hashing correctly.

Hash collisions are expected and handled by the collection. A hash code is a lookup aid, not a unique identity or a security guarantee.

## How It Works

The collection computes a key hash, selects a bucket, and then uses equality to distinguish keys that share a bucket. Average lookup is $O(1)$, but poor distribution or adversarial input can degrade performance.

## Application

- Use immutable keys for caches and indexes.
- Prefer records or value objects for meaningful composite keys.
- Choose comparers explicitly for string-keyed collections.
- Treat hash codes as process-local implementation details unless a type contract says otherwise.

## Common Mistakes

- Persisting hash codes as stable identifiers.
- Using mutable objects as keys.
- Omitting fields from equality or hashing.
- Assuming different keys always have different hash codes.
- Using a cryptographic hash when ordinary collection hashing is sufficient, or vice versa.

## Common Interview Questions

### Basic

- What is a hash code?
- Why can two different keys have the same hash code?
- Why must dictionary keys be stable?

### Intermediate

- How are collisions handled?
- What makes a good dictionary key?
- How do records and tuples help with composite keys?

### Advanced

- How can adversarial keys affect hash-table behavior?
- Why should hash codes generally not be persisted?
- How do randomized string hashes affect reproducibility?
- How would you design a key for a multi-tenant cache?
- What is the difference between collection hashing and cryptographic hashing?

### Follow-up Questions

- Can `GetHashCode()` return a unique value for every object?
- What happens if equality changes after insertion?
- Should a database ID use `GetHashCode()`?

### Code Prediction

What happens after the key is mutated?

```csharp
var key = new MutableKey(1);
var values = new Dictionary<MutableKey, string> { [key] = "value" };
key.Id = 2;
Console.WriteLine(values.ContainsKey(key));
```

## Practical Tasks

- Create an immutable composite cache key.
- Demonstrate a lookup failure caused by mutable key state.
- Test collision handling with a deliberately poor comparer.

## Readiness Criteria

Explain buckets, collisions, stable key state, equality contracts, and why ordinary hash codes are unsuitable as persistent or cryptographic identifiers.

## References

### Microsoft Learn

- [Object.GetHashCode method](https://learn.microsoft.com/dotnet/api/system.object.gethashcode)
- [Hash-based collections](https://learn.microsoft.com/dotnet/standard/collections/)
- [Records](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/record)
