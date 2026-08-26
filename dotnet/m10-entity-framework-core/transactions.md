# Transactions in EF Core

## Definition

`SaveChanges()` wraps all pending changes in an implicit transaction automatically — every insert/update/delete in one call either all succeed or all roll back together. An **explicit transaction** is needed when multiple `SaveChanges()` calls (or a mix of EF Core operations and raw SQL) must succeed or fail together as one larger unit, building on the ACID/transaction fundamentals from Module 9.

```csharp
using var transaction = await context.Database.BeginTransactionAsync();
try
{
    context.Orders.Add(order);
    await context.SaveChangesAsync();

    context.InventoryMovements.Add(movement);
    await context.SaveChangesAsync();

    await transaction.CommitAsync();
}
catch
{
    await transaction.RollbackAsync();
    throw;
}
```

## Alternatives & Trade-offs

Relying on `SaveChanges()`'s implicit transaction is sufficient and simplest whenever all related changes can be staged and saved in one call — which is the common case, since EF Core batches everything currently tracked as changed into one atomic operation. An explicit transaction is needed specifically when the unit of work must span more than one `SaveChanges()` call, or must include both EF Core operations and raw SQL together — at the cost of more code and the responsibility to commit/rollback correctly yourself.

## How It Works

### The implicit transaction — usually all you need

```csharp
context.Orders.Add(order);
context.Customers.Update(customer);
await context.SaveChangesAsync(); // both changes committed together, or both rolled back on failure — no explicit transaction needed
```

### When an explicit transaction is actually required

```csharp
using var transaction = await context.Database.BeginTransactionAsync();
try
{
    context.Orders.Add(order);
    await context.SaveChangesAsync(); // first SaveChanges — needs the order's generated Id

    var movement = new InventoryMovement { OrderId = order.Id, Quantity = -order.Quantity };
    context.InventoryMovements.Add(movement);
    await context.SaveChangesAsync(); // second SaveChanges — depends on the first having already run

    await transaction.CommitAsync(); // only now are both changes durably committed together
}
catch
{
    await transaction.RollbackAsync(); // undoes BOTH SaveChanges calls if anything failed
    throw;
}
```

Without the explicit transaction here, a failure between the two `SaveChangesAsync()` calls would leave the `Order` committed but the `InventoryMovement` missing — a partial, inconsistent state.

### Combining EF Core operations with raw SQL in one transaction

```csharp
using var transaction = await context.Database.BeginTransactionAsync();
await context.Database.ExecuteSqlInterpolatedAsync($"UPDATE Inventory SET Stock = Stock - {quantity} WHERE ProductId = {productId}");
context.Orders.Add(order);
await context.SaveChangesAsync();
await transaction.CommitAsync();
```

### Savepoints — partial rollback within a larger transaction

```csharp
using var transaction = await context.Database.BeginTransactionAsync();
await context.SaveChangesAsync();
await transaction.CreateSavepointAsync("AfterFirstSave");
try
{
    await DoRiskyOperationAsync();
}
catch
{
    await transaction.RollbackToSavepointAsync("AfterFirstSave"); // undo only the risky part, keep the earlier work
}
await transaction.CommitAsync();
```

## Application

Rely on `SaveChanges()`'s implicit transaction for the common case of a single unit of work. Use an explicit transaction when a logical operation genuinely spans multiple `SaveChanges()` calls or mixes EF Core with raw SQL, and always wrap it in a try/catch with an explicit rollback path.

## Common Mistakes

- Performing multiple related `SaveChanges()` calls without an explicit transaction, risking a partially-completed operation if a failure occurs between them.
- Forgetting to roll back explicitly in the `catch` block of an explicit transaction, leaving a partially-applied transaction open longer than necessary.
- Wrapping every single `SaveChanges()` call in its own unnecessary explicit transaction, when the implicit transaction already provides that guarantee for a single call.
- Holding a transaction open across a slow external call (an HTTP request, a long computation) between two `SaveChanges()` calls, unnecessarily extending lock duration on the underlying rows.

## Common Interview Questions

### Basic
- Does `SaveChanges()` already run inside a transaction by default?
- When would you need an explicit transaction in EF Core?

### Intermediate
- What happens if a failure occurs between two `SaveChanges()` calls without an explicit transaction wrapping them?
- How would you combine EF Core operations and raw SQL within the same transaction?

### Advanced
- What is a savepoint, and when would you use one instead of rolling back an entire transaction?
- How would you design a multi-step operation (needing a generated Id from an earlier step) to run safely within one explicit transaction?

### Follow-up Questions
- Should a transaction be held open across a slow external API call?
- Does rolling back an explicit transaction automatically undo changes already tracked in the context's change tracker?

### Code Prediction
Given the two-`SaveChangesAsync()` example above without wrapping it in `BeginTransactionAsync()`, what state does the database end up in if the second `SaveChangesAsync()` throws an exception after the first one already succeeded?

## Practical Tasks

- Implement a multi-step operation spanning two `SaveChanges()` calls, wrapped in an explicit transaction with correct rollback handling.
- Combine a raw SQL statement and an EF Core `SaveChanges()` call within the same explicit transaction.
- Implement a savepoint-based partial rollback for a multi-step operation where only part of it is risky.

## Readiness Criteria

Explain when the implicit `SaveChanges()` transaction is sufficient versus when an explicit transaction is required, implement explicit transactions with correct commit/rollback handling, and use savepoints appropriately.

## References

### Microsoft Learn

- [Transactions in EF Core](https://learn.microsoft.com/ef/core/saving/transactions)
