# Query Execution Plans

## Definition

An execution plan (query plan) is the database engine's chosen strategy for actually running a query — which indexes to use (or not), what order to join tables in, and what algorithm to use for each operation (index seek vs. table scan, nested loop vs. hash join). Reading it is how you move from guessing why a query is slow to actually knowing.

```sql
EXPLAIN SELECT * FROM Orders WHERE CustomerId = 42;              -- PostgreSQL
EXPLAIN ANALYZE SELECT * FROM Orders WHERE CustomerId = 42;       -- PostgreSQL, with actual runtime stats
SET SHOWPLAN_ALL ON;  -- SQL Server (or use the graphical "Include Actual Execution Plan" in SSMS)
```

## Alternatives & Trade-offs

Guessing why a query is slow based on intuition (adding an index "because it seems related") sometimes works but often misses the actual bottleneck. Reading the execution plan tells you precisely what the database is actually doing — whether it's using the index you expect, whether it's estimating row counts wildly wrong, where the time is actually being spent — turning performance tuning from guesswork into evidence-based investigation.

## How It Works

### Scan vs. seek — the most fundamental distinction

```
Table Scan / Sequential Scan — reads every row in the table, checking each against the filter
Index Scan                    — reads every row in an index (better than a table scan, still not targeted)
Index Seek                    — navigates directly to matching rows using the index structure — the fast path
```

Seeing a scan where a seek was expected is the single most common "why is this slow" finding — usually meaning either no suitable index exists, or the query is written in a way that prevents the optimizer from using one that does.

### Why a suitable index might still not be used

```sql
CREATE INDEX IX_Orders_OrderDate ON Orders(OrderDate);

-- Wrapping the indexed column in a function usually prevents the index from being used at all,
-- since the database would need to evaluate the function for every row to compare — the exact
-- work an index exists to avoid
SELECT * FROM Orders WHERE YEAR(OrderDate) = 2026; -- likely a full scan despite the index

-- Rewriting to leave the indexed column bare allows the index to be used normally
SELECT * FROM Orders WHERE OrderDate >= '2026-01-01' AND OrderDate < '2027-01-01';
```

### Join algorithms and why the choice matters

```
Nested Loop Join — for each row in the outer table, scan/seek the inner table; efficient when one side is small
Hash Join         — builds an in-memory hash table from one side, probes it with the other; efficient for large,
                     unsorted inputs without a useful index
Merge Join        — both inputs already sorted on the join key; efficient, but requires that sort order to exist
```

The optimizer picks based on estimated row counts and available indexes — a plan showing an unexpectedly expensive nested loop over a very large table is a common red flag.

### Estimated vs. actual row counts — a key diagnostic signal

```
Estimated rows: 10        Actual rows: 1,000,000
```

A large gap between estimated and actual row counts (from stale statistics, or a parameter-sensitive query optimized for an unrepresentative value) is one of the most common causes of the optimizer choosing a bad plan — it picked the right strategy for the *wrong* assumption about how much data it would touch.

## Application

Read the execution plan whenever a query is slower than expected, rather than guessing. Look specifically for scans where seeks were expected, large estimated-vs-actual row count discrepancies, and unexpectedly expensive join algorithms for the data sizes involved. Rewrite queries to avoid wrapping indexed columns in functions, which silently defeats index usage.

## Common Mistakes

- Adding an index without verifying via the execution plan that it's actually being used by the query it was meant to help.
- Wrapping an indexed column in a function or applying an implicit type conversion in a `WHERE` clause, silently preventing the optimizer from using an otherwise-suitable index.
- Trusting stale statistics without realizing the optimizer's row-count estimates (and therefore its chosen plan) can be wrong if the underlying data has changed significantly since statistics were last updated.
- Optimizing based on a single execution without checking whether the "actual" numbers in the plan match expectations, rather than just looking at the "estimated" plan shape.

## Common Interview Questions

### Basic
- What is a query execution plan?
- What's the difference between an index scan and an index seek?

### Intermediate
- Why does wrapping an indexed column in a function often prevent the optimizer from using an available index?
- What does a large gap between estimated and actual row counts in a plan typically indicate?

### Advanced
- How would you diagnose why a query using an existing, correctly-designed index is still performing a table scan?
- What's the difference between a nested loop join, a hash join, and a merge join, and when does the optimizer typically choose each?

### Follow-up Questions
- Does having an index guarantee the optimizer will use it?
- Can outdated statistics cause the optimizer to choose a worse plan than the data would otherwise justify?

### Code Prediction
Given `CREATE INDEX IX_Orders_OrderDate ON Orders(OrderDate)` and the query `WHERE YEAR(OrderDate) = 2026`, does the execution plan show an index seek or a table/index scan? What rewrite of the `WHERE` clause would change that?

## Practical Tasks

- Compare the execution plans of a query with a function-wrapped indexed column versus the equivalent rewritten range condition.
- Identify a query with a significant estimated-vs-actual row count discrepancy and investigate the likely cause (stale statistics, parameter sensitivity).
- Diagnose a slow join query by examining which join algorithm the optimizer chose and why.

## Readiness Criteria

Read an execution plan to identify scans-versus-seeks and join algorithm choices, recognize when an index isn't being used due to query phrasing, and use estimated-vs-actual row counts as a diagnostic signal.

## References

### Other

- [PostgreSQL: Using EXPLAIN](https://www.postgresql.org/docs/current/using-explain.html)
- [SQL Server: Execution plans](https://learn.microsoft.com/sql/relational-databases/performance/execution-plans)
