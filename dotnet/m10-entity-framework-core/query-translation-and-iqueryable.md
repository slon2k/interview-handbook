# Query Translation and IQueryable&lt;T&gt;

## Definition

`DbSet<T>` implements `IQueryable<T>` (introduced in Module 3 as a contrast with `IEnumerable<T>`); EF Core's specific `IQueryable` provider translates the LINQ expression tree you build into SQL, only executing against the database when the query is materialized (`ToList`, `First`, enumeration, etc.). Not every LINQ expression can be translated — the provider must recognize the pattern and know how to express it in SQL.

```csharp
var query = context.Orders.Where(o => o.Total > 100).OrderBy(o => o.OrderDate); // builds an expression tree, no SQL yet
var results = await query.ToListAsync(); // NOW translated to SQL and executed
```

## Alternatives & Trade-offs

Deferred execution and translation mean you can compose a query from multiple LINQ calls across different parts of your code before it actually runs, letting EF Core translate the *final* composed query into one efficient SQL statement. The trade-off is that translation isn't magic — LINQ expressions using arbitrary C# methods the provider doesn't recognize either throw at translation time or, in older EF Core versions, silently fall back to client-side evaluation (pulling more data than intended and filtering in memory).

## How It Works

### Building a query without executing it

```csharp
IQueryable<Order> query = context.Orders; // no SQL yet
query = query.Where(o => o.CustomerId == 7); // still building the expression tree
query = query.OrderByDescending(o => o.OrderDate);
var results = await query.ToListAsync(); // only now does EF Core translate and execute
```

### What can and can't be translated

```csharp
// Translatable — EF Core recognizes these and expresses them directly in SQL
var recent = await context.Orders.Where(o => o.OrderDate >= DateTime.UtcNow.AddDays(-30)).ToListAsync();

// NOT translatable — a custom C# method the provider has no SQL equivalent for
var formatted = await context.Orders.Where(o => CustomFormat(o.Status) == "shipped-formatted").ToListAsync();
// throws InvalidOperationException in modern EF Core, since it can't be translated to SQL
```

Modern EF Core (5+) throws by default when a query can't be translated, rather than silently falling back to loading everything and filtering in memory (which older EF Core versions did, and which could badly hurt performance without any error to signal it).

### Forcing client-side evaluation deliberately

```csharp
// Materialize what CAN be translated first, then apply untranslatable logic in memory afterward
var candidates = await context.Orders.Where(o => o.CustomerId == 7).ToListAsync(); // SQL executes here
var formatted = candidates.Where(o => CustomFormat(o.Status) == "shipped-formatted").ToList(); // in-memory, on IEnumerable<T> now
```

Calling `ToList()` (or similar) switches from `IQueryable<T>` to `IEnumerable<T>` — everything after that point runs in application memory, not the database, which is sometimes exactly what's needed but should be a deliberate choice, not an accident.

### `EF.Functions` — accessing database-specific functionality from LINQ

```csharp
var results = await context.Orders
    .Where(o => EF.Functions.Like(o.CustomerName, "Jo%"))
    .ToListAsync(); // translates to a SQL LIKE, which plain string methods might not map to as directly
```

## Application

Compose `IQueryable<T>` queries across method boundaries to build up filtering/sorting logic, understanding that nothing executes until materialization. Materialize (`ToListAsync()`) as late as possible to let EF Core translate the fullest possible query, but materialize deliberately before applying any logic the provider can't translate to SQL.

## Common Mistakes

- Materializing a query too early (calling `ToList()` before applying all intended filters), forcing filtering logic to run in memory on `IEnumerable<T>` instead of being pushed down to SQL.
- Writing a query with an untranslatable expression and being confused by the resulting exception, instead of recognizing which part of the expression the provider can't handle.
- In older EF Core versions (pre-3.0), unknowingly relying on silent client-side evaluation for a filter, loading far more data than intended with no warning.
- Confusing `IQueryable<T>` (translated to the database) with `IEnumerable<T>` (executed in application memory) when reasoning about where a given LINQ operator actually runs.

## Common Interview Questions

### Basic
- What is query translation, and when does an `IQueryable<T>` query actually execute?
- What happens if a LINQ expression can't be translated to SQL in modern EF Core?

### Intermediate
- Why does materializing a query too early hurt performance?
- What's the difference in behavior once you call `.ToList()` on an `IQueryable<T>` — where does subsequent LINQ run?

### Advanced
- How would you diagnose whether a specific query is being fully translated to SQL, versus partially evaluated in memory?
- What is `EF.Functions`, and why is it needed for some database-specific operations?

### Follow-up Questions
- Does building up an `IQueryable<T>` across multiple method calls execute anything before it's materialized?
- Did older EF Core versions handle untranslatable expressions the same way modern versions do?

### Code Prediction
```csharp
var query = context.Orders.Where(o => o.Total > 100);
var filtered = query.Where(o => CustomBusinessRule(o)); // CustomBusinessRule is a plain C# method
var results = await filtered.ToListAsync();
```
Assuming `CustomBusinessRule` can't be translated to SQL, what happens when `ToListAsync()` is called — does it throw, or does it silently evaluate everything in memory? Does the answer depend on the EF Core version?

## Practical Tasks

- Build a composed `IQueryable<T>` query across two separate methods and confirm (via logging or profiling) that only one SQL statement executes on materialization.
- Reproduce an untranslatable-expression exception and fix it by materializing before applying the untranslatable logic.
- Use `EF.Functions` to express a database-specific filter that a plain LINQ/string method wouldn't translate correctly.

## Readiness Criteria

Explain deferred execution and query translation precisely, diagnose translation failures, and deliberately choose when to materialize a query relative to translatable versus untranslatable logic.

## References

### Microsoft Learn

- [How query execution works](https://learn.microsoft.com/ef/core/querying/how-query-execution-works)
- [Client vs. server evaluation](https://learn.microsoft.com/ef/core/querying/client-eval)
