# N+1 Queries

## Definition

The N+1 problem occurs when code executes one query to fetch a list of N items, then executes one additional query per item to fetch related data — N+1 total queries where a single, properly-joined query (or a single eager load) would have sufficed. It's one of the most common real-world EF Core performance issues, and one of the easiest to introduce without noticing.

```csharp
var orders = await context.Orders.ToListAsync();       // query #1
foreach (var order in orders)
{
    var customer = order.Customer;                       // triggers one query PER order, if lazy loading is enabled
}
// Total: 1 + N queries, where N is the number of orders
```

## Alternatives & Trade-offs

N+1 is never intentional — it's always a bug, arising from either lazy loading or forgetting eager loading. The fix (`Include()`, or a projection with a joined `Select()`) trades nothing meaningful for correctness: a single well-formed query with a join is strictly better than N+1 round-trips to the database, both in total data transferred and in latency (N round-trips each pay full network latency, not just query execution time).

## How It Works

### The bug, made visible with logging

```csharp
var orders = await context.Orders.ToListAsync(); // SELECT * FROM Orders  (1 query)
foreach (var order in orders)
{
    Console.WriteLine(order.Customer.Name);
    // SELECT * FROM Customers WHERE Id = @p0   -- fires once per order, with lazy loading enabled
}
// For 500 orders: 501 total queries instead of 1
```

Enabling EF Core's query logging (`.LogTo(Console.WriteLine)` in development) is the single most effective way to actually *see* an N+1 problem happening, since the code itself looks completely unremarkable.

### The fix: eager loading

```csharp
var orders = await context.Orders.Include(o => o.Customer).ToListAsync();
foreach (var order in orders)
{
    Console.WriteLine(order.Customer.Name); // no additional query — already loaded
}
// Total: 1 query, with a JOIN to Customers
```

### The fix: projection

```csharp
var summaries = await context.Orders
    .Select(o => new { o.Id, CustomerName = o.Customer.Name })
    .ToListAsync();
// Total: 1 query, selecting only the needed columns via a JOIN
```

### N+1 can hide behind serialization, not just an obvious loop

```csharp
[HttpGet]
public async Task<IActionResult> Get() =>
    Ok(await context.Orders.ToListAsync()); // returns entities directly; JSON serialization then
                                              // walks every navigation property, triggering lazy loads
```

If lazy loading is enabled and controller actions return entities directly to be JSON-serialized, the serializer accessing every navigation property during serialization can trigger N+1 queries with no `foreach` loop visible anywhere in the application code at all — the loop is hidden inside the serializer.

## Application

Watch for N+1 anywhere a collection of entities is iterated (or serialized) and a related navigation property is accessed per item. Fix it with `Include()` for full related entities needed across the whole set, or a projecting `Select()` when only specific related fields are needed. Enable query logging in development specifically to catch N+1 patterns before they reach production, since the code alone often doesn't reveal the problem.

## Common Mistakes

- Not noticing N+1 because the code "looks fine" — nothing about `order.Customer.Name` inside a loop visually signals that it might trigger a database round-trip.
- Returning entities directly from an API for JSON serialization with lazy loading enabled, hiding N+1 behind the serializer instead of an obvious loop.
- Fixing one N+1 instance with `Include()` but missing a second, similar pattern elsewhere in the same codebase, since each instance is a separate design decision, not centrally managed.
- Assuming disabling lazy loading alone eliminates N+1 risk — forgetting an `Include()` with explicit/eager loading disabled just produces `NullReferenceException`s instead of N+1 queries, not automatically the correct fix.

## Common Interview Questions

### Basic
- What is the N+1 problem?
- How would you detect an N+1 query problem in a running application?

### Intermediate
- How does eager loading fix N+1, and how does projection offer an alternative fix?
- Why can returning entities directly for JSON serialization hide an N+1 problem?

### Advanced
- How would you audit a large existing codebase for N+1 patterns without lazy loading enabled to reveal them via query count spikes?
- What's the performance difference, in terms of both data volume and network round-trips, between N+1 queries and one query with a join?

### Follow-up Questions
- Does disabling lazy loading automatically fix all N+1 problems?
- Can N+1 occur with explicit loading, not just lazy loading?

### Code Prediction
Given 1,000 orders and a `foreach` loop accessing `order.Customer.Name` with lazy loading enabled, roughly how many total SQL queries execute? What would the count become after adding `.Include(o => o.Customer)` to the original query?

## Practical Tasks

- Enable EF Core query logging, reproduce an N+1 pattern, and observe the query count in the log output.
- Fix a reproduced N+1 problem using both `Include()` and an equivalent projecting `Select()`, comparing the generated SQL.
- Audit a small codebase returning entities directly from API endpoints for hidden N+1 risk via serialization.

## Readiness Criteria

Recognize N+1 patterns in code that looks superficially fine, use query logging to detect them, and fix them with the appropriate loading or projection strategy.

## References

### Microsoft Learn

- [Related data performance considerations](https://learn.microsoft.com/ef/core/performance/efficient-querying#load-related-data)
- [Simple logging](https://learn.microsoft.com/ef/core/logging-events-diagnostics/simple-logging)
