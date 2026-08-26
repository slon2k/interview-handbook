# AsNoTracking

## Definition

`AsNoTracking()` tells EF Core not to record the original state of entities returned by a query, since they're never going to be modified and saved back. This skips the change-tracker bookkeeping entirely, reducing memory usage and improving performance for read-only queries.

```csharp
var orders = await context.Orders.AsNoTracking().Where(o => o.Status == "Shipped").ToListAsync();
```

## Alternatives & Trade-offs

A tracking query (the default) costs extra memory and CPU to snapshot each entity's original state, but lets you modify and save the results directly without any extra step. `AsNoTracking()` skips that cost entirely, but the returned entities can't be handed to `SaveChanges()` for updates without first re-attaching them — it's purely a read-path optimization, not a general-purpose default.

## How It Works

### The performance case for AsNoTracking

```csharp
// Tracking (default): EF Core snapshots each of these 10,000 rows for potential future changes
var report = await context.Orders.Where(o => o.OrderDate >= startDate).ToListAsync();

// No-tracking: no snapshot overhead, since these rows are only being read and displayed, never modified
var report = await context.Orders.AsNoTracking().Where(o => o.OrderDate >= startDate).ToListAsync();
```

For a reporting query returning thousands of rows purely for display, the tracking overhead is pure waste — nothing about those objects will ever be passed back to `SaveChanges()`.

### Attempting to modify a no-tracking entity does nothing

```csharp
var order = await context.Orders.AsNoTracking().FirstAsync(o => o.Id == 42);
order.Status = "Shipped";
await context.SaveChangesAsync(); // does NOT update the database — the context has no idea this entity changed
```

This is a common source of "my update silently didn't work" bugs — the mutation happens in memory on a plain object the context knows nothing about.

### Setting a default tracking behavior for a whole context

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder options) =>
    options.UseSqlServer(connectionString)
           .UseQueryTrackingBehavior(QueryTrackingBehavior.NoTracking); // no-tracking by default for every query
```

Useful for a context dedicated entirely to read/query operations (see CQRS-style read models), where every query using this context should opt out of tracking unless explicitly overridden with `.AsTracking()`.

### `AsNoTrackingWithIdentityResolution()` — no tracking, but still deduplicates identical entities

```csharp
var orders = await context.Orders
    .Include(o => o.Customer)
    .AsNoTrackingWithIdentityResolution()
    .ToListAsync();
// if multiple orders share the same customer, all of them reference the SAME Customer object instance,
// rather than separate, duplicate Customer objects for each order — while still skipping change tracking
```

## Application

Use `AsNoTracking()` for any query whose results are purely read for display, reporting, or serialization to an API response — this covers the large majority of `GET` endpoints. Reserve tracking queries for the cases where an entity is loaded specifically to be modified and saved. Consider setting a context-wide `NoTracking` default for a context dedicated to read paths.

## Common Mistakes

- Mutating an entity loaded with `AsNoTracking()` and expecting `SaveChanges()` to persist the change, when the context has no record of it at all.
- Using tracking queries by default even for purely read-only endpoints, paying unnecessary memory/CPU overhead at scale.
- Forgetting that `AsNoTracking()` still executes a full query against the database — it only skips change-tracking bookkeeping, not the query itself.
- Using `AsNoTracking()` on a query whose results *will* later be attached and modified, without realizing `Attach`-based updates require explicit `IsModified` flags (see `change-tracking.md`) since the tracker never saw the original state.

## Common Interview Questions

### Basic
- What does `AsNoTracking()` do, and when should you use it?
- What happens if you modify a property on a no-tracking entity and call `SaveChanges()`?

### Intermediate
- Why does `AsNoTracking()` improve performance for read-only queries?
- What is `AsNoTrackingWithIdentityResolution()`, and how does it differ from plain `AsNoTracking()`?

### Advanced
- How would you configure a `DbContext` dedicated to read-only reporting queries to use no-tracking by default?
- What's the risk of using `AsNoTracking()` for a query whose results will later be attached and saved?

### Follow-up Questions
- Does `AsNoTracking()` skip executing the query against the database?
- Can a no-tracking query still use `Include()` to load related entities?

### Code Prediction
```csharp
var order = await context.Orders.AsNoTracking().FirstAsync(o => o.Id == 42);
order.Status = "Shipped";
await context.SaveChangesAsync();
```
Does the database reflect `Status = "Shipped"` after this runs? Why or why not?

## Practical Tasks

- Convert a purely read-only reporting query to use `AsNoTracking()` and measure the memory/performance difference for a large result set.
- Reproduce the "silent no-op update" bug from modifying a no-tracking entity, then fix it correctly.
- Configure a dedicated read-only `DbContext` with `NoTracking` as its default query behavior.

## Readiness Criteria

Use `AsNoTracking()` appropriately for read-only queries, explain why it doesn't support later modification without re-attaching, and configure context-wide tracking defaults where appropriate.

## References

### Microsoft Learn

- [Tracking vs. no-tracking queries](https://learn.microsoft.com/ef/core/querying/tracking)
