# Change Tracking

## Definition

EF Core's change tracker records the original state of every entity loaded through a tracking query, and compares it against the entity's current state when `SaveChanges()` is called, generating only the `UPDATE`/`INSERT`/`DELETE` statements needed to reflect what actually changed — not a full rewrite of every loaded entity.

```csharp
var order = await context.Orders.FirstAsync(o => o.Id == 42); // tracked; original state recorded
order.Status = "Shipped";                                      // only this property is now "modified"
await context.SaveChangesAsync();                               // generates: UPDATE Orders SET Status = 'Shipped' WHERE Id = 42
```

## Alternatives & Trade-offs

Change tracking is what makes `SaveChanges()` "just work" without manually building update statements — you mutate objects naturally, and EF Core figures out the minimal SQL. The cost is memory and CPU overhead for tracking every loaded entity's original state, which matters for read-heavy, no-modification-intended queries — this is exactly the case `AsNoTracking()` (covered separately) exists to optimize away.

## How It Works

### Entity states

```
Detached  — not tracked by any context
Unchanged — tracked, no modifications since it was loaded (or since the last SaveChanges)
Added     — new entity, will be INSERTed on SaveChanges
Modified  — tracked entity with at least one changed property, will be UPDATEd
Deleted   — marked for removal, will be DELETEd on SaveChanges
```

```csharp
var order = await context.Orders.FirstAsync(o => o.Id == 42);
Console.WriteLine(context.Entry(order).State); // Unchanged
order.Status = "Shipped";
Console.WriteLine(context.Entry(order).State); // Modified
```

### Only actually-changed properties are included in the generated SQL

```csharp
var order = await context.Orders.FirstAsync(o => o.Id == 42); // Total = 100, Status = "Pending"
order.Status = "Shipped"; // Total untouched
await context.SaveChangesAsync();
// UPDATE Orders SET Status = 'Shipped' WHERE Id = 42 — Total is not included, since it never changed
```

### Attaching a detached entity — telling EF Core about an entity it never loaded

```csharp
var order = new Order { Id = 42, Status = "Shipped" }; // e.g., deserialized from an API request, never queried
context.Attach(order);
context.Entry(order).Property(o => o.Status).IsModified = true; // explicitly mark only this property changed
await context.SaveChangesAsync(); // UPDATE Orders SET Status = 'Shipped' WHERE Id = 42
```

Without explicitly marking which properties changed, attaching a whole entity built from external data (like a deserialized API payload) risks overwriting every column with whatever the payload happened to contain — including columns the payload never intended to touch.

### The change tracker grows with every tracked entity

```csharp
var allOrders = await context.Orders.ToListAsync(); // every one of these is now tracked, using memory
```

Loading a large number of entities into a tracking query keeps all of their original-state snapshots in memory for the life of the context — a real cost at scale, addressed by `AsNoTracking()` for read-only scenarios.

## Application

Rely on change tracking for the common case: load an entity, mutate it, call `SaveChanges()`. Use `Attach` with explicit `IsModified` flags when updating an entity from external data without first querying it. Be aware that tracking many entities in one context has real memory cost, which is the reason `AsNoTracking()` exists as a deliberate opt-out for read-only queries.

## Common Mistakes

- Attaching a fully-populated entity from external input without marking specific properties as modified, accidentally overwriting columns the caller never intended to change.
- Loading a very large result set with tracking enabled for a purely read-only operation, incurring unnecessary memory overhead.
- Modifying an entity's properties and being surprised `SaveChanges()` doesn't persist the change — usually because the entity was loaded with `AsNoTracking()` or is genuinely `Detached`.
- Assuming `SaveChanges()` always issues one SQL statement per entity — a single call can generate multiple statements (inserts, updates, deletes) batched together based on everything currently tracked as changed.

## Common Interview Questions

### Basic
- What is EF Core's change tracker, and what does `SaveChanges()` do with it?
- What are the possible entity states (`Added`, `Modified`, etc.)?

### Intermediate
- Why does `SaveChanges()` only update the specific columns that actually changed, rather than the whole row?
- What's the risk of attaching an entity built from external input without marking specific properties as modified?

### Advanced
- How would you update a single property on an entity you haven't queried, without loading the full entity first?
- What's the memory cost implication of loading a large result set with tracking enabled, and how does that motivate `AsNoTracking()`?

### Follow-up Questions
- Does modifying a `Detached` entity's properties get picked up by `SaveChanges()`?
- Can `SaveChanges()` generate more than one SQL statement in a single call?

### Code Prediction
```csharp
var order = new Order { Id = 42, Status = "Shipped", Total = 0 }; // Total is default, never actually set by the caller
context.Attach(order);
await context.SaveChangesAsync();
```
If `Attach` is used without explicitly marking only `Status` as modified, what happens to the existing `Total` value in the database for order 42?

## Practical Tasks

- Load an entity, modify one property, and inspect the generated SQL to confirm only that column is updated.
- Update an entity from a deserialized external payload using `Attach` with explicit `IsModified` flags on only the intended properties.
- Compare the change tracker's memory behavior between a tracking query and an `AsNoTracking()` query for a large result set.

## Readiness Criteria

Explain entity states and how `SaveChanges()` uses them to generate minimal SQL, safely update entities from external data using `Attach`, and recognize the memory cost of tracking large result sets.

## References

### Microsoft Learn

- [Change tracking in EF Core](https://learn.microsoft.com/ef/core/change-tracking/)
- [Tracking vs. no-tracking queries](https://learn.microsoft.com/ef/core/querying/tracking)
