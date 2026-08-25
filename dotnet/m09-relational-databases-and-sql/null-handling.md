# NULL Handling and Three-Valued Logic

## Definition

`NULL` represents the absence of a known value — not zero, not an empty string, not "false." SQL's comparison logic is **three-valued**: any expression involving `NULL` evaluates to `TRUE`, `FALSE`, or `UNKNOWN`, and `UNKNOWN` is treated as not-true everywhere a boolean result is needed (`WHERE`, `HAVING`, `JOIN ON` conditions all exclude rows evaluating to `UNKNOWN`).

```sql
SELECT * FROM Orders WHERE Discount = NULL;  -- always returns zero rows — this comparison is always UNKNOWN
SELECT * FROM Orders WHERE Discount IS NULL; -- correct way to test for NULL
```

## Alternatives & Trade-offs

Three-valued logic is simply how the SQL standard models "unknown" rigorously, and every relational database follows it — there's no alternative to learn instead, only the discipline of writing comparisons that account for it correctly. The practical trade-off is between `NULL` (a real absence of data, correctly excluded from most aggregates and comparisons) and a sentinel value like `0` or `''` (which participates in ordinary comparisons and arithmetic but conflates "no value" with "a real value that happens to be zero/empty").

## How It Works

### `NULL = NULL` is `UNKNOWN`, not `TRUE`

```sql
SELECT NULL = NULL;   -- UNKNOWN (not TRUE)
SELECT NULL <> NULL;  -- also UNKNOWN (not TRUE)
SELECT 5 = NULL;      -- UNKNOWN
```

Any direct comparison against `NULL` using `=` or `<>` always yields `UNKNOWN`, which behaves like `FALSE` for filtering purposes — this is why `WHERE column = NULL` never matches any row, even rows that are actually `NULL` in that column.

### The correct way to test for `NULL`

```sql
SELECT * FROM Orders WHERE Discount IS NULL;
SELECT * FROM Orders WHERE Discount IS NOT NULL;
```

### `COALESCE` for providing a fallback value

```sql
SELECT Name, COALESCE(Discount, 0) AS EffectiveDiscount FROM Orders;
-- returns Discount's value if present, otherwise 0 — without changing what's actually stored
```

### `NULL` in aggregates and `NOT IN`

```sql
SELECT AVG(Discount) FROM Orders; -- NULLs are excluded, not treated as 0 — the average is over non-NULL rows only

-- Dangerous: if the subquery returns even one NULL, the entire NOT IN comparison becomes UNKNOWN for every row
SELECT * FROM Customers WHERE Id NOT IN (SELECT CustomerId FROM Orders WHERE CustomerId IS NOT NULL);
-- (the IS NOT NULL guard above is required precisely because of this trap)
```

`NOT IN` combined with a subquery that can return `NULL` is one of the most common real-world SQL bugs: if any row from the subquery is `NULL`, `NOT IN` silently returns zero rows for the entire outer query, with no error — because comparing against a `NULL` in the list makes every comparison `UNKNOWN`.

### `AND`/`OR` with `UNKNOWN`

```sql
TRUE AND UNKNOWN   -- UNKNOWN
FALSE AND UNKNOWN  -- FALSE (short-circuits correctly)
TRUE OR UNKNOWN    -- TRUE (short-circuits correctly)
FALSE OR UNKNOWN   -- UNKNOWN
```

## Application

Always use `IS NULL`/`IS NOT NULL` to test for `NULL`, never `=`/`<>`. Use `COALESCE` to provide fallback values for display or calculation without altering stored data. Be specifically cautious with `NOT IN` against a subquery that could produce `NULL` — prefer `NOT EXISTS` instead, which doesn't have this trap.

## Common Mistakes

- Writing `WHERE column = NULL` expecting it to match `NULL` rows, when it always evaluates to `UNKNOWN` and matches nothing.
- Using `NOT IN` with a subquery that can return `NULL`, silently causing the entire query to return zero rows.
- Assuming `AVG`/`SUM` treat `NULL` as zero, when they actually exclude `NULL` rows from the calculation entirely — which can produce a different result than expected if "no value" was meant to mean "zero."
- Forgetting that `UNKNOWN` isn't the same as `FALSE` logically, even though it behaves like `FALSE` for row-filtering purposes — this distinction matters when reasoning through complex `AND`/`OR` conditions involving `NULL`.

## Common Interview Questions

### Basic
- Why doesn't `column = NULL` work to find `NULL` rows?
- What is three-valued logic in SQL?

### Intermediate
- Why is `NOT IN` risky when the subquery can return `NULL` values, and what's the safer alternative?
- Does `AVG()` treat `NULL` values as zero?

### Advanced
- Walk through why `TRUE OR UNKNOWN` evaluates to `TRUE` but `FALSE AND UNKNOWN` evaluates to `FALSE`, while `TRUE AND UNKNOWN` and `FALSE OR UNKNOWN` both evaluate to `UNKNOWN`.
- How would you rewrite a `NOT IN` subquery pattern to avoid the `NULL` trap, and why does `NOT EXISTS` avoid it?

### Follow-up Questions
- Is `UNKNOWN` a value that can be stored in a column?
- Does `DISTINCT` treat multiple `NULL` values as duplicates of each other?

### Code Prediction
Given `SELECT * FROM Customers WHERE Id NOT IN (SELECT CustomerId FROM Orders)`, and the `Orders.CustomerId` column contains at least one `NULL` (perhaps from an anonymous/guest order), what does this query return — the expected set of customers with no orders, all customers, or zero rows?

## Practical Tasks

- Reproduce the `NOT IN` with `NULL` trap and rewrite the query using `NOT EXISTS` to fix it.
- Write a query using `COALESCE` to display a fallback value for a nullable column without modifying stored data.
- Trace through the truth table for `AND`/`OR` combined with `UNKNOWN` for a moderately complex `WHERE` clause.

## Readiness Criteria

Correctly test for `NULL` using `IS NULL`/`IS NOT NULL`, explain three-valued logic and its effect on filtering and aggregation, and recognize and fix the `NOT IN`/`NULL` subquery trap.

## References

### Other

- [PostgreSQL: NULL values](https://www.postgresql.org/docs/current/functions-comparison.html)
- [SQL Server: NULL values (Transact-SQL)](https://learn.microsoft.com/sql/t-sql/language-elements/null-and-unknown-transact-sql)
