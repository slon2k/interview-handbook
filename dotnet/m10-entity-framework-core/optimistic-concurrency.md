# Optimistic Concurrency

## Definition

Optimistic concurrency detects when two operations have modified the same row based on stale data, without locking the row for the duration of the read (unlike pessimistic locking, which holds a lock from read through write). EF Core implements this via a **concurrency token** — typically a `rowversion`/`timestamp` column — checked automatically as part of the `UPDATE`'s `WHERE` clause on `SaveChanges()`.

```csharp
public class Order
{
    public int Id { get; set; }
    public decimal Total { get; set; }
    [Timestamp]
    public byte[] RowVersion { get; set; } = null!;
}
```

## Alternatives & Trade-offs

Optimistic concurrency assumes conflicts are rare and only checks for them at write time, avoiding the throughput cost of holding locks for the entire read-then-write window — a good fit for most web applications, where a user reads data, thinks, then submits an update much later. Pessimistic locking (`SELECT ... FOR UPDATE` or explicit database locks) prevents conflicts from happening at all by blocking other transactions from touching the row until the lock is released, at the cost of reduced concurrency and the risk of a forgotten or long-held lock blocking other work.

## How It Works

### The concurrency token in the generated SQL

```sql
UPDATE Orders SET Total = @newTotal
WHERE Id = @id AND RowVersion = @originalRowVersion; -- fails to match if another transaction already updated this row
```

EF Core includes the originally-read `RowVersion` value in the `WHERE` clause of the generated `UPDATE`. If another transaction already changed the row (and therefore its `RowVersion`), this `UPDATE` affects zero rows — a mismatch EF Core detects and turns into an exception.

### Handling a concurrency conflict

```csharp
try
{
    await context.SaveChangesAsync();
}
catch (DbUpdateConcurrencyException ex)
{
    var entry = ex.Entries.Single();
    var currentValues = entry.CurrentValues;      // what this operation tried to save
    var databaseValues = await entry.GetDatabaseValuesAsync(); // what's actually in the database now

    // A real resolution strategy: merge, prompt the user to choose, or simply retry with fresh data —
    // never blindly overwrite without at least inspecting what changed
    entry.OriginalValues.SetValues(databaseValues!);
}
```

`DbUpdateConcurrencyException` is EF Core's signal that the row changed since it was read — the application must decide how to resolve it, since EF Core itself has no way to know which version should "win."

### Concurrency tokens on any column, not just a dedicated `RowVersion`

```csharp
builder.Property(o => o.Status).IsConcurrencyToken(); // any column can act as a concurrency check, not just rowversion
```

Using a business column as the token means a conflict is only detected if that *specific* column changed, rather than any change to the row at all — a more targeted (and more manual) alternative to a dedicated `rowversion` column that changes on every update.

## Application

Add a `[Timestamp]`/`rowversion` concurrency token to entities where conflicting concurrent updates are plausible and must be detected (orders, inventory, any shared mutable business record). Handle `DbUpdateConcurrencyException` deliberately — reload and retry, merge, or surface a clear conflict to the user — rather than either ignoring it or blindly overwriting.

## Common Mistakes

- Not adding a concurrency token at all, silently allowing a "last write wins" outcome where a user's changes can be overwritten without any indication a conflict occurred.
- Catching `DbUpdateConcurrencyException` and blindly retrying with the same stale data, without actually resolving what conflicted.
- Confusing optimistic concurrency (checked at write time) with pessimistic locking (held from read through write) when discussing the trade-offs in an interview.
- Assuming a concurrency token alone tells you *what* changed — `ex.Entries` and `GetDatabaseValuesAsync()` are needed to actually inspect the conflicting values and decide how to resolve them.

## Common Interview Questions

### Basic
- What is optimistic concurrency, and how does EF Core implement it?
- What is a concurrency token?

### Intermediate
- What happens when two operations attempt to update the same row, and one has a stale concurrency token?
- What's the difference between optimistic and pessimistic concurrency control?

### Advanced
- How would you implement a merge-based conflict resolution strategy using `DbUpdateConcurrencyException`?
- When would pessimistic locking be a better fit than optimistic concurrency for a specific operation?

### Follow-up Questions
- Does every column need to be a concurrency token, or can a subset of columns be checked instead?
- Is `DbUpdateConcurrencyException` thrown before or after the database actually attempts the update?

### Code Prediction
Two users load the same `Order` with `RowVersion = A`. User 1 saves a change, and the database updates `RowVersion` to `B`. User 2, still holding the stale `RowVersion = A`, then calls `SaveChangesAsync()`. What happens?

## Practical Tasks

- Add a `[Timestamp]` concurrency token to an entity and reproduce a `DbUpdateConcurrencyException` with two concurrent updates.
- Implement a conflict-resolution strategy that reloads current database values and lets the caller choose how to proceed.
- Compare optimistic concurrency against a pessimistic locking approach for the same update scenario.

## Readiness Criteria

Implement optimistic concurrency with a token, handle `DbUpdateConcurrencyException` with a real resolution strategy rather than blind retry or silent overwrite, and articulate the trade-off against pessimistic locking.

## References

### Microsoft Learn

- [Handling concurrency conflicts](https://learn.microsoft.com/ef/core/saving/concurrency)
