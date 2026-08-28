# Common Collection and LINQ Performance Issues

## Definition

Module 3 covered collection choice and LINQ deferred execution conceptually; this topic covers the specific, recurring performance mistakes that show up once those constructs are used carelessly in hot paths — repeated enumeration, wrong collection choice for the access pattern, and LINQ chains that do more work than necessary.

```csharp
// Enumerates the same IEnumerable<T> multiple times, potentially re-running expensive work each time
var query = orders.Where(o => o.Total > 100);
var count = query.Count();      // enumerates once
var list = query.ToList();      // enumerates AGAIN, from scratch, if `orders` is a lazy source (e.g., an EF Core IQueryable)
```

## Alternatives & Trade-offs

Writing LINQ fluently and idiomatically is usually the right default — clarity matters, and premature manual-loop optimization of code that isn't a measured bottleneck wastes effort. The trade-off appears specifically in hot paths or over large datasets, where repeated enumeration of a lazy sequence, an `O(n)` lookup where an `O(1)` one was available, or an unnecessarily long LINQ chain becomes a measurable cost worth addressing deliberately.

## How It Works

### Repeated enumeration of a deferred sequence

```csharp
IEnumerable<Order> query = context.Orders.Where(o => o.Total > 100); // IQueryable<T>, not yet executed

var count = query.Count();  // executes a COUNT query against the database
var list = query.ToList();  // executes the FULL query against the database AGAIN — a second round trip
```

Materializing once and reusing the materialized result avoids the duplicated work:

```csharp
var list = await query.ToListAsync(); // execute once
var count = list.Count;                // reuse the already-materialized result, no second query
```

### Wrong collection for the access pattern

```csharp
// O(n) lookup for every check, if orders is a List<T>
var order = orders.FirstOrDefault(o => o.Id == targetId);

// O(1) lookup if the data is already indexed by key (Module 3's Dictionary<TKey,TValue>)
var ordersById = orders.ToDictionary(o => o.Id);
var order = ordersById.GetValueOrDefault(targetId);
```

Building the dictionary once costs `O(n)`, but if many lookups follow, the amortized cost is far lower than repeating an `O(n)` scan for every single lookup — the same complexity-analysis reasoning from Module 3, applied to an actual performance investigation rather than an abstract exercise.

### `Count()` vs. `Any()` — doing only the work actually needed

```csharp
// Potentially enumerates the ENTIRE sequence just to get a count, when only presence matters
if (orders.Where(o => o.Status == "Pending").Count() > 0) { }

// Stops at the first match — often significantly cheaper for large sequences
if (orders.Any(o => o.Status == "Pending")) { }
```

### Excessive intermediate allocations from a long LINQ chain

```csharp
// Each intermediate LINQ operator can introduce its own enumerator/allocation overhead
var result = orders.Where(o => o.Total > 100).Select(o => o.Id).OrderBy(id => id).Take(10).ToList();
```

For most code, this overhead is negligible and the readability is worth it — but for a hot path processing millions of items per second, a hand-written loop doing the same work in one pass can meaningfully reduce allocation and iteration overhead, a trade-off only worth making where measurement justifies it.

## Application

Materialize a LINQ query once and reuse the result rather than re-enumerating a lazy source multiple times. Choose the collection type matching the actual access pattern (Module 3), especially for repeated lookups. Prefer `Any()` over `Count() > 0` for existence checks. Reserve hand-written-loop rewrites of LINQ chains for genuinely measured hot paths, not as a default style choice.

## Common Mistakes

- Enumerating the same `IQueryable<T>`/lazy `IEnumerable<T>` multiple times, causing repeated (and for EF Core, repeated database) execution.
- Using `List<T>.FirstOrDefault` for repeated key-based lookups instead of building a `Dictionary<TKey, TValue>` once.
- Using `Count() > 0` instead of `Any()` for existence checks, potentially enumerating far more of a sequence than necessary.
- Rewriting readable LINQ into manual loops preemptively, without measurement showing the specific code path is actually a meaningful bottleneck.

## Common Interview Questions

### Basic
- Why can enumerating an `IQueryable<T>` multiple times be a performance problem specifically for EF Core-backed queries?
- Why is `Any()` generally preferred over `Count() > 0` for existence checks?

### Intermediate
- How would you fix a piece of code that re-enumerates the same lazy sequence multiple times?
- When would converting a list to a `Dictionary<TKey, TValue>` for repeated lookups be worth the upfront cost?

### Advanced
- How would you decide whether a specific LINQ-heavy hot path is worth rewriting as a manual loop, versus leaving it as idiomatic LINQ?
- How does repeated enumeration of an `IQueryable<T>` interact with the query-translation concepts from Module 10?

### Follow-up Questions
- Does calling `.ToList()` on a query prevent all further re-execution risk for that data?
- Is `Any()` always faster than `Count() > 0`, or does it depend on the underlying sequence?

### Code Prediction
Given `var query = context.Orders.Where(o => o.Total > 100);` followed by both `query.Count()` and `query.ToList()` calls with no materialization in between, how many round trips to the database does this code make, and why?

## Practical Tasks

- Identify and fix a case of repeated enumeration of a lazy/deferred sequence in a sample codebase.
- Replace a repeated `List<T>` linear-scan lookup pattern with a `Dictionary<TKey, TValue>`-based one, and measure the difference for a large dataset.
- Benchmark `Count() > 0` versus `Any()` for an early-terminating existence check on a large sequence.

## Readiness Criteria

Recognize and fix repeated enumeration of lazy sequences, choose collection types matching actual access patterns, and reserve manual-loop rewrites for genuinely measured hot paths.

## References

### Microsoft Learn

- [LINQ and deferred execution](https://learn.microsoft.com/dotnet/csharp/programming-guide/concepts/linq/deferred-execution-example)
- [How query execution works (EF Core)](https://learn.microsoft.com/ef/core/querying/how-query-execution-works)
