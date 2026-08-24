# `IEnumerable<T>` and Iterators

## Definition

`IEnumerable<T>` represents a sequence that can be enumerated. Its `GetEnumerator` method provides an enumerator that moves through the sequence and exposes the current element.

```csharp
IEnumerable<int> values = [1, 2, 3];

foreach (int value in values)
{
    Console.WriteLine(value);
}
```

An iterator method uses `yield return` to produce elements lazily.

```csharp
static IEnumerable<int> PositiveValues(IEnumerable<int> values)
{
    foreach (int value in values)
    {
        if (value > 0)
        {
            yield return value;
        }
    }
}
```

## Alternatives & Trade-offs

Use `IEnumerable<T>` for pull-based sequence processing. Use a materialized collection when repeated access, count, or indexing is required. Use `IAsyncEnumerable<T>` for asynchronous streaming.

Iterators reduce upfront work and memory usage, but execution and exceptions occur during enumeration rather than when the method is called.

## How It Works

The compiler transforms an iterator method into a state machine. Each call to `MoveNext` advances the state machine and produces the next value. The sequence may execute again every time it is enumerated.

An enumerator should be disposed, which `foreach` handles automatically. An iterator can use `yield break` to end the sequence.

## Application

- Stream data without materializing the entire result.
- Compose filters and transformations.
- Encapsulate traversal logic.
- Use `IAsyncEnumerable<T>` when producing values requires asynchronous work.

## Common Mistakes

- Assuming an `IEnumerable<T>` is already materialized.
- Enumerating a sequence multiple times when it performs expensive work.
- Modifying a collection while enumerating it.
- Hiding I/O or side effects inside an iterator.
- Returning `IEnumerable<T>` when the API requires stable repeated results.

## Common Interview Questions

### Basic

- What is `IEnumerable<T>`?
- What does `yield return` do?
- What is an enumerator?

### Intermediate

- When does an iterator method execute?
- What does `foreach` do internally?
- What is the difference between `IEnumerable<T>` and `ICollection<T>`?

### Advanced

- How does the compiler-generated iterator state machine work?
- How do disposal and `yield return` interact?
- What are the performance costs of interface-based enumeration?
- How can repeated enumeration change observable behavior?
- When should an iterator be replaced by a materialized result or async stream?

### Follow-up Questions

- When are iterator exceptions thrown?
- Can an iterator yield null values?
- What happens when enumeration stops early?

### Code Prediction

When is the message printed?

```csharp
static IEnumerable<int> Values()
{
    Console.WriteLine("started");
    yield return 1;
}

IEnumerable<int> values = Values();
Console.WriteLine("created");
foreach (int value in values) { }
```

## Practical Tasks

- Implement an iterator that reads a sequence in pages.
- Demonstrate execution on creation versus enumeration.
- Compare a lazy iterator with a materialized list.

## Readiness Criteria

Explain enumeration, iterator state machines, deferred execution, disposal, repeated enumeration, and when a materialized or asynchronous sequence is more suitable.

## References

### Microsoft Learn

- [`IEnumerable<T>` interface](https://learn.microsoft.com/dotnet/api/system.collections.generic.ienumerable-1)
- [Iterators](https://learn.microsoft.com/dotnet/csharp/iterators)
- [Iterator pattern](https://learn.microsoft.com/dotnet/standard/design-guidelines/iterator-pattern)
