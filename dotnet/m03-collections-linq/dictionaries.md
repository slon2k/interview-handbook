# `Dictionary<TKey, TValue>`

## Definition

`Dictionary<TKey, TValue>` stores values associated with unique keys and provides average $O(1)$ lookup, insertion, and removal when hashing distributes keys well.

```csharp
var scores = new Dictionary<string, int>
{
    ["Ada"] = 10
};
```

## Alternatives & Trade-offs

Use a dictionary for unsorted key-based lookup. Use `SortedDictionary<TKey, TValue>` for sorted data with frequent updates, or `SortedList<TKey, TValue>` for sorted, mostly-read data where compact array storage is useful. Use `Lookup<TKey, TValue>` when one key maps naturally to multiple immutable results.

## How It Works

The dictionary hashes a key and uses an equality comparer to locate a bucket. Correct key behavior requires consistent equality and hashing. Collisions are handled internally; poor hash distribution can degrade performance.

`SortedDictionary<TKey, TValue>` is tree-backed, with $O(\log n)$ lookup, insertion, and removal. `SortedList<TKey, TValue>` uses sorted arrays, with $O(\log n)$ lookup but $O(n)$ insertion and removal because elements may need to move. A `Lookup<TKey, TValue>` is typically created by LINQ grouping and is useful for one-to-many read-only access.

`CollectionsMarshal.GetValueRefOrAddDefault` exposes a reference to dictionary storage. It is an advanced optimization: the reference must not outlive operations that can resize the dictionary, and ordinary indexer or `TryGetValue` code is preferable unless profiling justifies it.

## Application

Use dictionaries for indexes, caches, configuration maps, frequency counts, and joins by key.

## Common Mistakes

- Mutating a key after insertion.
- Assuming enumeration order is a business contract.
- Using `ContainsKey` followed by an indexer when `TryGetValue` is clearer.
- Using a culture-sensitive comparer for identifiers.
- Ignoring duplicate-key behavior during construction.

## Common Interview Questions

### Basic

- What is a dictionary?
- What is the role of a key?
- What is the average lookup complexity?

### Intermediate

- How do equality and hashing affect dictionary keys?
- What is the difference between `TryGetValue` and the indexer?
- When would you use a custom comparer?

### Advanced

- How do collisions affect dictionary performance?
- Why must a key's hash remain stable while stored?
- How do capacity and resizing affect allocations?
- When is `CollectionsMarshal.GetValueRefOrAddDefault` appropriate?
- How should dictionaries be designed for concurrent access?
- When should you choose `SortedList<TKey, TValue>` over `SortedDictionary<TKey, TValue>`?

### Follow-up Questions

- Can a dictionary contain a null key?
- Does a dictionary preserve insertion order as a contract?
- What happens when a key is added twice?

### Code Prediction

What is printed?

```csharp
var values = new Dictionary<string, int> { ["a"] = 1 };
values["a"] = 2;
Console.WriteLine(values.Count);
Console.WriteLine(values["a"]);
```

## Practical Tasks

- Build a word-frequency counter with an explicit comparer.
- Demonstrate the failure caused by mutating a key.
- Compare dictionary lookup with repeated list search.
- Create a `Lookup<TKey, TValue>` from a sequence and compare it with a dictionary of lists.

## Readiness Criteria

Explain hashing, equality, lookup complexity, comparer selection, resizing, key immutability, and safe lookup patterns.

## References

### Microsoft Learn

- [Dictionary<TKey, TValue> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.dictionary-2)
- [Lookup<TKey, TValue> class](https://learn.microsoft.com/dotnet/api/system.linq.lookup-2)
- [SortedDictionary<TKey, TValue> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.sorteddictionary-2)
- [SortedList<TKey, TValue> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.sortedlist-2)
- [Equality comparer guidance](https://learn.microsoft.com/dotnet/api/system.collections.generic.iequalitycomparer-1)
