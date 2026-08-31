# SQL Query Exercises

## What This Assesses

Can you write correct, reasonably efficient SQL from a plain-language requirement, using joins, aggregation, and window functions fluently, and avoid the classic traps (Module 9's `NULL`/outer-join gotchas) that separate junior from mid-level SQL knowledge.

## Format and Time Expectations

A schema description (2-4 tables) plus a plain-language question, solved on a whiteboard or shared doc — usually no real database to test against, so care and precision matter more than trial-and-error.

## Exercise 1: Customers With No Orders

**Problem:** Given `Customers(Id, Name)` and `Orders(Id, CustomerId, Total)`, find every customer who has never placed an order.

```sql
SELECT c.Name
FROM Customers c
LEFT JOIN Orders o ON o.CustomerId = c.Id
WHERE o.Id IS NULL;
```

**What a strong answer demonstrates:** Using a `LEFT JOIN` plus `IS NULL` (Module 9's join content) rather than a `NOT IN` subquery — and if using `NOT IN`, explicitly guarding against the `NULL`-in-subquery trap (Module 9) that silently returns zero rows if `Orders.CustomerId` ever contains a `NULL`.

**Common mistakes:** Writing `WHERE CustomerId NOT IN (SELECT CustomerId FROM Orders)` without considering whether `Orders.CustomerId` could contain `NULL`, silently breaking the whole query.

## Exercise 2: Top 3 Highest-Value Orders Per Customer

**Problem:** Given the same schema, return each customer's 3 highest-value orders.

```sql
WITH RankedOrders AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY CustomerId ORDER BY Total DESC) AS RowNum
    FROM Orders
)
SELECT * FROM RankedOrders WHERE RowNum <= 3;
```

**What a strong answer demonstrates:** Reaching directly for a window function (Module 9) rather than a correlated subquery or self-join, and choosing `ROW_NUMBER()` deliberately (versus `RANK()`) with an explanation of how ties would be handled differently by each.

**Common mistakes:** Attempting `ORDER BY Total DESC LIMIT 3` globally, which returns the 3 highest orders *overall*, not 3 per customer — missing the "per group" requirement entirely.

## Exercise 3: Monthly Revenue with Month-Over-Month Change

**Problem:** Given `Orders(Id, Total, OrderDate)`, compute total revenue per month, along with the percentage change from the previous month.

```sql
WITH MonthlyRevenue AS (
    SELECT DATE_TRUNC('month', OrderDate) AS Month, SUM(Total) AS Revenue
    FROM Orders
    GROUP BY DATE_TRUNC('month', OrderDate)
)
SELECT Month, Revenue,
       Revenue - LAG(Revenue) OVER (ORDER BY Month) AS ChangeFromPreviousMonth,
       (Revenue - LAG(Revenue) OVER (ORDER BY Month)) / LAG(Revenue) OVER (ORDER BY Month) * 100 AS PercentChange
FROM MonthlyRevenue;
```

**What a strong answer demonstrates:** Combining `GROUP BY` (for the monthly aggregation) with `LAG()` (Module 9's window-function content) for the period-over-period comparison — recognizing these solve two different, composable problems rather than trying to force one operator to do both.

**Common mistakes:** Attempting the month-over-month comparison with a self-join on "this month = previous month's month + 1," which is fragile compared to `LAG()` and breaks for months with zero orders (a gap in the sequence).

## Exercise 4: Diagnosing a Slow Query

**Problem:** Given `SELECT * FROM Orders WHERE YEAR(OrderDate) = 2026` runs slowly despite an index on `OrderDate`, explain why and rewrite it.

```sql
-- Rewritten to leave the indexed column bare, allowing the index to actually be used
SELECT * FROM Orders WHERE OrderDate >= '2026-01-01' AND OrderDate < '2027-01-01';
```

**What a strong answer demonstrates:** Recognizing that wrapping the indexed column in a function (`YEAR(OrderDate)`) prevents the optimizer from using the index (Module 9's execution-plans content), since it would need to evaluate the function for every row to compare — and rewriting as an equivalent range condition that leaves the column bare.

**Common mistakes:** Suggesting "add another index" without recognizing the actual problem is the function wrapping, which would prevent *any* index on that column from being used the same way.

## Readiness Criteria

Write joins, window functions, and aggregations fluently and correctly on the first attempt, proactively avoid the `NULL`/outer-join traps from Module 9, and diagnose why a query isn't using an available index rather than only suggesting more indexes.

## References

- [Inner and outer joins (Module 9)](../m09-relational-databases-and-sql/joins.md)
- [Window functions (Module 9)](../m09-relational-databases-and-sql/window-functions.md)
- [NULL handling and three-valued logic (Module 9)](../m09-relational-databases-and-sql/null-handling.md)
- [Query execution plans (Module 9)](../m09-relational-databases-and-sql/query-execution-plans.md)
