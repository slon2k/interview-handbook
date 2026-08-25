# Grouping and Aggregation

## Definition

`GROUP BY` collapses rows sharing the same value(s) in specified columns into a single summary row per group, typically combined with **aggregate functions** (`COUNT`, `SUM`, `AVG`, `MIN`, `MAX`) that compute one value across all rows in each group. `HAVING` filters groups after aggregation, distinct from `WHERE`, which filters individual rows before grouping.

```sql
SELECT CustomerId, COUNT(*) AS OrderCount, SUM(Total) AS TotalSpent
FROM Orders
GROUP BY CustomerId;
```

## Alternatives & Trade-offs

`GROUP BY` is the standard, universally-supported way to compute per-category summaries directly in the database, pushing aggregation work to the database engine instead of pulling all rows into application code to aggregate manually — usually far more efficient, especially with an appropriate index. Aggregating in application code instead can be simpler for very small result sets or when the aggregation logic is too complex for SQL, but scales poorly and duplicates work the database is specifically optimized to do.

## How It Works

### `WHERE` filters rows before grouping; `HAVING` filters groups after

```sql
SELECT CustomerId, COUNT(*) AS OrderCount
FROM Orders
WHERE OrderDate >= '2026-01-01'   -- filters individual rows first
GROUP BY CustomerId
HAVING COUNT(*) > 5;              -- filters resulting groups after aggregation
```

Using `WHERE COUNT(*) > 5` directly would fail — aggregate functions aren't available yet at the point `WHERE` is evaluated, since grouping hasn't happened.

### Every non-aggregated column in `SELECT` must appear in `GROUP BY`

```sql
-- WRONG (in strict SQL standard / most databases): CustomerName isn't grouped or aggregated
SELECT CustomerId, CustomerName, COUNT(*) FROM Orders GROUP BY CustomerId;

-- RIGHT: either group by both columns, or aggregate CustomerName too
SELECT CustomerId, CustomerName, COUNT(*) FROM Orders GROUP BY CustomerId, CustomerName;
```

Some databases (notably older MySQL configurations) permit this and pick an arbitrary value for the ungrouped column, which is a common source of subtly wrong results when a query is later run against a stricter database.

### `NULL` handling in aggregates

```sql
SELECT AVG(Discount) FROM Orders; -- NULL values are excluded from the average entirely, not treated as 0
SELECT COUNT(Discount) FROM Orders; -- counts only non-NULL Discount values
SELECT COUNT(*) FROM Orders; -- counts all rows regardless of NULLs in any column
```

### Multiple grouping columns

```sql
SELECT CustomerId, YEAR(OrderDate) AS OrderYear, SUM(Total)
FROM Orders
GROUP BY CustomerId, YEAR(OrderDate);
-- one summary row per unique (CustomerId, OrderYear) combination
```

## Application

Use `GROUP BY` with aggregates for any per-category summary (totals per customer, counts per status, averages per region) directly in SQL rather than pulling raw rows into application code. Use `HAVING` specifically to filter on the aggregated result, and `WHERE` to filter the underlying rows before aggregation for better performance (filtering early reduces how much data needs to be grouped).

## Common Mistakes

- Trying to filter on an aggregate function using `WHERE` instead of `HAVING`.
- Selecting a non-aggregated, non-grouped column alongside `GROUP BY`, relying on database-specific lenient behavior that produces an arbitrary (and potentially wrong) value.
- Forgetting that `AVG`/`SUM`/etc. silently exclude `NULL` values rather than treating them as zero, which can skew results if `NULL` actually means "zero" in context.
- Filtering rows in `HAVING` that could have been filtered earlier in `WHERE`, doing unnecessary aggregation work on rows that would have been excluded anyway.

## Common Interview Questions

### Basic
- What does `GROUP BY` do, and what are the common aggregate functions?
- What's the difference between `WHERE` and `HAVING`?

### Intermediate
- Why must every non-aggregated column in `SELECT` also appear in `GROUP BY`?
- How do aggregate functions like `AVG` and `SUM` handle `NULL` values?

### Advanced
- How would you write a query finding customers whose total spend exceeds a threshold, using `HAVING` correctly?
- Why does filtering in `WHERE` before grouping typically perform better than filtering the same condition in `HAVING` after grouping, when the filter doesn't depend on an aggregate?

### Follow-up Questions
- Can you use an aggregate function's alias directly in the same query's `HAVING` clause?
- Does `COUNT(*)` count rows with `NULL` values in any column?

### Code Prediction
Given `SELECT CustomerId, COUNT(*) FROM Orders GROUP BY CustomerId HAVING COUNT(*) > 3`, what would happen if `HAVING` were replaced with `WHERE COUNT(*) > 3`?

## Practical Tasks

- Write a query computing total and average order value per customer, filtering to only customers with more than a given number of orders.
- Reproduce the "non-aggregated column not in GROUP BY" scenario against a database that enforces it strictly, and fix the query.
- Write a query grouping by multiple columns (e.g., customer and year) and explain the resulting grouping granularity.

## Readiness Criteria

Correctly use `GROUP BY` with aggregate functions, distinguish `WHERE` from `HAVING` precisely, and reason about `NULL` handling within aggregates.

## References

### Other

- [PostgreSQL: Aggregate functions](https://www.postgresql.org/docs/current/tutorial-agg.html)
- [SQL Server: GROUP BY](https://learn.microsoft.com/sql/t-sql/queries/select-group-by-transact-sql)
