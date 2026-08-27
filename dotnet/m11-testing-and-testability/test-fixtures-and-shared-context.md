# Test Fixtures and Shared Context

## Definition

A test fixture is shared context/setup used across multiple tests — expensive-to-create resources (a database connection, a `WebApplicationFactory`) that shouldn't be recreated per test, but must still be managed carefully to avoid leaking state between tests. xUnit provides `IClassFixture<T>` (shared across tests in one class) and `ICollectionFixture<T>` (shared across multiple test classes).

```csharp
public class DatabaseFixture : IDisposable
{
    public AppDbContext Context { get; }
    public DatabaseFixture() => Context = CreateRealTestDbContext(); // expensive setup, done once
    public void Dispose() => Context.Dispose();
}

public class OrderTests : IClassFixture<DatabaseFixture>
{
    private readonly DatabaseFixture _fixture;
    public OrderTests(DatabaseFixture fixture) => _fixture = fixture; // same instance injected into every test in this class
}
```

## Alternatives & Trade-offs

Recreating expensive resources (a real database, a full `WebApplicationFactory`) for every single test is safe (no state leakage) but slow, especially across a large test suite. Sharing a fixture across tests avoids that cost, but reintroduces the exact risk xUnit's default per-test-instance model exists to prevent — one test's leftover state can silently affect another's — unless the fixture and tests are written carefully to reset or isolate whatever actually matters.

## How It Works

### `IClassFixture<T>` — shared within one test class

```csharp
public class OrderTests : IClassFixture<DatabaseFixture>
{
    public OrderTests(DatabaseFixture fixture) => _fixture = fixture; // constructed once per class, injected into every test
}
```

The fixture's constructor runs once for the whole class, not once per test method — that's the entire point, and also the entire risk: any state left in the fixture by one test is visible to the next.

### `ICollectionFixture<T>` — shared across multiple test classes

```csharp
[CollectionDefinition("Database collection")]
public class DatabaseCollection : ICollectionFixture<DatabaseFixture> { }

[Collection("Database collection")]
public class OrderTests { /* shares the same DatabaseFixture instance as other classes in this collection */ }

[Collection("Database collection")]
public class CustomerTests { /* also shares it */ }
```

Useful when the same expensive resource (one Testcontainers database instance, for example) should genuinely be shared across an entire suite of related test classes, not just one.

### Isolating each test's data within a shared fixture

```csharp
public class OrderTests : IClassFixture<DatabaseFixture>
{
    [Fact]
    public async Task Test1()
    {
        using var transaction = await _fixture.Context.Database.BeginTransactionAsync();
        // ... perform test operations ...
        await transaction.RollbackAsync(); // undo everything this test did, regardless of outcome
    }
}
```

Wrapping each individual test in a transaction that's always rolled back is a common way to get the speed benefit of a shared, expensive database connection while still keeping each test's data changes isolated from the next.

### Disposal — cleaning up the fixture when it's truly done

```csharp
public class DatabaseFixture : IDisposable
{
    public void Dispose() => Context.Dispose(); // called once, after every test in scope has finished
}
```

## Application

Use fixtures for genuinely expensive, safely-shareable setup — a real database connection, a `WebApplicationFactory`, a Testcontainers instance. Explicitly isolate per-test data changes within a shared fixture (via a rolled-back transaction, or resetting known mutable state) rather than assuming sharing is automatically safe.

## Common Mistakes

- Sharing a fixture across tests without isolating each test's data changes, causing one test's side effects to silently affect another's expected starting state.
- Creating an expensive resource per-test when it could safely be shared via a fixture, unnecessarily slowing down the whole suite.
- Forgetting to dispose fixture resources properly, leaking connections or containers across test runs.
- Using `IClassFixture<T>` when the actual need is `ICollectionFixture<T>` (or vice versa), either duplicating expensive setup unnecessarily or sharing state more broadly than intended.

## Common Interview Questions

### Basic
- What is a test fixture, and what problem does it solve?
- What's the difference between `IClassFixture<T>` and `ICollectionFixture<T>`?

### Intermediate
- What's the risk of sharing a fixture across multiple tests, and how would you mitigate it?
- Why might wrapping each test in a rolled-back transaction be a good pattern for a shared database fixture?

### Advanced
- How would you design a fixture sharing a single Testcontainers database instance across an entire test suite, while keeping each test's data isolated?
- What's the trade-off between fixture-level setup cost and the risk of state leakage between tests?

### Follow-up Questions
- Does xUnit guarantee tests within a class run in a specific order relative to fixture state?
- Can a fixture itself depend on another fixture?

### Code Prediction
Two tests in the same class share a `DatabaseFixture` via `IClassFixture<DatabaseFixture>`. The first test inserts a row and doesn't clean it up. What effect could this have on the second test if it queries the same table expecting a specific starting row count?

## Practical Tasks

- Implement a shared database fixture using `IClassFixture<T>`, with each test wrapped in a rolled-back transaction for isolation.
- Set up an `ICollectionFixture<T>` shared across two test classes and verify the expensive resource is only constructed once.
- Reproduce a state-leakage bug from an unisolated shared fixture, then fix it.

## Readiness Criteria

Use fixtures to share expensive setup safely, isolate per-test state within a shared fixture, and choose between class-scoped and collection-scoped fixtures appropriately.

## References

### Microsoft Learn

- [Unit testing C# with xUnit](https://learn.microsoft.com/dotnet/core/testing/unit-testing-with-dotnet-test)

### Other

- [xUnit.net: Shared context between tests](https://xunit.net/docs/shared-context)
