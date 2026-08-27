# ASP.NET Core Integration Testing and WebApplicationFactory

## Definition

`WebApplicationFactory<TEntryPoint>` boots a real (in-memory) instance of an ASP.NET Core application for testing — real middleware pipeline, real routing, real DI container — and provides an `HttpClient` that talks to it directly in-process, without a real network socket. This is the standard way to write integration tests that exercise the actual application, not just isolated pieces of it.

```csharp
public class OrdersApiTests : IClassFixture<WebApplicationFactory<Program>>
{
    private readonly HttpClient _client;
    public OrdersApiTests(WebApplicationFactory<Program> factory) => _client = factory.CreateClient();

    [Fact]
    public async Task GetOrder_ReturnsOk_WhenOrderExists()
    {
        var response = await _client.GetAsync("/orders/42");
        response.EnsureSuccessStatusCode();
    }
}
```

## Alternatives & Trade-offs

`WebApplicationFactory` gives high confidence — the real middleware pipeline, real routing, real DI wiring, all actually running — without the overhead of a real deployed instance and real network calls (it's in-process). Unit tests of individual components are faster and more isolated but can't catch problems in how everything is actually wired together (a missing DI registration, incorrect middleware ordering, a routing conflict) — exactly the class of bug `WebApplicationFactory`-based tests exist to catch.

## How It Works

### Overriding services for the test — swap real dependencies for fakes

```csharp
public class CustomWebApplicationFactory : WebApplicationFactory<Program>
{
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            services.RemoveAll<IOrderRepository>(); // remove the real, database-backed registration
            services.AddScoped<IOrderRepository, FakeOrderRepository>(); // substitute a fake for the test run
        });
    }
}
```

This lets the test exercise the real HTTP pipeline, real routing, real model binding and validation — while still substituting the parts (usually the database) that shouldn't be hit for real in a fast test.

### Testing the full request/response cycle, including serialization

```csharp
[Fact]
public async Task CreateOrder_ReturnsCreatedWithLocationHeader()
{
    var response = await _client.PostAsJsonAsync("/orders", new CreateOrderRequest { CustomerId = 7 });
    Assert.Equal(HttpStatusCode.Created, response.StatusCode);
    Assert.NotNull(response.Headers.Location); // verifies real middleware/serialization behavior, not just a service method's return value
}
```

This kind of test catches real bugs a unit test of `OrderService` alone never could — a missing `[FromBody]` attribute, a serialization configuration issue, a routing conflict, an incorrectly-registered middleware.

### Combining with Testcontainers for real database integration

```csharp
public class DatabaseWebApplicationFactory : WebApplicationFactory<Program>
{
    private readonly MsSqlContainer _sqlContainer;
    protected override void ConfigureWebHost(IWebHostBuilder builder)
    {
        builder.ConfigureServices(services =>
        {
            services.RemoveAll<DbContextOptions<AppDbContext>>();
            services.AddDbContext<AppDbContext>(o => o.UseSqlServer(_sqlContainer.GetConnectionString()));
        });
    }
}
```

Combining `WebApplicationFactory` with a real Testcontainers-hosted database gives the highest-fidelity integration test short of a full deployed environment — see `database-integration-tests.md`.

## Application

Use `WebApplicationFactory`-based tests to verify that the application's real wiring — middleware, routing, DI, serialization — works correctly end-to-end, substituting only the specific dependencies (usually external services or the database) that would make the test slow or unreliable. Reserve unit tests for individual business logic and use integration tests specifically for the seams that only show up when everything runs together.

## Common Mistakes

- Substituting so many services in a `WebApplicationFactory`-based test that it no longer meaningfully differs from a plain unit test, losing the actual benefit of testing real wiring.
- Not substituting the real database/external dependencies at all, making the integration test suite slow, flaky, or dependent on external systems being available.
- Forgetting that `WebApplicationFactory` tests run in-process — there's no real network involved, which matters for correctly interpreting timing or connection-related test behavior.
- Sharing one `WebApplicationFactory` instance across many test classes without appropriate fixture scoping, risking state leakage similar to the general fixture-sharing risk covered earlier in this module.

## Common Interview Questions

### Basic
- What is `WebApplicationFactory`, and what does it provide for testing?
- Why does an integration test using `WebApplicationFactory` catch bugs a unit test wouldn't?

### Intermediate
- How would you substitute a real database-backed service for a fake one within a `WebApplicationFactory`-based test?
- Does `WebApplicationFactory` make real network calls?

### Advanced
- How would you combine `WebApplicationFactory` with Testcontainers for a high-fidelity database integration test?
- How would you decide which services to substitute versus leave real, for a given integration test's purpose?

### Follow-up Questions
- Can `WebApplicationFactory` be used to test authentication/authorization behavior realistically?
- Does `WebApplicationFactory` run the application's actual `Program.cs` startup logic?

### Code Prediction
A unit test of `OrderService.CreateOrderAsync` passes, but the actual `POST /orders` endpoint returns a `500` in production due to a missing DI registration for `IOrderValidator`. Would a `WebApplicationFactory`-based integration test for the same endpoint have caught this, and why did the unit test miss it?

## Practical Tasks

- Set up a `WebApplicationFactory`-based test suite for a small API, substituting the database with a fake repository.
- Write a test verifying the full HTTP response contract (status code, headers, body shape) for an endpoint, not just its underlying service logic.
- Combine `WebApplicationFactory` with a Testcontainers-hosted database for one high-fidelity integration test.

## Readiness Criteria

Set up and use `WebApplicationFactory` for realistic integration tests, substitute dependencies appropriately without losing the test's real value, and explain what this level of test catches that unit tests cannot.

## References

### Microsoft Learn

- [Integration tests in ASP.NET Core](https://learn.microsoft.com/aspnet/core/test/integration-tests)
