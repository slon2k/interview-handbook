# LINQ Query Operators

## Definition

Language Integrated Query (LINQ) provides standard operators for filtering, projection, ordering, grouping, joining, aggregation, and testing sequences.

```csharp
var names = users
    .Where(user => user.IsActive)
    .Select(user => user.Name)
    .OrderBy(name => name);
```

## Alternatives & Trade-offs

LINQ improves composability and readability for sequence transformations. A loop may be clearer for complex stateful logic or may be easier to optimize in a measured hot path.

Most LINQ operators return an `IEnumerable<T>` and many execute lazily. Operators such as `ToList`, `ToArray`, `Count`, and `First` consume the sequence.

## How It Works

Operators compose by wrapping a source sequence. A pipeline is usually evaluated when its result is enumerated.

- `Where` filters elements.
- `Select` maps one element to one result.
- `SelectMany` maps one element to a sequence and flattens the results.
- `GroupBy` creates groups.
- `Join` matches keys from two sequences.
- `Aggregate`, `Sum`, `Min`, and `Max` reduce a sequence.
- `Any`, `All`, `Contains`, `First`, and `Single` inspect or retrieve results.

## Application

- Transform collections into view models.
- Filter and order search results.
- Group and aggregate reporting data.
- Flatten nested collections.
- Compose reusable in-memory query pipelines.

## Common Mistakes

- Using `Single` when multiple matches are valid.
- Using `First` when absence should be reported explicitly.
- Confusing `Select` with `SelectMany`.
- Calling `ToList` repeatedly inside a pipeline.
- Assuming a query runs when it is assigned to a variable.
- Performing multiple enumeration of an expensive source.

## Common Interview Questions

### Basic

- What is LINQ?
- What is the difference between `Where` and `Select`?
- What is the difference between `First` and `Single`?

### Intermediate

- What is the difference between `Select` and `SelectMany`?
- Which operators are deferred and which are immediate?
- How do grouping and joining work?
- What is the difference between `Any` and `Count() > 0`?

### Advanced

- How does operator composition affect enumeration complexity?
- How can operator ordering reduce work?
- What allocations are introduced by common LINQ pipelines?
- How do equality comparers affect `Distinct`, `GroupBy`, and joins?
- When should a loop replace a LINQ pipeline?
- How do in-memory LINQ and provider-translated LINQ differ?

### Follow-up Questions

- Does `Where` change the source collection?
- What happens when `Single` finds two matches?
- When does `OrderBy` need to consume the entire sequence?
- Why can `Any` be faster than `Count() > 0`?

### Code Prediction

What is printed?

```csharp
var result = new[] { 1, 2, 3 }
    .Where(value => value > 1)
    .Select(value => value * 2);

Console.WriteLine(string.Join(",", result));
```

## Practical Tasks

- Transform a collection into a grouped report.
- Rewrite a nested projection using `SelectMany`.
- Compare `Any`, `Count`, `First`, and `Single` for several requirements.
- Review a LINQ pipeline for unnecessary materialization.

## Readiness Criteria

You should be able to compose common operators, choose the correct terminal operation, explain execution timing, and identify complexity, allocation, and provider-translation concerns.

## References

### Microsoft Learn

- [LINQ](https://learn.microsoft.com/dotnet/csharp/linq/)
- [Standard query operators overview](https://learn.microsoft.com/dotnet/csharp/linq/standard-query-operators/)
- [Enumerable class](https://learn.microsoft.com/dotnet/api/system.linq.enumerable)
