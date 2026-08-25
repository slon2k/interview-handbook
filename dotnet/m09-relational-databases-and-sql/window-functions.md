# Window Functions

## Definition

A window function computes a value across a set of related rows (a "window") without collapsing them into a single output row the way `GROUP BY` does — each input row still appears in the output, now with an added computed column. Common window functions include `ROW_NUMBER()`, `RANK()`, `DENSE_RANK()`, `LAG()`/`LEAD()`, and aggregate functions used with `OVER()` instead of `GROUP BY`.

```sql
SELECT CustomerId, OrderDate, Total,
       ROW_NUMBER() OVER (PARTITION BY CustomerId ORDER BY OrderDate DESC) AS RowNum
FROM Orders;
```

## Alternatives & Trade-offs

`GROUP BY` answers "what's the total/count/average per group," collapsing rows. Window functions answer "what's this row's rank/running total/comparison to neighboring rows *within* its group," while keeping every row intact — a fundamentally different question that `GROUP BY` alone cannot express without a self-join or subquery, both of which are typically far less readable and less efficient than the equivalent window function.

## How It Works

### `PARTITION BY` — dividing rows into groups the window function computes over independently

```sql
SELECT CustomerId, OrderDate, Total,
       SUM(Total) OVER (PARTITION BY CustomerId ORDER BY OrderDate) AS RunningTotal
FROM Orders;
-- each customer gets their own independent running total, restarting at each new CustomerId
```

### `ROW_NUMBER()` vs. `RANK()` vs. `DENSE_RANK()` — handling ties differently

```sql
SELECT Name, Score,
       ROW_NUMBER() OVER (ORDER BY Score DESC) AS RowNum,   -- 1,2,3,4 — always unique, arbitrary tiebreak
       RANK()       OVER (ORDER BY Score DESC) AS Rank,     -- 1,2,2,4 — ties share rank, next rank skips
       DENSE_RANK() OVER (ORDER BY Score DESC) AS DenseRank -- 1,2,2,3 — ties share rank, no skip
FROM Scores;
```

Choosing the wrong one for a "top N" or leaderboard query produces subtly different, easily-missed-in-review results when ties exist.

### A very common practical pattern: "top N per group"

```sql
WITH RankedOrders AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY CustomerId ORDER BY Total DESC) AS RowNum
    FROM Orders
)
SELECT * FROM RankedOrders WHERE RowNum <= 3; -- each customer's 3 largest orders
```

This is the standard way to express "top N per group," which is awkward or inefficient to express with plain `GROUP BY` alone.

### `LAG`/`LEAD` — comparing a row to a neighboring row

```sql
SELECT OrderDate, Total,
       LAG(Total) OVER (ORDER BY OrderDate) AS PreviousOrderTotal,
       Total - LAG(Total) OVER (ORDER BY OrderDate) AS ChangeFromPrevious
FROM Orders;
```

## Application

Reach for a window function whenever a query needs to rank, compare to a neighboring row, or compute a running/moving aggregate while still returning one row per input row — running totals, "top N per group," period-over-period comparisons, deduplication (keep only the first row per group via `ROW_NUMBER() = 1`).

## Common Mistakes

- Using `RANK()` when `ROW_NUMBER()` was actually intended (or vice versa), producing wrong results specifically when ties exist — easy to miss until real duplicate values show up in production data.
- Forgetting `PARTITION BY`, causing the window function to compute across the entire result set instead of independently per group.
- Trying to filter directly on a window function's result in the same `SELECT`'s `WHERE` clause — window functions are computed after `WHERE`, so the result must be wrapped in a subquery or CTE and filtered there instead.
- Reaching for a self-join or correlated subquery to solve a "top N per group" or running-total problem that a window function would express far more simply and efficiently.

## Common Interview Questions

### Basic
- What is a window function, and how does it differ from `GROUP BY`?
- What does `PARTITION BY` do inside an `OVER()` clause?

### Intermediate
- What's the difference between `ROW_NUMBER()`, `RANK()`, and `DENSE_RANK()` when ties exist?
- How would you find the top 3 highest-value orders per customer using a window function?

### Advanced
- Why can't a window function's result be filtered directly in the same query's `WHERE` clause, and how do you work around it?
- How would you compute a period-over-period percentage change using `LAG()`?

### Follow-up Questions
- Can multiple window functions with different `PARTITION BY`/`ORDER BY` clauses be used in the same query?
- Are window functions part of the SQL standard, or vendor-specific extensions?

### Code Prediction
Given three orders tied at the highest `Total` for a customer, what values does `ROW_NUMBER()` assign to them versus `RANK()`? If the query is `WHERE RowNum <= 1` using each function, how many rows does each version return for that customer?

## Practical Tasks

- Write a query returning each customer's 3 largest orders using `ROW_NUMBER()` and a CTE.
- Compute a running total of orders per customer over time using `SUM() OVER (PARTITION BY ... ORDER BY ...)`.
- Compute month-over-month change in total sales using `LAG()`.

## Readiness Criteria

Choose the correct window function (`ROW_NUMBER` vs. `RANK` vs. `DENSE_RANK`) for a given tie-handling requirement, express "top N per group" and running-total patterns fluently, and correctly filter on a window function's result via a CTE or subquery.

## References

### Other

- [PostgreSQL: Window functions](https://www.postgresql.org/docs/current/tutorial-window.html)
- [SQL Server: OVER clause](https://learn.microsoft.com/sql/t-sql/queries/select-over-clause-transact-sql)
