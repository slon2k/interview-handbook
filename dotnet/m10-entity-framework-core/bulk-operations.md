# Bulk Operations

## Definition

Bulk operations update or delete many rows directly in the database without loading each entity into the change tracker first. EF Core 7+ provides `ExecuteUpdate`/`ExecuteDelete` for this natively; third-party libraries existed to fill this gap in earlier versions and still offer additional capabilities like bulk inserts.

```csharp
await context.Orders
    .Where(o => o.Status == "Pending" && o.OrderDate < cutoffDate)
    .ExecuteUpdateAsync(setters => setters.SetProperty(o => o.Status, "Expired"));
```

## Alternatives & Trade-offs

Loading entities, mutating them in a loop, and calling `SaveChanges()` is simple and lets ordinary business logic/validation run per entity, but is very inefficient for large row counts — it means loading every row into memory, tracking each one, and issuing individual `UPDATE`/`DELETE` statements. `ExecuteUpdate`/`ExecuteDelete` issue one SQL statement affecting all matching rows directly, dramatically faster for bulk changes, at the cost of completely bypassing change tracking, entity-level validation, and any `SaveChanges`-time interceptors or business logic tied to individual entity mutation.

## How It Works

### The slow way — load, mutate, save

```csharp
var expiredOrders = await context.Orders
    .Where(o => o.Status == "Pending" && o.OrderDate < cutoffDate)
    .ToListAsync(); // loads every matching row into memory and the change tracker

foreach (var order in expiredOrders)
{
    order.Status = "Expired"; // tracked, individually
}
await context.SaveChangesAsync(); // issues one UPDATE per order
```

For 100,000 matching rows, this loads 100,000 entities into memory and issues 100,000 individual `UPDATE` statements (or batches of them, depending on provider) — expensive both in memory and round-trips.

### The fast way — `ExecuteUpdate`/`ExecuteDelete`

```csharp
var rowsUpdated = await context.Orders
    .Where(o => o.Status == "Pending" && o.OrderDate < cutoffDate)
    .ExecuteUpdateAsync(setters => setters.SetProperty(o => o.Status, "Expired"));
// generates: UPDATE Orders SET Status = 'Expired' WHERE Status = 'Pending' AND OrderDate < @cutoffDate
// one statement, no entities loaded, no change tracking involved at all

var rowsDeleted = await context.Orders
    .Where(o => o.Status == "Cancelled" && o.OrderDate < oldCutoffDate)
    .ExecuteDeleteAsync();
```

### What's lost by bypassing the change tracker

```csharp
public class Order
{
    public string Status { get; set; } = "";
    // suppose a domain method enforces a rule when Status changes, e.g., recording an audit log entry
    public void SetStatus(string newStatus) { AuditLog.Add($"Status changed to {newStatus}"); Status = newStatus; }
}
```

`ExecuteUpdateAsync` sets `Status` directly in the database — it never calls `SetStatus()`, never runs any business logic tied to that mutation, and never triggers `SaveChanges`-time behavior like `DbContext.SavingChanges` events or concurrency token checks the way normal entity updates would.

## Application

Use `ExecuteUpdate`/`ExecuteDelete` for genuinely bulk, business-logic-free operations affecting many rows at once (expiring old records, bulk status updates, cleanup jobs). Use the normal load-mutate-`SaveChanges` path whenever per-entity business logic, validation, or concurrency checks must run as part of the update.

## Common Mistakes

- Using the slow load-mutate-`SaveChanges` pattern for a genuinely bulk operation affecting a large number of rows, when `ExecuteUpdate`/`ExecuteDelete` would be dramatically faster.
- Using `ExecuteUpdate`/`ExecuteDelete` for an operation that actually needs per-entity business logic or validation to run, silently bypassing rules the application depends on.
- Forgetting that `ExecuteUpdate`/`ExecuteDelete` don't go through the change tracker, so any already-tracked in-memory changes to the same rows aren't reconciled with the bulk operation automatically.
- Not checking EF Core version support — `ExecuteUpdate`/`ExecuteDelete` require EF Core 7+; earlier versions need a third-party bulk-operations library for equivalent performance.

## Common Interview Questions

### Basic
- What do `ExecuteUpdate` and `ExecuteDelete` do, and how do they differ from the normal `SaveChanges` flow?
- Why are bulk operations faster than loading and saving entities individually?

### Intermediate
- What business-logic risk comes from using `ExecuteUpdate` instead of loading and mutating entities normally?
- When would you choose the slower load-mutate-save pattern despite its performance cost?

### Advanced
- How would you decide, for a cleanup job affecting a large number of rows, whether it's safe to bypass change tracking with `ExecuteUpdate`?
- What are the trade-offs of a third-party bulk-operations library compared to EF Core's native `ExecuteUpdate`/`ExecuteDelete`?

### Follow-up Questions
- Does `ExecuteUpdate` trigger EF Core's concurrency token checks?
- Can `ExecuteUpdate` be combined with a `Where()` filter using arbitrary LINQ conditions?

### Code Prediction
Given an `Order` entity whose `Status` setter also appends to an in-memory audit log collection, what happens to that audit log when `ExecuteUpdateAsync` is used to bulk-update `Status` for 10,000 matching rows?

## Practical Tasks

- Compare the performance and SQL generated between a load-mutate-`SaveChanges` bulk update and an equivalent `ExecuteUpdateAsync` call.
- Identify a bulk operation in a hypothetical codebase currently using the slow pattern and justify migrating it to `ExecuteUpdate`.
- Identify a scenario where per-entity business logic makes `ExecuteUpdate` the wrong choice despite the performance benefit.

## Readiness Criteria

Choose between `ExecuteUpdate`/`ExecuteDelete` and the normal entity-mutation path based on whether per-entity business logic is actually needed, and explain the performance and change-tracking trade-offs precisely.

## References

### Microsoft Learn

- [ExecuteUpdate and ExecuteDelete](https://learn.microsoft.com/ef/core/saving/execute-insert-update-delete)
