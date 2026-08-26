# Testing EF-Based Code

## Definition

Testing code that depends on `DbContext` has three common approaches: an **in-memory provider** (EF Core's `UseInMemoryDatabase`, a simplified fake database for tests), **SQLite in-memory mode** (a real relational database, but not the same engine as production), and **Testcontainers** (spinning up the actual production database engine — SQL Server, PostgreSQL — in a container for the test run). Each makes a different trade-off between test speed and fidelity to production behavior.

```csharp
var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString())
    .Options;
using var context = new AppDbContext(options);
```

## Alternatives & Trade-offs

The in-memory provider is fast and simple to set up, but it isn't a relational database at all — it doesn't enforce foreign keys the same way, doesn't translate LINQ through a real SQL provider, and can silently pass tests for queries that would fail (or behave differently) against a real database. SQLite in-memory mode is a genuine relational database, closer to real SQL behavior, but is still a different engine than production (no window functions in older SQLite versions, different type behavior). Testcontainers runs the actual production database engine, giving the highest fidelity, at the cost of slower test startup and requiring Docker in the test environment.

## How It Works

### In-memory provider — fast, but not real SQL

```csharp
var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseInMemoryDatabase(databaseName: Guid.NewGuid().ToString()) // unique per test to avoid cross-test pollution
    .Options;

using var context = new AppDbContext(options);
context.Orders.Add(new Order { CustomerId = 7 });
await context.SaveChangesAsync();
```

This is fine for testing basic change-tracking and repository logic, but a query using a database-specific function, or a test intended to catch a real foreign-key violation, won't behave the same way it would against SQL Server or PostgreSQL.

### SQLite in-memory — closer to real relational behavior

```csharp
var connection = new SqliteConnection("DataSource=:memory:");
connection.Open(); // must stay open for the lifetime of the in-memory database

var options = new DbContextOptionsBuilder<AppDbContext>().UseSqlite(connection).Options;
using var context = new AppDbContext(options);
context.Database.EnsureCreated(); // creates the schema from the current model
```

Actual SQL is generated and executed against a real (if lightweight) relational engine — catching more real bugs than the in-memory provider, though still not identical to the production database engine's specific behavior.

### Testcontainers — the real production database

```csharp
await using var sqlContainer = new MsSqlBuilder().Build();
await sqlContainer.StartAsync();

var options = new DbContextOptionsBuilder<AppDbContext>()
    .UseSqlServer(sqlContainer.GetConnectionString())
    .Options;
using var context = new AppDbContext(options);
await context.Database.MigrateAsync(); // run real migrations against the real engine
```

This gives the highest confidence that tests reflect actual production behavior — including database-specific SQL, real constraint enforcement, and real query translation — at the cost of slower test execution and a Docker dependency in CI.

### Testing repository/service logic without hitting a database at all

```csharp
// Unit test: mock/fake the repository interface entirely, no DbContext involved
var fakeRepository = new FakeOrderRepository();
var service = new OrderService(fakeRepository);
```

Not every test needs a real or simulated database at all — pure business logic sitting above the repository layer (see Module 4's repository pattern) can often be tested with a fake implementation, reserving database-backed tests specifically for the persistence layer itself.

## Application

Use fast, fake repositories for testing business logic that doesn't need real database behavior. Use SQLite in-memory or the EF Core in-memory provider for quick persistence-layer tests where perfect fidelity isn't critical. Use Testcontainers for integration tests that specifically need to validate real database-specific behavior — constraint enforcement, migrations, database-specific SQL functions.

## Common Mistakes

- Relying entirely on the in-memory provider for tests that need to validate real relational behavior (foreign key enforcement, specific SQL translation), then being surprised when production behaves differently.
- Testing pure business logic against a real or simulated database when a fake repository would be faster and sufficient.
- Sharing one in-memory database instance across multiple tests without unique names/isolation, causing test pollution and flaky results.
- Never running any tests against the actual production database engine, missing real bugs that only the in-memory or SQLite alternatives can't reveal.

## Common Interview Questions

### Basic
- What are the three common approaches to testing EF Core-based code?
- What's the main limitation of EF Core's in-memory provider?

### Intermediate
- Why might a test pass against the in-memory provider but fail against a real SQL Server database?
- What does Testcontainers provide that SQLite in-memory mode doesn't?

### Advanced
- How would you decide which of the three testing approaches is appropriate for a given test — a repository unit test, an integration test, or a migration test?
- How would you structure a test suite to balance fast feedback (in-memory) with high confidence (Testcontainers) without every test paying the slowest cost?

### Follow-up Questions
- Does the in-memory provider enforce foreign key constraints the same way a real database does?
- Can Testcontainers run real EF Core migrations as part of test setup?

### Code Prediction
A test uses the EF Core in-memory provider and inserts two orders with duplicate values in a column that has a `UNIQUE` constraint configured in the real database. Does the in-memory provider test catch this as a failure, the way it would against a real SQL Server database?

## Practical Tasks

- Write a repository test using the EF Core in-memory provider and identify one behavior it wouldn't catch that a real database would.
- Set up a SQLite in-memory test for the same repository and compare its behavior against the in-memory provider.
- Set up a Testcontainers-based integration test running real migrations against a containerized SQL Server or PostgreSQL instance.

## Readiness Criteria

Choose the appropriate testing approach for a given scenario based on the fidelity/speed trade-off, and recognize which real-database behaviors each approach does and doesn't validate.

## References

### Microsoft Learn

- [Testing code that uses EF Core](https://learn.microsoft.com/ef/core/testing/)

### Other

- [Testcontainers for .NET](https://dotnet.testcontainers.org/)
