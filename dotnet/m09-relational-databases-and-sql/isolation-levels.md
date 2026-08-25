# Isolation Levels

## Definition

The isolation level of a transaction controls how much its uncommitted changes are visible to (or affected by) other concurrent transactions, trading consistency guarantees against concurrency/throughput. The standard levels, from weakest to strongest isolation, are **Read Uncommitted**, **Read Committed** (the common default), **Repeatable Read**, and **Serializable**.

```sql
SET TRANSACTION ISOLATION LEVEL READ COMMITTED;
BEGIN TRANSACTION;
SELECT Balance FROM Accounts WHERE Id = 1;
COMMIT TRANSACTION;
```

## Alternatives & Trade-offs

Weaker isolation levels allow more concurrency (less locking, less blocking between transactions) but permit more anomalies — other transactions' in-progress or since-changed data can leak through in ways that surprise developers who assume a transaction sees a perfectly stable snapshot. Stronger isolation levels prevent those anomalies but increase locking/blocking and can reduce throughput under concurrent load, or in the case of Serializable, cause more transactions to fail and need retrying due to conflicts.

## How It Works

### The three classic anomalies isolation levels address

```
Dirty read      — reading another transaction's uncommitted (possibly about-to-be-rolled-back) changes
Non-repeatable read — re-reading the same row within a transaction returns a different value, because another
                       transaction committed a change to it in between
Phantom read    — re-running the same query within a transaction returns a different SET of rows, because
                   another transaction inserted/deleted rows matching the query's condition in between
```

### What each level prevents

```
Read Uncommitted — prevents nothing; dirty reads are possible
Read Committed   — prevents dirty reads; non-repeatable reads and phantoms are still possible
Repeatable Read  — prevents dirty and non-repeatable reads; phantoms are still possible (in the standard;
                    some engines, like PostgreSQL, prevent phantoms too at this level via MVCC)
Serializable     — prevents all three; transactions behave as if run one at a time, sequentially
```

### Read Committed in practice — the common default

```sql
-- Transaction A
BEGIN TRANSACTION;
SELECT Balance FROM Accounts WHERE Id = 1; -- reads 100

-- Transaction B (concurrently) commits a change to the same row
UPDATE Accounts SET Balance = 50 WHERE Id = 1; COMMIT;

-- Transaction A, still open
SELECT Balance FROM Accounts WHERE Id = 1; -- reads 50 — a non-repeatable read, allowed at Read Committed
COMMIT;
```

### Serializable — strongest guarantee, at a real cost

```sql
SET TRANSACTION ISOLATION LEVEL SERIALIZABLE;
BEGIN TRANSACTION;
-- the database behaves as though this transaction ran completely alone;
-- a conflicting concurrent transaction may be forced to fail and retry rather than proceed
COMMIT TRANSACTION;
```

Serializable is the safest but most restrictive level — under real concurrent load it can cause more transaction failures (requiring application-level retry logic) than a weaker level would.

## Application

Default to Read Committed for most application transactions — it's the standard default in most databases and prevents the most damaging anomaly (dirty reads) while allowing good concurrency. Escalate to Repeatable Read or Serializable specifically for operations where a non-repeatable or phantom read would cause a real correctness problem (e.g., a financial calculation reading the same aggregate twice within one transaction and needing it to stay consistent) — and be prepared to handle transaction failures/retries at higher isolation levels under contention.

## Common Mistakes

- Assuming a transaction automatically sees a fully consistent, unchanging snapshot of the whole database regardless of isolation level — that guarantee only holds at Repeatable Read/Serializable, not the common Read Committed default.
- Escalating to Serializable everywhere "to be safe," without accounting for the throughput cost and the need for retry logic when transactions conflict and fail.
- Not understanding that isolation levels are about concurrent transaction interaction, not about single-transaction atomicity (which Read Uncommitted still provides).
- Assuming all databases implement each isolation level identically — PostgreSQL's MVCC-based Repeatable Read prevents phantom reads, which the SQL standard doesn't strictly require at that level, while other engines may differ.

## Common Interview Questions

### Basic
- What are the four standard isolation levels, from weakest to strongest?
- What is a dirty read?

### Intermediate
- What's the difference between a non-repeatable read and a phantom read?
- What isolation level is typically the default, and why?

### Advanced
- Why does Serializable isolation risk more transaction failures under concurrent load, and how should application code handle that?
- How would you decide which isolation level a specific business operation actually needs, rather than defaulting to the strongest available?

### Follow-up Questions
- Does Read Uncommitted ever make sense for a real application?
- Can isolation level be set per-transaction rather than as a database-wide default?

### Code Prediction
Two concurrent transactions run at Read Committed isolation. Transaction A reads a row's value twice, with Transaction B committing a change to that row in between the two reads. What value does Transaction A's second read see, and would the answer differ at Repeatable Read isolation?

## Practical Tasks

- Reproduce a non-repeatable read at Read Committed isolation with two concurrent sessions, then observe it disappear at Repeatable Read.
- Design retry logic for a Serializable transaction that may fail due to a concurrent conflict.
- Justify, for a specific business operation, whether Read Committed is sufficient or a stronger isolation level is genuinely needed.

## Readiness Criteria

Explain all three classic anomalies and which isolation levels prevent each, justify an isolation-level choice for a given scenario rather than defaulting to the strongest available, and design retry handling for isolation-related transaction failures.

## References

### Other

- [PostgreSQL: Transaction isolation](https://www.postgresql.org/docs/current/transaction-iso.html)
- [SQL Server: Transaction isolation levels](https://learn.microsoft.com/sql/t-sql/statements/set-transaction-isolation-level-transact-sql)
