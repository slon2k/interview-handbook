# Global Query Filters

## Definition

A global query filter is a `WHERE` condition EF Core automatically applies to every query against a given entity type, without needing to repeat it at every call site. The most common use case is **soft delete** (excluding rows flagged as deleted) and **multi-tenancy** (automatically scoping queries to the current tenant).

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Order>().HasQueryFilter(o => !o.IsDeleted);
}

var orders = await context.Orders.ToListAsync(); // automatically excludes IsDeleted == true rows
```

## Alternatives & Trade-offs

A global query filter guarantees a rule is applied everywhere automatically, eliminating the risk of a developer forgetting `WHERE IsDeleted = 0` in one of many query call sites. The trade-off is that the filter is easy to forget exists at all when reading code — a query that looks like it fetches "all orders" is quietly scoped, which can confuse debugging or reporting scenarios that genuinely need the excluded rows, unless the filter is deliberately bypassed.

## How It Works

### Soft delete via a global query filter

```csharp
public class Order
{
    public int Id { get; set; }
    public bool IsDeleted { get; set; }
}

modelBuilder.Entity<Order>().HasQueryFilter(o => !o.IsDeleted);

context.Orders.Remove(order); // if this maps to an actual DELETE, soft-delete isn't implemented yet —
                                // soft delete instead means setting order.IsDeleted = true and calling SaveChanges
```

Every ordinary query against `Orders` now silently excludes deleted rows — `context.Orders.ToListAsync()`, `FirstAsync()`, `Count()`, all of it — without needing `Where(o => !o.IsDeleted)` repeated everywhere.

### Multi-tenancy via a global query filter

```csharp
public class AppDbContext : DbContext
{
    private readonly string _currentTenantId;
    public AppDbContext(string currentTenantId) => _currentTenantId = currentTenantId;

    protected override void OnModelCreating(ModelBuilder modelBuilder)
    {
        modelBuilder.Entity<Order>().HasQueryFilter(o => o.TenantId == _currentTenantId);
    }
}
```

Every query automatically scopes to the current tenant, closing off an entire category of "forgot to filter by tenant" data-leak bugs across every call site at once.

### Deliberately bypassing a filter when you genuinely need to

```csharp
var allOrdersIncludingDeleted = await context.Orders.IgnoreQueryFilters().ToListAsync();
```

`IgnoreQueryFilters()` is the explicit escape hatch — used sparingly, for administrative/reporting scenarios that genuinely need to see filtered-out rows, never as a routine workaround.

### Global filters and relationships

```csharp
// If Order has a query filter and is loaded via Include() from Customer,
// the filter still applies to the included Orders collection automatically
var customers = await context.Customers.Include(c => c.Orders).ToListAsync();
```

## Application

Use global query filters for cross-cutting, always-applicable rules that must never be forgotten at any call site — soft delete and multi-tenancy are the two textbook cases. Use `IgnoreQueryFilters()` explicitly and sparingly for the specific administrative/reporting scenarios that need the excluded rows, documenting why at each use.

## Common Mistakes

- Forgetting a global query filter exists on an entity, leading to confusion when a query "should" return certain rows but silently doesn't.
- Using `IgnoreQueryFilters()` routinely instead of only for genuinely justified administrative cases, defeating the safety guarantee the filter exists to provide.
- Implementing "soft delete" as just a global query filter without also considering unique constraints (a soft-deleted row with a unique `Email` might block a new row from reusing that email, since it still physically exists).
- Not testing that a global filter also applies correctly through relationships and included navigation properties, not just direct `DbSet<T>` queries.

## Common Interview Questions

### Basic
- What is a global query filter, and what's the classic use case?
- How would you implement soft delete using a global query filter?

### Intermediate
- How do you deliberately bypass a global query filter when you genuinely need the excluded rows?
- What's the risk of forgetting that a global query filter exists on an entity?

### Advanced
- How would you implement multi-tenancy using a global query filter tied to the current request's tenant context?
- What complication does soft delete introduce for unique constraints, and how would you address it?

### Follow-up Questions
- Does a global query filter apply automatically to entities loaded via `Include()`?
- Can a global query filter reference external context, like the currently authenticated user?

### Code Prediction
Given `HasQueryFilter(o => !o.IsDeleted)` on `Order`, what does `await context.Orders.CountAsync()` return if there are 100 total order rows, 20 of which have `IsDeleted = true`?

## Practical Tasks

- Implement soft delete for an entity using a global query filter, including deliberate bypass via `IgnoreQueryFilters()` for an admin report.
- Implement a multi-tenancy global query filter scoped to a value provided at `DbContext` construction time.
- Verify that a global query filter on a child entity is respected when the parent is loaded via `Include()`.

## Readiness Criteria

Implement global query filters for soft delete and multi-tenancy correctly, use `IgnoreQueryFilters()` deliberately and sparingly, and recognize the interaction between soft delete and unique constraints.

## References

### Microsoft Learn

- [Global query filters](https://learn.microsoft.com/ef/core/querying/filters)
