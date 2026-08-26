# Eager, Explicit, and Lazy Loading

## Definition

These are the three ways EF Core can load related data: **eager loading** (`Include()`) fetches related entities as part of the initial query; **explicit loading** fetches related entities on demand via a separate, deliberate call after the main entity is already loaded; **lazy loading** automatically fetches related entities the moment a navigation property is accessed, transparently issuing a query behind the scenes.

```csharp
// Eager
var orders = await context.Orders.Include(o => o.Customer).ToListAsync();

// Explicit
var order = await context.Orders.FirstAsync(o => o.Id == 42);
await context.Entry(order).Reference(o => o.Customer).LoadAsync();

// Lazy (requires proxies enabled and virtual navigation properties)
var order = await context.Orders.FirstAsync(o => o.Id == 42);
var name = order.Customer.Name; // triggers a query automatically, the moment Customer is accessed
```

## Alternatives & Trade-offs

Eager loading front-loads exactly what you need in a controlled number of queries, making performance predictable and explicit in the code. Explicit loading defers the decision until you actually know you need the related data, useful for conditional logic. Lazy loading is the most convenient to write (you just access the property) but the most dangerous — it hides exactly when and how many queries fire, which is the direct cause of the N+1 problem covered in the next topic.

## How It Works

### Eager loading — one query, joined or split (see `split-vs-single-queries.md`)

```csharp
var orders = await context.Orders
    .Include(o => o.Customer)
    .Include(o => o.Items).ThenInclude(i => i.Product) // nested Include for multi-level relationships
    .ToListAsync();
```

### Explicit loading — load related data only when actually needed

```csharp
var order = await context.Orders.FirstAsync(o => o.Id == 42);
if (needsCustomerDetails)
{
    await context.Entry(order).Reference(o => o.Customer).LoadAsync(); // separate query, only if the condition holds
}
```

### Lazy loading and why it's risky by default

```csharp
public class Order
{
    public virtual Customer Customer { get; set; } = null!; // must be virtual for lazy-loading proxies to intercept it
}

builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseLazyLoadingProxies().UseSqlServer(connectionString)); // opt-in, not the default
```

```csharp
var orders = await context.Orders.ToListAsync(); // one query
foreach (var order in orders)
{
    Console.WriteLine(order.Customer.Name); // with lazy loading, this fires ONE query per order — N+1
}
```

Lazy loading turns an innocent-looking `foreach` loop into N additional queries with no visual signal in the code that anything expensive is happening — this is precisely why it's opt-in in EF Core rather than the default, unlike some other ORMs.

## Application

Default to eager loading (`Include()`) for known, fixed data-access patterns — it's explicit in the code about what's being fetched and when. Use explicit loading for genuinely conditional related-data needs. Avoid lazy loading in server-side, high-throughput applications specifically because it hides query cost inside ordinary property access, making N+1 bugs easy to introduce accidentally.

## Common Mistakes

- Enabling lazy loading in a web API for convenience, then unknowingly triggering N+1 queries through ordinary-looking property access in a loop or serializer.
- Using eager loading (`Include`) to fetch large related collections that are never actually used by the calling code, wasting bandwidth and memory.
- Forgetting `ThenInclude` for multi-level relationships, silently leaving a deeper navigation property unloaded (`null` or empty) even though a shallower one was eagerly loaded.
- Mixing loading strategies inconsistently across a codebase without a clear convention, making query behavior unpredictable from one endpoint to the next.

## Common Interview Questions

### Basic
- What are the three EF Core loading strategies?
- Why is lazy loading opt-in rather than the default in EF Core?

### Intermediate
- What's the difference between eager and explicit loading, and when would you choose one over the other?
- What requirement must a navigation property satisfy to support lazy loading?

### Advanced
- How does lazy loading specifically cause the N+1 problem, in terms of when and how many queries actually fire?
- How would you audit a codebase for accidental lazy-loading-triggered N+1 queries, given that the code "looks fine" on the surface?

### Follow-up Questions
- Can eager and explicit loading be combined in the same query workflow?
- Does `Include()` load a collection navigation property differently from a reference navigation property?

### Code Prediction
Given lazy loading enabled and a `foreach` loop over 100 orders accessing `order.Customer.Name` inside the loop, how many total SQL queries execute — 1, 2, or 101? Why?

## Practical Tasks

- Convert a lazy-loading-enabled codebase suffering from N+1 queries to use explicit `Include()` calls instead, and verify the query count drops.
- Implement explicit loading for a genuinely conditional related-data scenario.
- Use `Include().ThenInclude()` to eagerly load a two-level-deep relationship and verify both levels are populated.

## Readiness Criteria

Choose the correct loading strategy for a given scenario, explain precisely why lazy loading causes N+1 queries, and use `Include`/`ThenInclude` correctly for multi-level relationships.

## References

### Microsoft Learn

- [Loading related data](https://learn.microsoft.com/ef/core/querying/related-data/)
- [Lazy loading](https://learn.microsoft.com/ef/core/querying/related-data/lazy)
