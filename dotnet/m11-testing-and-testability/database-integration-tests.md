# Database Integration Tests and Testcontainers

## Definition

A database integration test verifies code against a real database engine rather than a fake or simplified substitute — catching real SQL translation issues, real constraint enforcement, and real migration behavior that Module 10's testing-fidelity discussion flagged as gaps in the EF Core in-memory provider. **Testcontainers** spins up the actual production database engine in a Docker container for the duration of the test run, then tears it down afterward.

```csharp
public class OrderRepositoryTests : IAsyncLifetime
{
    private readonly MsSqlContainer _sqlContainer = new MsSqlBuilder().Build();
    private AppDbContext _context = null!;

    public async Task InitializeAsync()
    {
        await _sqlContainer.StartAsync();
        var options = new DbContextOptionsBuilder<AppDbContext>().UseSqlServer(_sqlContainer.GetConnectionString()).Options;
        _context = new AppDbContext(options);
        await _context.Database.MigrateAsync(); // run the actual production migrations
    }

    public Task DisposeAsync() => _sqlContainer.DisposeAsync().AsTask();
}
```

## Alternatives & Trade-offs

The EF Core in-memory provider and SQLite in-memory mode (from Module 10) are fast but don't perfectly replicate the production database engine's behavior. Testcontainers runs the real engine, giving the highest confidence that tests reflect actual production behavior, at the cost of slower startup (pulling and starting a container) and a Docker dependency in the test/CI environment. The right choice is usually a mix: fast fakes/in-memory for most tests, a smaller set of Testcontainers-based tests specifically for the persistence layer where real-engine fidelity matters most.

## How It Works

### Running real migrations against a real, disposable database

```csharp
public async Task InitializeAsync()
{
    await _sqlContainer.StartAsync();
    var context = CreateContext(_sqlContainer.GetConnectionString());
    await context.Database.MigrateAsync(); // the exact migrations that will run in production
}
```

Running actual migrations (rather than `EnsureCreated()`, which generates a schema directly from the current model, bypassing migrations entirely) verifies that the migrations themselves are correct — catching the exact kind of "the generated migration silently dropped data" bug covered in Module 10's `migrations.md`.

### Test isolation with a real database — per-test cleanup

```csharp
[Fact]
public async Task AddOrder_PersistsToDatabase()
{
    await using var transaction = await _context.Database.BeginTransactionAsync();
    _context.Orders.Add(new Order { CustomerId = 7 });
    await _context.SaveChangesAsync();

    var saved = await _context.Orders.FirstOrDefaultAsync(o => o.CustomerId == 7);
    Assert.NotNull(saved);

    await transaction.RollbackAsync(); // undo this test's changes, keeping the shared container clean for the next test
}
```

### Sharing one container across a test suite for speed, isolating per-test

```csharp
[CollectionDefinition("Database")]
public class DatabaseCollection : ICollectionFixture<DatabaseContainerFixture> { }
// one container started once for the whole test run, shared via ICollectionFixture,
// with each individual test still isolating its own data via a rolled-back transaction
```

Starting a fresh container per test would be safest for isolation but prohibitively slow; sharing one container across the suite (via the fixture patterns from earlier in this module) while isolating each test's data changes gets most of the speed benefit without the cross-test pollution risk.

## Application

Use Testcontainers for the persistence-layer tests specifically meant to validate real database behavior — migrations, constraint enforcement, database-specific query translation. Combine with the fixture-sharing and transaction-rollback isolation patterns already covered in this module to keep the suite reasonably fast despite using a real engine.

## Common Mistakes

- Relying entirely on the EF Core in-memory provider for tests that specifically need to validate real database behavior (as flagged in Module 10), missing bugs that only appear against the real engine.
- Starting a fresh container per test instead of sharing one across a suite with proper per-test isolation, making the test suite unnecessarily slow.
- Using `EnsureCreated()` instead of running real migrations, missing the chance to catch migration-specific bugs the test could otherwise reveal.
- Not accounting for Testcontainers' Docker dependency in CI environments that might not support it, without a documented fallback or explicit CI configuration.

## Common Interview Questions

### Basic
- What is Testcontainers, and what problem does it solve for database testing?
- Why might a test against a real database catch bugs that the EF Core in-memory provider wouldn't?

### Intermediate
- How would you keep a Testcontainers-based test suite reasonably fast despite using a real database engine?
- What's the difference between running real migrations versus `EnsureCreated()` in a test's setup?

### Advanced
- How would you design a test suite balancing fast, fake-based unit tests with a smaller set of high-fidelity Testcontainers-based integration tests?
- What CI environment considerations does adopting Testcontainers introduce?

### Follow-up Questions
- Does a Testcontainers-based test require Docker to be available in the environment running the tests?
- Can Testcontainers be used for databases other than SQL Server or PostgreSQL?

### Code Prediction
A migration silently converts a column rename into drop-and-add (the exact bug covered in Module 10's `migrations.md`), destroying existing data. Would a test using the EF Core in-memory provider with `EnsureCreated()` catch this bug? Would a Testcontainers-based test running the real migration catch it?

## Practical Tasks

- Set up a Testcontainers-based test running real EF Core migrations against a containerized SQL Server or PostgreSQL instance.
- Implement per-test isolation for a shared Testcontainers instance using rolled-back transactions.
- Compare what a migration bug reveals under the EF Core in-memory provider versus a Testcontainers-based test.

## Readiness Criteria

Set up Testcontainers-based database integration tests with real migrations, balance test suite speed against fidelity using shared containers and per-test isolation, and know when this level of testing is actually warranted.

## References

### Other

- [Testcontainers for .NET](https://dotnet.testcontainers.org/)
