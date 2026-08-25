# Subqueries and CTEs

## Definition

A **subquery** is a query nested inside another query, usable in `SELECT`, `WHERE`, `FROM`, or as part of an expression. A **CTE** (Common Table Expression, `WITH ... AS`) names a temporary result set that can be referenced elsewhere in the same statement, often making a complex query more readable than deeply nested subqueries — and, with `WITH RECURSIVE`/recursive CTEs, enabling queries over hierarchical data.

```sql
WITH RecentOrders AS (
    SELECT * FROM Orders WHERE OrderDate >= '2026-01-01'
)
SELECT CustomerId, COUNT(*) FROM RecentOrders GROUP BY CustomerId;
```

## Alternatives & Trade-offs

Subqueries and CTEs both let you compose queries from smaller, independently-understandable pieces instead of one flat statement. CTEs generally read more clearly for multi-step logic (name each step, then combine them), and are the only practical way to express recursive queries over hierarchical or graph-like data. Subqueries embedded directly in `WHERE` or `SELECT` are more concise for a single, simple filter, but nested subqueries several levels deep quickly become hard to read and reason about.

## How It Works

### Subquery in `WHERE`

```sql
SELECT Name FROM Customers
WHERE Id IN (SELECT CustomerId FROM Orders WHERE Total > 1000);
```

### Correlated subquery — references the outer query, runs conceptually once per outer row

```sql
SELECT c.Name,
       (SELECT COUNT(*) FROM Orders o WHERE o.CustomerId = c.Id) AS OrderCount
FROM Customers c;
```

A correlated subquery can be expensive if the query optimizer can't rewrite it into an efficient join-based execution plan — it's worth checking the execution plan (see `query-execution-plans.md`) rather than assuming it will always be slow or always be optimized away.

### CTE for readability

```sql
WITH CustomerTotals AS (
    SELECT CustomerId, SUM(Total) AS TotalSpent
    FROM Orders
    GROUP BY CustomerId
)
SELECT c.Name, ct.TotalSpent
FROM Customers c
JOIN CustomerTotals ct ON ct.CustomerId = c.Id
WHERE ct.TotalSpent > 1000;
```

Naming the intermediate result (`CustomerTotals`) makes the overall intent easier to follow than embedding the same aggregation as a nested subquery inline.

### Recursive CTE for hierarchical data

```sql
WITH RECURSIVE OrgChart AS (
    SELECT Id, Name, ManagerId, 1 AS Level FROM Employees WHERE ManagerId IS NULL  -- anchor: top of the hierarchy
    UNION ALL
    SELECT e.Id, e.Name, e.ManagerId, oc.Level + 1
    FROM Employees e
    JOIN OrgChart oc ON e.ManagerId = oc.Id  -- recursive: each level references the previous
)
SELECT * FROM OrgChart ORDER BY Level;
```

(SQL Server uses `WITH OrgChart AS (...)` without the `RECURSIVE` keyword but the same anchor/recursive-member structure; PostgreSQL and standard SQL require `RECURSIVE`.)

## Application

Use a CTE to break a complex query into named, readable steps, especially once nesting subqueries would go more than one or two levels deep. Use recursive CTEs specifically for hierarchical or graph-like data (org charts, category trees, bill-of-materials structures) that a fixed number of joins can't express. Use simple subqueries for straightforward, single-purpose filters where a CTE would be unnecessary ceremony.

## Common Mistakes

- Nesting subqueries several levels deep instead of extracting named CTEs, making the query very hard to read or debug.
- Assuming a correlated subquery is always slow, or always automatically optimized — actually check the execution plan for a specific query rather than assuming either way.
- Writing a recursive CTE without a terminating condition that actually converges, risking a runaway or extremely expensive query.
- Using a subquery in `SELECT` that returns more than one row when only a single scalar value is expected, causing a runtime error.

## Common Interview Questions

### Basic
- What is a CTE, and how does it differ from a subquery?
- What is a correlated subquery?

### Intermediate
- Why might a CTE be preferred over a deeply nested subquery for readability?
- What is a recursive CTE used for?

### Advanced
- How would you write a recursive CTE to compute the full reporting hierarchy under a given manager, including their level of depth?
- How would you diagnose whether a correlated subquery is being executed efficiently, and what would make you rewrite it as a join instead?

### Follow-up Questions
- Can a CTE be referenced more than once within the same statement?
- Is a CTE materialized (computed once) or potentially re-evaluated, depending on the database engine?

### Code Prediction
Given a correlated subquery computing `OrderCount` per customer for a table with a million customers, what's the risk if the subquery can't be rewritten by the optimizer into a join-based plan, compared to an equivalent `LEFT JOIN` with `GROUP BY`?

## Practical Tasks

- Rewrite a deeply nested subquery as a readable CTE.
- Write a recursive CTE to traverse a hierarchical `Employees` table and compute each employee's depth level.
- Compare the execution plan of a correlated subquery against an equivalent join-based rewrite for the same result.

## Readiness Criteria

Write and read both subqueries and CTEs fluently, choose between them based on readability and performance considerations, and write a working recursive CTE for hierarchical data.

## References

### Other

- [PostgreSQL: WITH queries (CTEs)](https://www.postgresql.org/docs/current/queries-with.html)
- [SQL Server: WITH common_table_expression](https://learn.microsoft.com/sql/t-sql/queries/with-common-table-expression-transact-sql)
