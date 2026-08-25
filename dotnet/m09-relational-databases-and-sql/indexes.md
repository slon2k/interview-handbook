# Indexes

## Definition

An index is a separate data structure (typically a B-tree) that lets the database find rows matching a condition without scanning every row in the table, at the cost of extra storage and slower writes (since the index must also be updated on every insert/update/delete affecting indexed columns).

```sql
CREATE INDEX IX_Orders_CustomerId ON Orders(CustomerId);
```

## Alternatives & Trade-offs

No index means every query filtering on that column requires a full table scan — fine for a small table, increasingly expensive as row count grows. An index makes lookups on that column fast (typically logarithmic rather than linear in table size), but every index adds overhead to every write against the table (the index must be kept up to date) and consumes additional storage — indexing every column "just in case" is a real cost, not a free optimization.

## How It Works

### What an index actually does

```sql
-- Without an index on CustomerId: the database scans every row in Orders to find matches
SELECT * FROM Orders WHERE CustomerId = 42;

-- With an index on CustomerId: the database can navigate directly to matching rows
CREATE INDEX IX_Orders_CustomerId ON Orders(CustomerId);
SELECT * FROM Orders WHERE CustomerId = 42; -- now uses an index seek instead of a table scan
```

### Composite indexes and column order matters

```sql
CREATE INDEX IX_Orders_CustomerId_OrderDate ON Orders(CustomerId, OrderDate);

-- Can use the index efficiently — CustomerId is the leading column
SELECT * FROM Orders WHERE CustomerId = 42 AND OrderDate > '2026-01-01';
SELECT * FROM Orders WHERE CustomerId = 42;

-- Cannot use this index efficiently — OrderDate alone isn't the leading column
SELECT * FROM Orders WHERE OrderDate > '2026-01-01';
```

A composite index is only useful for queries filtering on a *prefix* of its columns, left to right — like a phone book sorted by last name then first name: useful for finding "all Smiths," useless for finding "all Johns" regardless of last name.

### Covering indexes — avoiding a lookup back to the table entirely

```sql
CREATE INDEX IX_Orders_Covering ON Orders(CustomerId) INCLUDE (Total, OrderDate);

SELECT Total, OrderDate FROM Orders WHERE CustomerId = 42;
-- the index itself contains everything needed to answer the query — no need to fetch the full row
```

### The write-cost trade-off

```sql
-- Five indexes on Orders means every INSERT/UPDATE/DELETE touching indexed columns
-- must also update all five index structures, not just the underlying table
```

Over-indexing a write-heavy table can noticeably slow down writes for marginal or unused read benefit — indexes should be added deliberately, based on actual query patterns, not preemptively on every column.

## Application

Index columns frequently used in `WHERE`, `JOIN`, and `ORDER BY` clauses — especially foreign key columns, which aren't automatically indexed by most databases despite being joined on constantly. Design composite indexes with the most selective or most commonly-filtered column first, matching actual query patterns. Use covering indexes for hot, read-heavy queries where avoiding the extra lookup back to the table row matters. Avoid indexing columns that are rarely queried or that change constantly, where the write overhead isn't justified.

## Common Mistakes

- Not indexing foreign key columns, since most databases don't do this automatically, leaving joins on those columns slow.
- Creating a composite index in the wrong column order for the queries actually being run, making the index useless for the most common filter pattern.
- Indexing every column "just in case," accumulating write overhead across many rarely-used indexes.
- Assuming an index always helps — for very small tables, or queries returning a large fraction of the table's rows, the database may correctly choose a full table scan over using an index anyway, since a scan can sometimes be genuinely faster in that specific case (see `query-execution-plans.md`).

## Common Interview Questions

### Basic
- What is an index, and what problem does it solve?
- What's the cost of adding an index?

### Intermediate
- Why does column order matter in a composite index?
- Why should foreign key columns typically be indexed even though most databases don't do it automatically?

### Advanced
- What is a covering index, and how does it avoid an extra lookup back to the table?
- How would you decide which columns of a table actually need an index, based on real query patterns rather than guessing?

### Follow-up Questions
- Can an index ever make a query slower?
- Does adding an index require rewriting existing queries to benefit from it?

### Code Prediction
Given `CREATE INDEX IX_Orders_CustomerId_OrderDate ON Orders(CustomerId, OrderDate)`, does a query filtering only on `WHERE OrderDate > '2026-01-01'` (with no `CustomerId` filter) use this index efficiently? Why or why not?

## Practical Tasks

- Add an index to a foreign key column and compare query performance before and after using `EXPLAIN`/execution plan output.
- Design a composite index for a set of common query patterns, choosing the correct column order.
- Design a covering index for a specific hot query and verify it avoids a lookup back to the base table.

## Readiness Criteria

Explain how indexes work and their write-cost trade-off, design composite indexes with correct column ordering for real query patterns, and recognize when an index won't actually help a given query.

## References

### Other

- [PostgreSQL: Indexes](https://www.postgresql.org/docs/current/indexes.html)
- [SQL Server: Indexes](https://learn.microsoft.com/sql/relational-databases/indexes/indexes)
