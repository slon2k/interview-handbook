# Raw SQL in EF Core

## Definition

EF Core lets you drop down to raw SQL when LINQ can't express what's needed — a database-specific function, a query the LINQ provider translates inefficiently, or an operation LINQ has no equivalent for. `FromSqlInterpolated`/`FromSqlRaw` run raw SQL and map results to entities; `ExecuteSqlInterpolated`/`ExecuteSqlRaw` run raw SQL with no result mapping (for direct updates/deletes not going through change tracking).

```csharp
var orders = await context.Orders
    .FromSqlInterpolated($"SELECT * FROM Orders WHERE Total > {minTotal}")
    .ToListAsync();
```

## Alternatives & Trade-offs

LINQ-to-SQL translation covers the large majority of everyday queries without ever needing raw SQL, and keeps queries database-agnostic and refactor-safe. Raw SQL is the right escape hatch specifically when the LINQ provider can't translate what's needed, or when a hand-tuned query outperforms what the provider generates — at the cost of losing some of LINQ's compile-time safety and database independence, and reintroducing the SQL-injection discipline from Module 9 if not done carefully.

## How It Works

### `FromSqlInterpolated` — safe parameterization, looks like string interpolation but isn't

```csharp
var minTotal = 100m;
var orders = await context.Orders
    .FromSqlInterpolated($"SELECT * FROM Orders WHERE Total > {minTotal}")
    .ToListAsync();
// EF Core treats {minTotal} as a genuine SQL parameter, not string-concatenated — safe from injection,
// exactly like the FromSqlInterpolated example in Module 9's sql-injection-prevention.md
```

### `FromSqlRaw` — must be parameterized manually, easy to get wrong

```csharp
// SAFE: parameter placeholder used explicitly
var orders = await context.Orders
    .FromSqlRaw("SELECT * FROM Orders WHERE Total > {0}", minTotal)
    .ToListAsync();

// VULNERABLE: string-concatenated, exactly the SQL injection risk from Module 9
var orders = await context.Orders
    .FromSqlRaw($"SELECT * FROM Orders WHERE Total > {minTotal}") // DO NOT DO THIS
    .ToListAsync();
```

### Composing LINQ on top of raw SQL — with restrictions

```csharp
var orders = await context.Orders
    .FromSqlInterpolated($"SELECT * FROM Orders WHERE Total > {minTotal}")
    .Where(o => o.Status == "Pending") // additional LINQ composes on top, translated and appended
    .ToListAsync();
```

Composed LINQ on top of raw SQL works for filtering/ordering, but the raw SQL itself must return columns matching the entity's shape — you can't project inside the raw SQL string and still map to the full entity type.

### `ExecuteSqlInterpolated` — for direct writes with no entity mapping or change tracking

```csharp
var rowsAffected = await context.Database.ExecuteSqlInterpolatedAsync(
    $"UPDATE Orders SET Status = 'Cancelled' WHERE OrderDate < {cutoffDate}");
```

This bypasses change tracking entirely — useful (and often faster) for bulk operations that don't need entity-level business logic to run, but see `bulk-operations.md` for EF Core's more structured `ExecuteUpdate`/`ExecuteDelete` alternative.

## Application

Reach for raw SQL when a specific query needs a database feature or hand-tuned performance LINQ translation can't provide. Always use `FromSqlInterpolated`/parameterized `FromSqlRaw`, never string-concatenated SQL, applying the exact same injection discipline covered in Module 9 regardless of being inside an ORM.

## Common Mistakes

- String-concatenating values into `FromSqlRaw`, recreating the SQL injection vulnerability an ORM is otherwise assumed to protect against by default.
- Assuming `FromSqlInterpolated`'s string-interpolation-like syntax means it isn't parameterized — it is, safely, despite looking like plain string interpolation.
- Trying to project columns inside the raw SQL string while still mapping to a full entity type, when the raw SQL's shape must match the entity exactly for entity mapping to work.
- Reaching for raw SQL prematurely for something LINQ actually can express perfectly well, adding unnecessary loss of database independence and compile-time safety.

## Common Interview Questions

### Basic
- When would you use raw SQL instead of LINQ in EF Core?
- What's the difference between `FromSqlInterpolated` and `FromSqlRaw`?

### Intermediate
- Why is `FromSqlInterpolated` safe from SQL injection despite looking like string interpolation?
- What's the difference between `FromSqlRaw`/`FromSqlInterpolated` and `ExecuteSqlRaw`/`ExecuteSqlInterpolated`?

### Advanced
- How would you safely compose additional LINQ filtering on top of a raw SQL query?
- What are the risks and appropriate use cases of bypassing change tracking with `ExecuteSqlInterpolated` for a direct update?

### Follow-up Questions
- Can raw SQL results be mapped to a type that isn't a full mapped entity?
- Does `FromSqlRaw` require explicit parameterization, or does it handle it automatically like `FromSqlInterpolated`?

### Code Prediction
Given `context.Orders.FromSqlRaw($"SELECT * FROM Orders WHERE CustomerName = '{name}'")` where `name` comes from user input, is this code vulnerable to SQL injection? What single change makes it safe?

## Practical Tasks

- Write a query using `FromSqlInterpolated` for a scenario LINQ can't express directly, and verify the generated parameter is safely handled.
- Identify and fix a `FromSqlRaw` call using string concatenation instead of proper parameterization.
- Compose additional LINQ filtering on top of a raw SQL query and verify the combined SQL executed.

## Readiness Criteria

Use raw SQL safely and only when genuinely needed, correctly parameterize both `FromSqlInterpolated` and `FromSqlRaw`, and apply the same SQL-injection discipline inside an ORM as outside one.

## References

### Microsoft Learn

- [Raw SQL queries](https://learn.microsoft.com/ef/core/querying/sql-queries)
