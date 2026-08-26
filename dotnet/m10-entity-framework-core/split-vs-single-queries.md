# Split vs. Single Queries

## Definition

When eager-loading multiple collection navigation properties in one query, EF Core defaults to a **single query** joining everything together — but joining multiple one-to-many collections at once causes a **cartesian explosion**, where the result set's row count multiplies across each joined collection. `AsSplitQuery()` instead issues a separate SQL query per included collection, avoiding that multiplication at the cost of extra round-trips.

```csharp
// Single query (default): one SQL statement with joins
var orders = await context.Orders.Include(o => o.Items).Include(o => o.Payments).ToListAsync();

// Split query: separate SQL statements, one per Include
var orders = await context.Orders.Include(o => o.Items).Include(o => o.Payments).AsSplitQuery().ToListAsync();
```

## Alternatives & Trade-offs

A single query means one round-trip to the database, but joining multiple collections multiplies rows (an order with 3 items and 2 payments returns 6 duplicated-order rows in the raw result set, which EF Core then has to de-duplicate in memory). A split query avoids the row multiplication and the resulting extra data transfer/de-duplication cost, but pays for it with multiple round-trips — and multiple separate queries mean the data could theoretically become inconsistent if changes happen between them (a much smaller concern than it sounds, but worth knowing).

## How It Works

### The cartesian explosion problem

```csharp
var order = await context.Orders
    .Include(o => o.Items)     // 3 items
    .Include(o => o.Payments)  // 2 payments
    .FirstAsync(o => o.Id == 42);
```

A single joined query for one order with 3 items and 2 payments returns `3 × 2 = 6` raw rows (every combination of item and payment), which EF Core must then de-duplicate back into the correct object graph (1 order, 3 items, 2 payments) in memory. For larger collections, this multiplication grows very quickly and can transfer far more data than the final result actually contains.

### Fixing it with a split query

```csharp
var order = await context.Orders
    .Include(o => o.Items)
    .Include(o => o.Payments)
    .AsSplitQuery()
    .FirstAsync(o => o.Id == 42);
// Generates 3 separate SQL queries: one for the order, one for its items, one for its payments —
// no row multiplication, no de-duplication needed
```

### Setting a default behavior for a whole context

```csharp
protected override void OnConfiguring(DbContextOptionsBuilder options) =>
    options.UseSqlServer(connectionString, o => o.UseQuerySplittingBehavior(QuerySplittingBehavior.SplitQuery));
```

EF Core emits a warning by default when multiple collection includes are combined in a single query, specifically to prompt a deliberate choice between single and split query behavior rather than leaving it as an unconsidered default.

### When single query is still the better choice

```csharp
// Only one collection navigation included — no cartesian explosion risk, single query is fine as-is
var order = await context.Orders.Include(o => o.Items).FirstAsync(o => o.Id == 42);
```

Split queries aren't universally better — for a query with only one included collection, or small collections where the multiplication is negligible, the single round-trip of the default behavior is usually preferable.

## Application

Use `AsSplitQuery()` specifically when eager-loading more than one collection navigation property in the same query, especially for larger collections where the cartesian multiplication would meaningfully inflate the result set. Leave the default single-query behavior for queries including at most one collection, or where collections are small enough that multiplication isn't a real concern.

## Common Mistakes

- Ignoring EF Core's warning about multiple collection includes in a single query, unknowingly paying the cartesian-explosion cost on every execution.
- Defaulting every query to `AsSplitQuery()` regardless of whether it actually includes multiple collections, adding unnecessary round-trips where the single-query default would have been fine.
- Not realizing that split queries execute as separate statements (not wrapped in an implicit transaction by default), which matters if strict cross-query consistency is required for a specific scenario.
- Assuming the choice between single and split queries is a EF Core internal detail with no visible impact, rather than a real performance trade-off worth measuring for a specific query's actual collection sizes.

## Common Interview Questions

### Basic
- What is the cartesian explosion problem in the context of EF Core?
- What does `AsSplitQuery()` do?

### Intermediate
- Why does including multiple collection navigation properties in one query multiply the row count?
- When would you NOT want to use `AsSplitQuery()`?

### Advanced
- How would you decide, for a specific query with two included collections of known typical sizes, whether single or split query behavior performs better?
- What consistency consideration applies to split queries that doesn't apply to a single joined query?

### Follow-up Questions
- Does EF Core warn you by default when a query might suffer from cartesian explosion?
- Can split-query behavior be set as a context-wide default instead of per-query?

### Code Prediction
An order with 10 items and 5 payments is loaded with `Include(o => o.Items).Include(o => o.Payments)` using the default single-query behavior. Roughly how many raw rows does the underlying SQL query return before EF Core de-duplicates them into the final object graph?

## Practical Tasks

- Reproduce the cartesian explosion by including two collection navigation properties in a single query, and inspect the raw row count via query logging.
- Fix it using `AsSplitQuery()` and compare the number of executed SQL statements and total data transferred.
- Decide, for a query with only one included collection, whether split-query behavior would offer any benefit.

## Readiness Criteria

Explain the cartesian explosion problem precisely, decide when `AsSplitQuery()` is worth its extra round-trips, and recognize EF Core's warning as a signal to make that choice deliberately.

## References

### Microsoft Learn

- [Single vs. split queries](https://learn.microsoft.com/ef/core/querying/single-split-queries)
