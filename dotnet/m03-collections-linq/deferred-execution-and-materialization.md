# Deferred Execution and Materialization

## Definition

Deferred execution means a query is defined now but its work occurs later, usually when it is enumerated. Materialization executes a sequence and stores its results in a collection such as a list or array.

```csharp
var query = values.Where(value => value > 0); // deferred
var result = query.ToList();                  // materialized
```

## Alternatives & Trade-offs

Deferred queries can reduce unnecessary work and support streaming. Materialization provides a stable snapshot, enables repeated access, and prevents the source from being re-evaluated, but it uses memory and performs work immediately.

## How It Works

A deferred query generally creates an object that wraps its source and predicate. Each enumeration walks the source and applies the operators. Materializing with `ToList` or `ToArray` consumes the source once and stores the result.

Some operators are streaming, such as `Where` and `Select`; others require buffering or full consumption, such as `OrderBy` and usually `GroupBy`.

### Streaming Execution

Streaming execution means an operator can yield an item as soon as it has enough information to produce it, without loading the entire source into memory first. `Where`, `Select`, `Take`, and `Skip` are common examples for in-memory sequences.

```csharp
IEnumerable<int> positive = values
	.Where(value => value > 0)
	.Select(value => value * 2)
	.Take(10);
```

The pipeline remains deferred and can process only the items needed by `Take`. Streaming does not guarantee that the source itself is fast, finite, or memory-free; it may still perform I/O or hold state between elements.

### Buffering Execution

Buffering operators must inspect or retain more of the source before they can yield reliable results. `OrderBy` must see all elements before returning the first sorted element, and `GroupBy` generally retains groups while consuming the source.

Streaming and deferred execution are related but distinct concepts: a query can be deferred but buffering, and a query can be executed immediately while still processing a source incrementally.

## Application

- Materialize when results will be enumerated repeatedly.
- Keep a query deferred when the source is cheap, current data is desired, or streaming matters.
- Materialize before disposing a source that the query depends on.
- Use `ToList` or `ToArray` at a clear boundary rather than throughout a pipeline.

## Common Mistakes

- Assuming assigning a query executes it.
- Re-enumerating a database, file, or network source.
- Returning a deferred query over a disposed context or stream.
- Materializing large data sets without filtering first.
- Modifying the source between query creation and enumeration unexpectedly.

## Common Interview Questions

### Basic

- What is deferred execution?
- What does `ToList` do?
- When does a `Where` query execute?

### Intermediate

- Which operators stream and which buffer?
- Why can repeated enumeration be expensive?
- When should a method return a materialized collection?

### Advanced

- How can deferred execution change correctness when the source mutates?
- How do buffering operators affect memory complexity?
- How would you preserve a snapshot while avoiding unnecessary copies?
- How can deferred work outlive the lifetime of its data source?
- How do multiple enumeration analyzers identify likely defects?

### Follow-up Questions

- Does `ToList` make elements immutable?
- Does `Count()` always enumerate the entire source?
- What happens if a deferred query throws?

### Code Prediction

What values are printed?

```csharp
var values = new List<int> { 1, 2 };
var query = values.Where(value => value > 1);
values.Add(3);
Console.WriteLine(string.Join(",", query));
```

## Practical Tasks

- Demonstrate source mutation before and after enumeration.
- Identify buffering operators in a query.
- Fix a method that returns a query over a disposed resource.
- Compare repeated enumeration with one materialization.

## Readiness Criteria

Explain when queries execute, distinguish streaming from buffering, choose materialization deliberately, and identify lifetime, mutation, memory, and repeated-enumeration risks.

## References

### Microsoft Learn

- [Deferred execution and lazy evaluation](https://learn.microsoft.com/dotnet/standard/linq/deferred-execution-lazy-evaluation)
- [Enumerable.ToList<TSource>](https://learn.microsoft.com/dotnet/api/system.linq.enumerable.tolist-1)
- [Enumerable.ToArray<TSource>](https://learn.microsoft.com/dotnet/api/system.linq.enumerable.toarray-1)
