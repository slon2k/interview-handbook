# Mocks, Stubs, and Fakes

## Definition

These are all "test doubles" — substitutes for a real dependency, differing in intent: a **stub** provides canned answers to calls, with no behavior of its own. A **fake** is a working, simplified implementation (an in-memory repository standing in for a database-backed one). A **mock** additionally lets you verify *how* it was called — which methods, with what arguments, how many times. Moq and NSubstitute are the two most common .NET mocking libraries.

```csharp
// Stub: returns a fixed value, no verification
var stubRepository = new Mock<IOrderRepository>();
stubRepository.Setup(r => r.GetByIdAsync(42)).ReturnsAsync(new Order { Id = 42 });

// Mock: same setup, but also verifies a specific call happened
mockNotifier.Verify(n => n.NotifyAsync("customer@example.com"), Times.Once);

// Fake: a real, simplified working implementation
public class FakeOrderRepository : IOrderRepository
{
    private readonly List<Order> _orders = new();
    public Task<Order?> GetByIdAsync(int id) => Task.FromResult(_orders.FirstOrDefault(o => o.Id == id));
}
```

## Alternatives & Trade-offs

A fake behaves like a real (if simplified) implementation — often the most realistic and least brittle substitute, but more work to write than a one-line mock setup. A mock/stub via Moq or NSubstitute is quick to set up for a single test's specific scenario, but many small per-test setups can become repetitive, and mocks specifically risk coupling a test to *how* a unit calls its dependencies rather than *what* it achieves (see `testing-behavior-not-implementation.md`).

## How It Works

### Moq syntax

```csharp
var mockRepository = new Mock<IOrderRepository>();
mockRepository.Setup(r => r.GetByIdAsync(42)).ReturnsAsync(new Order { Id = 42, Total = 100m });
mockRepository.Setup(r => r.GetByIdAsync(It.Is<int>(id => id != 42))).ReturnsAsync((Order?)null);

var service = new OrderService(mockRepository.Object);
var result = await service.GetOrderTotalAsync(42);

mockRepository.Verify(r => r.GetByIdAsync(42), Times.Once); // verifies the call actually happened
```

### NSubstitute syntax — often considered more concise for simple cases

```csharp
var repository = Substitute.For<IOrderRepository>();
repository.GetByIdAsync(42).Returns(new Order { Id = 42, Total = 100m });

var service = new OrderService(repository);
var result = await service.GetOrderTotalAsync(42);

await repository.Received(1).GetByIdAsync(42);
```

### A fake in action — used across many tests without per-test setup

```csharp
public class FakeOrderRepository : IOrderRepository
{
    private readonly List<Order> _orders = new();
    public Task AddAsync(Order order) { _orders.Add(order); return Task.CompletedTask; }
    public Task<Order?> GetByIdAsync(int id) => Task.FromResult(_orders.FirstOrDefault(o => o.Id == id));
}

[Fact]
public async Task PlaceOrder_ThenRetrieve_ReturnsTheSameOrder()
{
    var repository = new FakeOrderRepository();
    var service = new OrderService(repository);
    await service.PlaceOrderAsync(new Order { Id = 1 });
    var retrieved = await service.GetOrderAsync(1); // works naturally, no per-test mock setup needed
}
```

Because a fake actually maintains state and behaves consistently, it can support a whole sequence of realistic operations across one test (add then retrieve) more naturally than stitching together several mock setups.

## Application

Use fakes for dependencies that are simple enough to reimplement in-memory and used consistently across many tests (a repository interface). Use mocks/stubs for dependencies that are hard to fake realistically, or when a specific test genuinely needs to verify a particular interaction occurred (like confirming a notification was sent). Prefer outcome-based assertions over mock verification where a real outcome is available to check instead.

## Common Mistakes

- Using a mock where a fake would be simpler and more realistic, especially for dependencies reused across many tests with the same basic behavior.
- Over-verifying mock interactions instead of checking actual observable outcomes, coupling tests to implementation details (see `testing-behavior-not-implementation.md`).
- Confusing the terms stub/fake/mock in an interview or design discussion — the distinction (canned answers vs. working implementation vs. call-verification) is a common thing interviewers probe precisely.
- Writing an overly elaborate fake that duplicates significant logic from the real implementation, itself becoming a maintenance burden and a source of bugs.

## Common Interview Questions

### Basic
- What's the difference between a stub, a fake, and a mock?
- What do libraries like Moq and NSubstitute provide?

### Intermediate
- When would you prefer a fake over a mock for a repository dependency?
- Why might over-verifying mock interactions couple a test to implementation details?

### Advanced
- How would you decide whether a dependency is a good candidate for a hand-written fake versus a per-test mock setup?
- What risk does an overly elaborate fake introduce, and how would you keep a fake's complexity in check?

### Follow-up Questions
- Can a mock also act as a stub (returning canned values) in the same test?
- Is verifying a call happened always the wrong choice — are there legitimate cases for it?

### Code Prediction
A test mocks `INotifier.NotifyAsync` and verifies it was called exactly once with a specific email address. If the underlying `OrderService` is refactored to batch notifications differently but still ultimately sends the same email once, would this test likely still pass? What does that reveal about the risk of mock-verification-heavy tests?

## Practical Tasks

- Write the same test twice — once using a Moq-based mock, once using a hand-written fake — and compare readability and resilience to refactoring.
- Identify a test with excessive mock verification and refactor it to assert on outcomes instead where possible.
- Build a fake implementation of a repository interface and use it across several related tests without per-test mock setup.

## Readiness Criteria

Distinguish stubs, fakes, and mocks precisely, choose the right kind of test double for a given dependency, and avoid over-coupling tests to implementation via excessive mock verification.

## References

### Other

- [Moq documentation](https://github.com/devlooped/moq)
- [NSubstitute documentation](https://nsubstitute.github.io/)
- [Martin Fowler: Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
