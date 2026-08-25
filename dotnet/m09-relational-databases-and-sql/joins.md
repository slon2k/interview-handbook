# Inner and Outer Joins

## Definition

A `JOIN` combines rows from two or more tables based on a related column. An **inner join** returns only rows with matches in both tables. An **outer join** (`LEFT`, `RIGHT`, or `FULL`) returns matched rows plus unmatched rows from one or both sides, filling in `NULL` for columns that have no match.

```sql
SELECT c.Name, o.Total
FROM Customers c
INNER JOIN Orders o ON o.CustomerId = c.Id;
-- only customers who have at least one order appear
```

## Alternatives & Trade-offs

Inner joins are the default choice when you only care about rows that genuinely correspond on both sides. Outer joins are necessary whenever "no match" is itself meaningful information — customers with zero orders, products never sold — but require careful handling of the resulting `NULL`s in any column from the unmatched side, since forgetting that can silently produce wrong aggregates or comparisons.

## How It Works

### Inner join — only matching rows

```sql
SELECT c.Name, o.Total
FROM Customers c
INNER JOIN Orders o ON o.CustomerId = c.Id;
-- Customers with zero orders are excluded entirely
```

### Left (outer) join — all rows from the left table, matched or not

```sql
SELECT c.Name, o.Total
FROM Customers c
LEFT JOIN Orders o ON o.CustomerId = c.Id;
-- Every customer appears at least once; customers with no orders show Total = NULL
```

### The classic outer-join trap: filtering in `WHERE` silently turns it into an inner join

```sql
-- WRONG: this filter drops every row where o.Total IS NULL,
-- which includes exactly the "customers with no orders" rows the LEFT JOIN was meant to keep
SELECT c.Name, o.Total
FROM Customers c
LEFT JOIN Orders o ON o.CustomerId = c.Id
WHERE o.Total > 100;

-- RIGHT: move the condition into the join itself to preserve unmatched left rows
SELECT c.Name, o.Total
FROM Customers c
LEFT JOIN Orders o ON o.CustomerId = c.Id AND o.Total > 100;
```

A condition on the right-hand table placed in `WHERE` implicitly requires a non-null match, silently converting a `LEFT JOIN` back into behavior equivalent to an `INNER JOIN` — a very common and subtle bug.

### Full outer join — all rows from both sides

```sql
SELECT c.Name, o.Total
FROM Customers c
FULL OUTER JOIN Orders o ON o.CustomerId = c.Id;
-- includes customers with no orders AND orders with no matching customer (data-integrity issue if it happens)
```

### Counting correctly across a join

```sql
-- WRONG: COUNT(*) counts joined rows, which is wrong if a customer has multiple orders
SELECT c.Id, COUNT(*) FROM Customers c LEFT JOIN Orders o ON o.CustomerId = c.Id GROUP BY c.Id;

-- Often intended: count of matched orders, correctly excluding the NULL placeholder row
SELECT c.Id, COUNT(o.Id) FROM Customers c LEFT JOIN Orders o ON o.CustomerId = c.Id GROUP BY c.Id;
```

`COUNT(*)` counts rows including the single `NULL`-filled placeholder row produced for an unmatched customer; `COUNT(o.Id)` counts only non-null `Id` values, correctly yielding `0` for a customer with no orders instead of `1`.

## Application

Use inner joins when only matched pairs matter. Use left joins specifically to answer "which rows exist with or without a match" questions (customers with no orders, products never sold), and always place any filter on the right-hand table inside the `ON` clause, not `WHERE`, if unmatched rows must be preserved.

## Common Mistakes

- Filtering on the right-hand table's column in `WHERE` after a `LEFT JOIN`, silently converting it into an inner join.
- Using `COUNT(*)` instead of `COUNT(column)` after an outer join, miscounting rows that only exist due to the join's `NULL`-filled placeholder.
- Forgetting that a `FULL OUTER JOIN` can reveal orphaned rows (e.g., an order with no matching customer) that indicate a referential-integrity problem elsewhere.
- Joining on a column without an index, causing a full table scan on one or both sides for large tables (see `indexes.md`).

## Common Interview Questions

### Basic
- What's the difference between an inner join and a left join?
- What does a left join return for rows in the left table with no match?

### Intermediate
- Why does filtering a left-joined column in `WHERE` silently behave like an inner join?
- What's the difference between `COUNT(*)` and `COUNT(column)` after a left join?

### Advanced
- How would you write a query to find all customers who have never placed an order?
- What does a `FULL OUTER JOIN` revealing unmatched rows on both sides typically indicate about data integrity?

### Follow-up Questions
- Is a `RIGHT JOIN` ever necessary, or can it always be rewritten as a `LEFT JOIN`?
- Does join order matter for correctness with inner joins? With outer joins?

### Code Prediction
Given `LEFT JOIN Orders o ON o.CustomerId = c.Id WHERE o.Total > 100`, does a customer with zero orders appear in the result set? What change makes it appear alongside customers who do have qualifying orders?

## Practical Tasks

- Write a query finding all customers with zero orders using a left join and an `IS NULL` check.
- Reproduce the `WHERE`-clause outer-join bug, observe the incorrect result, and fix it by moving the condition into the `ON` clause.
- Write a query correctly counting orders per customer, including customers with zero orders, using `COUNT(column)` rather than `COUNT(*)`.

## Readiness Criteria

Choose the correct join type for a given question, avoid the `WHERE`-clause outer-join trap, and correctly count/aggregate across joined tables including unmatched rows.

## References

### Other

- [PostgreSQL: Joins between tables](https://www.postgresql.org/docs/current/tutorial-join.html)
- [SQL Server: Join fundamentals](https://learn.microsoft.com/sql/relational-databases/performance/joins)
