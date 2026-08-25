# SQL Injection Prevention

## Definition

SQL injection is an attack where untrusted input is concatenated directly into a SQL statement, letting an attacker alter the query's structure and logic — reading unauthorized data, bypassing authentication, or modifying/deleting data. Prevention is straightforward and near-total: never build SQL by string-concatenating input; always use **parameterized queries**.

```csharp
// VULNERABLE: user input becomes part of the SQL statement's structure
var sql = $"SELECT * FROM Users WHERE Username = '{username}'";

// SAFE: username is passed as a parameter value, never interpreted as SQL
var sql = "SELECT * FROM Users WHERE Username = @Username";
command.Parameters.AddWithValue("@Username", username);
```

## Alternatives & Trade-offs

There isn't really a trade-off here — parameterized queries cost nothing in readability or performance (many databases even cache the query plan more effectively when the same parameterized statement is reused with different values) and eliminate the entire vulnerability class. String concatenation of SQL is never actually necessary for passing values; it only ever appears from either unfamiliarity or, rarely, genuinely needing to parameterize an *identifier* (a table or column name), which parameters can't do and requires a different, more careful mitigation (a strict allow-list, never raw concatenation of user input).

## How It Works

### The attack

```csharp
var username = "admin' --";
var sql = $"SELECT * FROM Users WHERE Username = '{username}' AND Password = '{password}'";
// becomes: SELECT * FROM Users WHERE Username = 'admin' --' AND Password = '...'
// the -- comments out the rest of the query, bypassing the password check entirely
```

```csharp
var username = "'; DROP TABLE Users; --";
// a sufficiently permissive database driver/context could execute this as two statements,
// destroying the Users table entirely
```

### Parameterized queries close the vulnerability structurally, not just by sanitizing

```csharp
using var command = new SqlCommand("SELECT * FROM Users WHERE Username = @Username", connection);
command.Parameters.AddWithValue("@Username", username);
// the database driver sends the query text and the parameter value SEPARATELY —
// the value can never be interpreted as part of the SQL statement's structure,
// regardless of what characters it contains
```

This is a structural guarantee, not input filtering — even `username = "'; DROP TABLE Users; --"` is treated as a literal string value to compare against, never as SQL syntax, because the query structure was already fixed before the parameter value was ever supplied.

### EF Core and Dapper parameterize automatically

```csharp
// EF Core: LINQ is translated to parameterized SQL automatically
var user = await context.Users.FirstOrDefaultAsync(u => u.Username == username);

// Dapper: also parameterizes automatically when using its parameter syntax
var user = await connection.QueryFirstOrDefaultAsync<User>(
    "SELECT * FROM Users WHERE Username = @Username", new { Username = username });
```

Using an ORM or a query library correctly is usually sufficient protection by default — the risk reappears specifically when raw SQL is built manually, including inside EF Core's own raw-SQL escape hatches.

### Raw SQL inside an ORM is still vulnerable if concatenated

```csharp
// Still vulnerable, even though it's EF Core — the raw SQL string is concatenated with user input
var sql = $"SELECT * FROM Users WHERE Username = '{username}'";
context.Users.FromSqlRaw(sql);

// Safe: parameter placeholders passed separately, even within a raw SQL escape hatch
context.Users.FromSqlInterpolated($"SELECT * FROM Users WHERE Username = {username}");
```

`FromSqlInterpolated` looks like string interpolation but actually parameterizes the interpolated values safely — a subtle but important distinction from `FromSqlRaw` with manual concatenation.

## Application

Always use parameterized queries (or an ORM/query library that generates them automatically) for any SQL built with values from external input — request bodies, query strings, headers, even values from another internal system that isn't fully trusted. Treat any raw-SQL escape hatch in an ORM with the same discipline as hand-written ADO.NET.

## Common Mistakes

- Concatenating or string-interpolating untrusted input directly into a SQL statement, even for a value that "seems safe" like a numeric ID (a string-typed numeric-looking parameter can still carry injection payloads if not actually parameterized).
- Using an ORM's raw-SQL escape hatch (`FromSqlRaw`, ADO.NET raw commands) with string concatenation, losing the automatic protection the ORM provides for its normal LINQ-based queries.
- Believing input validation/sanitization alone is sufficient protection instead of using parameterized queries — validation can reduce risk but is not a structural guarantee the way parameterization is.
- Attempting to parameterize a table or column name (an identifier) using the same mechanism as a value parameter — most drivers don't support this, and identifiers require a strict allow-list check instead.

## Common Interview Questions

### Basic
- What is SQL injection, and how does it happen?
- What is a parameterized query, and how does it prevent injection?

### Intermediate
- Why is input validation alone not considered sufficient protection against SQL injection?
- What's the difference between `FromSqlRaw` and `FromSqlInterpolated` in EF Core, in terms of injection safety?

### Advanced
- Why can't table or column names be parameterized the same way values can, and how would you safely accept a user-selected sort column?
- Explain precisely why a parameterized query is immune to injection regardless of the parameter's content, in terms of how the database driver processes the statement.

### Follow-up Questions
- Does using an ORM automatically guarantee protection against SQL injection?
- Is a numeric-looking parameter (like an ID) ever safe to concatenate directly, if it's validated as numeric first?

### Code Prediction
Given `var sql = $"SELECT * FROM Orders WHERE Id = {id}";` where `id` comes from a route parameter already constrained by `{id:int}` at the routing layer (see Module 7), is this specific case actually exploitable for SQL injection? Why does routing-level type constraint change the risk calculus here compared to a raw string input?

## Practical Tasks

- Identify and fix a SQL-injection vulnerability in a given piece of ADO.NET code using parameterized queries.
- Compare `FromSqlRaw` and `FromSqlInterpolated` usage for the same query and identify which is safe.
- Design a safe approach for accepting a user-selected sort column name without allowing arbitrary SQL injection through an identifier.

## Readiness Criteria

Explain why parameterized queries provide a structural (not just sanitization-based) guarantee against SQL injection, correctly use safe raw-SQL patterns in an ORM, and safely handle the identifier-parameterization edge case.

## References

### Microsoft Learn

- [SQL injection](https://learn.microsoft.com/sql/relational-databases/security/sql-injection)
- [EF Core: raw SQL queries](https://learn.microsoft.com/ef/core/querying/sql-queries)

### Other

- [OWASP: SQL Injection](https://owasp.org/www-community/attacks/SQL_Injection)
