# Pagination (SQL-Level)

## Definition

SQL-level pagination limits how many rows a query returns and which subset, using `OFFSET`/`FETCH` (or `LIMIT`/`OFFSET` in PostgreSQL/MySQL) for offset-based paging, or a `WHERE` condition against the last-seen row's key for keyset (cursor-based) paging. This is the database-side mechanics underlying the API-level pagination contract covered in Module 7.

```sql
SELECT * FROM Orders
ORDER BY Id
OFFSET 50 ROWS FETCH NEXT 25 ROWS ONLY; -- SQL Server: skip 50, take 25

SELECT * FROM Orders ORDER BY Id LIMIT 25 OFFSET 50; -- PostgreSQL/MySQL equivalent
```

## Alternatives & Trade-offs

Offset-based paging is simple and lets a query jump directly to any page number, but the database still has to walk through and discard every skipped row internally — for `OFFSET 100000`, that's 100,000 rows read and thrown away before the requested page is even reached, regardless of indexing. Keyset pagination avoids that entirely by seeking directly to a position using an indexed column, at the cost of only supporting "next"/"previous" navigation relative to a known key, not arbitrary page jumps.

## How It Works

### Offset pagination and its scaling problem

```sql
SELECT * FROM Orders ORDER BY Id OFFSET 100000 ROWS FETCH NEXT 25 ROWS ONLY;
```

Even with an index on `Id`, most database engines still need to traverse the first 100,000 rows in index order before it can return the next 25 — `OFFSET`'s cost grows with the offset value itself, not just the page size.

### Keyset pagination — seek directly using an indexed column

```sql
-- First page
SELECT * FROM Orders ORDER BY Id FETCH FIRST 25 ROWS ONLY;

-- Next page: seek directly to rows after the last-seen Id, using the index — no rows are skipped/discarded
SELECT * FROM Orders WHERE Id > 1042 ORDER BY Id FETCH FIRST 25 ROWS ONLY;
```

The `WHERE Id > 1042` clause lets the database use the index on `Id` to jump directly to the right starting point, regardless of how deep into the dataset that point is — the cost stays roughly constant per page, unlike `OFFSET`.

### Keyset pagination with a non-unique sort column requires a tiebreaker

```sql
-- Sorting by OrderDate alone risks skipping or duplicating rows when multiple orders share the same date
SELECT * FROM Orders
WHERE (OrderDate, Id) > ('2026-08-01', 1042) -- composite comparison: tiebreak on Id when OrderDate ties
ORDER BY OrderDate, Id
FETCH FIRST 25 ROWS ONLY;
```

Any keyset pagination scheme sorting on a non-unique column needs a unique tiebreaker column (usually the primary key) included in both the sort and the comparison, or rows can be skipped or duplicated across pages when ties exist.

### Total count — an extra cost either way

```sql
SELECT COUNT(*) FROM Orders; -- a separate, potentially expensive full-table scan, needed only if a total is displayed
```

If a UI needs "page 4 of 20," a total row count must be computed, which is itself a cost — some designs deliberately omit an exact total (showing "more available" instead) specifically to avoid this expense at scale.

## Application

Use `OFFSET`/`FETCH` for smaller datasets or UIs genuinely needing page-number jumping. Use keyset pagination for large or high-churn tables where consistent performance matters more than arbitrary page access — this is the SQL-level implementation of the keyset pagination approach discussed at the API-contract level in Module 7.

## Common Mistakes

- Using `OFFSET` on a very large table without realizing the cost scales with the offset value, not just the page size.
- Keyset-paginating on a non-unique column without a tiebreaker, causing rows to be skipped or duplicated across pages when ties occur.
- Computing an exact `COUNT(*)` on every paginated request against a very large table, adding significant cost for a number that may not need to be perfectly precise.
- Not having an index that actually supports the keyset comparison (`WHERE Id > @lastId ORDER BY Id`), turning an intended "cheap seek" into a full scan anyway.

## Common Interview Questions

### Basic
- What's the SQL syntax for offset-based pagination in your database of choice?
- Why does `OFFSET` become slower for later pages?

### Intermediate
- How does keyset pagination avoid the cost that makes deep `OFFSET` pagination slow?
- Why does keyset pagination need a unique tiebreaker column when sorting by a non-unique column?

### Advanced
- How would you design keyset pagination for a multi-column sort (e.g., sort by status, then by date)?
- How would you avoid the cost of an exact `COUNT(*)` on every paginated request against a very large, frequently-queried table?

### Follow-up Questions
- Can keyset pagination support jumping directly to page 50?
- Does an index on the sort column automatically make `OFFSET` pagination fast regardless of how large the offset is?

### Code Prediction
Given `SELECT * FROM Orders ORDER BY OrderDate FETCH FIRST 25 ROWS ONLY` used for keyset pagination without including `Id` as a tiebreaker, what could go wrong across two sequential page requests if several orders share the exact same `OrderDate`?

## Practical Tasks

- Implement both offset-based and keyset-based pagination for the same table and compare execution plans at a large offset value.
- Implement keyset pagination with a composite tiebreaker for a non-unique sort column.
- Design a pagination approach for a UI that avoids computing an exact total row count on every request.

## Readiness Criteria

Implement both offset and keyset pagination correctly at the SQL level, explain why offset pagination's cost scales with depth, and design a tiebreaker-safe keyset scheme for non-unique sort columns.

## References

### Other

- [PostgreSQL: LIMIT and OFFSET](https://www.postgresql.org/docs/current/queries-limit.html)
- [SQL Server: OFFSET-FETCH clause](https://learn.microsoft.com/sql/t-sql/queries/select-order-by-clause-transact-sql#using-offset-and-fetch-to-limit-the-rows-returned)
