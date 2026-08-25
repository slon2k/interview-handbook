# Views and Stored Procedures

## Definition

A **view** is a named, saved query that behaves like a virtual table — querying it re-runs the underlying query each time (unless it's a materialized view, which caches results). A **stored procedure** is a named, precompiled batch of SQL (including control flow, parameters, and multiple statements) invoked as a unit, typically used for reusable business operations rather than simple reads.

```sql
CREATE VIEW ActiveCustomerOrders AS
SELECT c.Name, o.Id, o.Total
FROM Customers c
JOIN Orders o ON o.CustomerId = c.Id
WHERE c.IsActive = 1;

SELECT * FROM ActiveCustomerOrders WHERE Total > 100;
```

## Alternatives & Trade-offs

A view simplifies a commonly-used, complex query into a reusable name, and can also restrict which columns/rows a set of users can see (a security/access-control use). A stored procedure centralizes multi-statement logic (including transactions and conditional flow) in the database itself, which can reduce round-trips and centralize business rules that must run consistently regardless of caller — at the cost of splitting business logic between application code and the database, which can make a codebase harder to understand and test as a whole, and ties that logic to a specific database engine's procedural dialect.

## How It Works

### Views for query reuse and access control

```sql
CREATE VIEW CustomerSummary AS
SELECT Id, Name, Email FROM Customers; -- deliberately excludes sensitive columns like PasswordHash

GRANT SELECT ON CustomerSummary TO ReportingRole; -- ReportingRole never sees the underlying full table
```

A view can expose a restricted subset of columns/rows to a role that shouldn't have access to the full underlying table — a lightweight access-control mechanism independent of application-layer permission checks.

### Materialized views — trading freshness for read speed

```sql
CREATE MATERIALIZED VIEW MonthlySalesSummary AS
SELECT DATE_TRUNC('month', OrderDate) AS Month, SUM(Total) AS TotalSales
FROM Orders GROUP BY DATE_TRUNC('month', OrderDate);

REFRESH MATERIALIZED VIEW MonthlySalesSummary; -- must be explicitly refreshed; not automatically live
```

Unlike a regular view, a materialized view's results are computed and stored once, then served instantly on read — but the data is only as fresh as the last refresh, which is a deliberate consistency/performance trade-off.

### Stored procedures for multi-statement operations

```sql
CREATE PROCEDURE PlaceOrder
    @CustomerId INT, @ProductId INT, @Quantity INT
AS
BEGIN
    BEGIN TRANSACTION;
    UPDATE Inventory SET Stock = Stock - @Quantity WHERE ProductId = @ProductId AND Stock >= @Quantity;
    IF @@ROWCOUNT = 0
    BEGIN
        ROLLBACK TRANSACTION;
        THROW 50000, 'Insufficient stock', 1;
    END
    INSERT INTO Orders (CustomerId, ProductId, Quantity) VALUES (@CustomerId, @ProductId, @Quantity);
    COMMIT TRANSACTION;
END;
```

```sql
EXEC PlaceOrder @CustomerId = 7, @ProductId = 3, @Quantity = 2;
```

## Application

Use views to simplify frequently-repeated complex queries and to restrict column/row access for specific roles. Use materialized views for expensive aggregations that don't need to be perfectly real-time. Use stored procedures sparingly in a modern .NET codebase — mostly for operations genuinely needing tight transactional coupling with minimal round-trips — since most application logic today lives in the application layer (via an ORM or Dapper) rather than split into database-side procedural code, which keeps business logic in one place, in a testable, source-controlled language.

## Common Mistakes

- Scattering business logic across many stored procedures, making it hard to understand, test, or version alongside the rest of the application codebase.
- Assuming a regular (non-materialized) view improves performance — it doesn't cache anything; it's simply a saved query text that re-executes fully every time it's queried.
- Forgetting to refresh a materialized view and treating its data as real-time when it's only as fresh as the last refresh.
- Using views purely for convenience without considering the access-control benefit they can also provide, missing an opportunity to simplify permission management.

## Common Interview Questions

### Basic
- What is a view, and how does it differ from a regular table?
- What is a stored procedure, and when might you use one?

### Intermediate
- What is a materialized view, and what trade-off does it make compared to a regular view?
- Why might a view be used for access control rather than just query simplification?

### Advanced
- What are the trade-offs of putting business logic in stored procedures versus keeping it entirely in application code?
- How would you decide whether a specific expensive aggregation query is a good candidate for a materialized view?

### Follow-up Questions
- Does querying a regular (non-materialized) view execute the underlying query each time?
- Can a view be updatable (support `INSERT`/`UPDATE` through it) under some conditions?

### Code Prediction
A materialized view summarizing daily sales was last refreshed at midnight. A new order is placed at 9 AM. Does querying the materialized view at 10 AM reflect that new order? What would need to happen for it to?

## Practical Tasks

- Create a view restricting access to a subset of columns of a sensitive table, and grant a role access to the view instead of the underlying table.
- Create a materialized view for an expensive aggregation and demonstrate the effect of refreshing it.
- Write a stored procedure that performs a multi-step operation within a single transaction, including a rollback path for a failure condition.

## Readiness Criteria

Explain the difference between views, materialized views, and stored procedures, and make a deliberate, justified choice about where business logic should live rather than defaulting to one approach.

## References

### Other

- [PostgreSQL: Views](https://www.postgresql.org/docs/current/sql-createview.html)
- [PostgreSQL: Materialized views](https://www.postgresql.org/docs/current/rules-materializedviews.html)
- [SQL Server: Stored procedures](https://learn.microsoft.com/sql/relational-databases/stored-procedures/stored-procedures-database-engine)
