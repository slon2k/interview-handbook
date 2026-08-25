# Constraints

## Definition

A constraint is a rule the database enforces on data automatically, rejecting any statement that would violate it. Beyond primary and foreign keys (covered separately), the common constraint types are `NOT NULL` (a value is required), `UNIQUE` (no duplicate values across rows), `CHECK` (an arbitrary boolean condition must hold), and `DEFAULT` (a value used when none is provided).

```sql
CREATE TABLE Products (
    Id INT PRIMARY KEY IDENTITY,
    Sku NVARCHAR(50) NOT NULL UNIQUE,
    Price DECIMAL(10,2) NOT NULL CHECK (Price >= 0),
    Status NVARCHAR(20) NOT NULL DEFAULT 'Active'
);
```

## Alternatives & Trade-offs

Enforcing a rule as a database constraint guarantees it holds regardless of which application, script, or migration writes to the table — it cannot be bypassed by a code path that forgets to validate. Enforcing the same rule only in application code is more flexible to change without a migration, but leaves the database vulnerable to invalid data from any writer that doesn't go through that validation logic. Most systems use both: constraints as a non-negotiable safety net, application validation for richer, more specific error messages before the constraint is even reached.

## How It Works

### `CHECK` for business rules the database can enforce directly

```sql
ALTER TABLE Orders ADD CONSTRAINT CK_Orders_PositiveTotal CHECK (Total >= 0);
ALTER TABLE Employees ADD CONSTRAINT CK_Employees_ValidAge CHECK (Age BETWEEN 18 AND 100);
```

A `CHECK` constraint rejects any insert or update that would violate it, regardless of source — a benefit over relying solely on application-layer validation.

### `UNIQUE` vs. `PRIMARY KEY`

```sql
CREATE TABLE Users (
    Id INT PRIMARY KEY IDENTITY,   -- the main identifier, referenced by foreign keys
    Email NVARCHAR(200) NOT NULL UNIQUE  -- also must be unique, but isn't the primary identifier
);
```

A table can have only one primary key but multiple `UNIQUE` constraints — useful when a natural attribute (email, username) must be unique but a surrogate key is still used as the primary identifier.

### `DEFAULT` values

```sql
CREATE TABLE Orders (
    Id INT PRIMARY KEY IDENTITY,
    Status NVARCHAR(20) NOT NULL DEFAULT 'Pending',
    CreatedAt DATETIME2 NOT NULL DEFAULT SYSUTCDATETIME()
);

INSERT INTO Orders DEFAULT VALUES; -- Status becomes 'Pending', CreatedAt becomes the current timestamp
```

### Constraint violations fail loudly, not silently

```sql
INSERT INTO Products (Sku, Price) VALUES ('SKU-1', -5.00);
-- Error: violates CHECK constraint "CK_Products_Price" — the row is never inserted
```

This is the point: a constraint violation is a hard failure the application must handle, not a silent correction — catching it and surfacing a clear error (see `sql-injection-prevention.md`'s sibling concern of never trusting input) is far safer than allowing invalid data to persist.

## Application

Use `NOT NULL` for every column that's logically always required. Use `UNIQUE` for natural identifiers that must not duplicate, even when they aren't the primary key. Use `CHECK` for simple, self-contained business rules that should never be violated regardless of which application writes the data. Use `DEFAULT` to avoid requiring every insert statement to specify a value that usually doesn't vary.

## Common Mistakes

- Omitting `NOT NULL` on columns that are logically always required, allowing incomplete data that then must be defensively checked everywhere the column is read.
- Relying only on application-level checks for rules that could be enforced as a `CHECK` constraint, leaving the database vulnerable to invalid data from any other writer.
- Not handling constraint-violation exceptions gracefully in application code, letting a raw database error leak to the end user instead of a clear, friendly message.
- Adding overly complex business logic into `CHECK` constraints that would be better expressed and tested in application code, rather than reserving constraints for simple, universal invariants.

## Common Interview Questions

### Basic
- What are the main types of constraints beyond primary and foreign keys?
- What's the difference between `UNIQUE` and `PRIMARY KEY`?

### Intermediate
- Why would you enforce a rule as a `CHECK` constraint instead of only validating it in application code?
- What happens when an insert or update violates a constraint?

### Advanced
- How do you decide which business rules belong as database constraints versus application-level validation?
- How would you gracefully handle a constraint-violation exception in application code to surface a clear error rather than a raw database exception?

### Follow-up Questions
- Can a table have more than one `UNIQUE` constraint?
- Does adding a `NOT NULL` constraint to an existing column with existing `NULL` values succeed immediately?

### Code Prediction
Given `CHECK (Price >= 0)` on a `Products` table, what happens when application code (with a bug) attempts to insert a product with `Price = -10`? Does the invalid row get partially inserted, or does the entire statement fail?

## Practical Tasks

- Add appropriate `NOT NULL`, `UNIQUE`, `CHECK`, and `DEFAULT` constraints to a given table design.
- Reproduce a constraint-violation error and handle it gracefully in application code with a clear error message.
- Decide, for a set of business rules, which belong as database constraints and which belong in application-level validation.

## Readiness Criteria

Apply the appropriate constraint type for a given rule, explain why database-enforced constraints matter beyond application validation, and handle constraint violations gracefully in application code.

## References

### Other

- [PostgreSQL: Constraints](https://www.postgresql.org/docs/current/ddl-constraints.html)
- [SQL Server: Unique constraints and check constraints](https://learn.microsoft.com/sql/relational-databases/tables/unique-constraints-and-check-constraints)
