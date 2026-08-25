# Transactions and ACID

## Definition

A transaction groups multiple statements into a single all-or-nothing unit of work. **ACID** describes the guarantees a properly-implemented transaction provides: **Atomicity** (all statements succeed or none do), **Consistency** (the database moves from one valid state to another, respecting all constraints), **Isolation** (concurrent transactions don't see each other's uncommitted changes — with isolation *levels* controlling exactly how much, see the next topic), and **Durability** (once committed, changes survive a crash).

```sql
BEGIN TRANSACTION;
UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;
COMMIT TRANSACTION;
-- either both balance changes happen, or neither does — there is no in-between state visible to other transactions
```

## Alternatives & Trade-offs

Wrapping related statements in a transaction guarantees the operation is atomic even if something fails partway through (a crash, a constraint violation, an explicit rollback) — critical for anything involving multiple related writes that must succeed or fail together (a funds transfer, an order placement that also decrements inventory). Not using a transaction for such an operation risks leaving the database in a partially-updated, inconsistent state if a failure occurs between the individual statements.

## How It Works

### Atomicity — all or nothing

```sql
BEGIN TRANSACTION;
UPDATE Accounts SET Balance = Balance - 100 WHERE Id = 1;
-- suppose the application crashes here, before the second statement runs
UPDATE Accounts SET Balance = Balance + 100 WHERE Id = 2;
COMMIT TRANSACTION;
```

If the crash happens before `COMMIT`, the entire transaction is rolled back automatically on recovery — the first `UPDATE` is undone too, so money is never lost or duplicated. Without a transaction, the first `UPDATE` would have already committed independently, leaving `Accounts(Id=1)` debited with no corresponding credit anywhere.

### Explicit rollback on a business-rule failure

```sql
BEGIN TRANSACTION;
UPDATE Inventory SET Stock = Stock - 5 WHERE ProductId = 3;
IF (SELECT Stock FROM Inventory WHERE ProductId = 3) < 0
BEGIN
    ROLLBACK TRANSACTION; -- undo the stock decrement entirely
    THROW 50000, 'Insufficient stock', 1;
END
COMMIT TRANSACTION;
```

### Consistency — constraints are still enforced within a transaction

```sql
BEGIN TRANSACTION;
INSERT INTO Orders (CustomerId, Total) VALUES (999, 50.00); -- CustomerId 999 doesn't exist
COMMIT TRANSACTION;
-- fails: violates the foreign key constraint, and the entire transaction rolls back automatically
```

A transaction doesn't bypass constraints — it guarantees that if any statement inside it would leave the database in an invalid (constraint-violating) state, the whole transaction fails together.

### Durability — the point of `COMMIT`

Once `COMMIT` returns successfully, the change is guaranteed to survive a subsequent crash (the database has written it to durable storage, typically via a write-ahead log) — this is the specific guarantee that distinguishes a committed transaction from data that might still be lost.

## Application

Wrap any set of related writes that must succeed or fail together in an explicit transaction — financial transfers, order placement combined with inventory adjustment, any multi-table update representing one logical business operation. Keep transactions as short as reasonably possible, since a long-running transaction holds locks (see `isolation-levels.md`) that can block other concurrent work.

## Common Mistakes

- Performing multiple related writes without a transaction, risking a partially-completed operation if a failure occurs between statements.
- Holding a transaction open for far longer than necessary (e.g., across a slow external API call), unnecessarily blocking other concurrent transactions.
- Assuming a transaction protects against every kind of inconsistency — it enforces atomicity and constraint-consistency, but application-level business logic bugs (like incorrect calculation logic) aren't caught by ACID guarantees themselves.
- Forgetting to explicitly roll back on an application-detected business-rule failure (as opposed to a database-detected constraint violation), leaving the transaction open or accidentally committed in an unintended state.

## Common Interview Questions

### Basic
- What does each letter in ACID stand for?
- What does it mean for a transaction to be atomic?

### Intermediate
- What guarantee does `COMMIT` provide that distinguishes committed data from uncommitted data?
- Why should transactions generally be kept as short as possible?

### Advanced
- How does a transaction's atomicity guarantee interact with a mid-transaction application crash — what happens to partially-executed statements?
- Give an example of a multi-table operation that must be wrapped in a transaction, and explain what could go wrong without one.

### Follow-up Questions
- Does wrapping statements in a transaction prevent all forms of application-level bugs?
- Can a transaction span multiple different databases?

### Code Prediction
Given the funds-transfer example above, if the application crashes immediately after the first `UPDATE` but before `COMMIT`, what is `Accounts(Id=1)`'s balance after the database recovers and the crashed transaction is rolled back?

## Practical Tasks

- Wrap a multi-step operation (inventory decrement + order creation) in a transaction with an explicit rollback path for a business-rule failure.
- Reproduce a partial-update inconsistency by deliberately omitting a transaction, then fix it.
- Explain, for a code review, why a specific long-running operation shouldn't hold a transaction open across an external API call.

## Readiness Criteria

Explain each ACID guarantee precisely with a concrete example, wrap multi-statement operations in transactions correctly including explicit rollback paths, and recognize when transaction scope is too broad.

## References

### Other

- [PostgreSQL: Transactions](https://www.postgresql.org/docs/current/tutorial-transactions.html)
- [SQL Server: Transactions](https://learn.microsoft.com/sql/t-sql/language-elements/transactions-transact-sql)
