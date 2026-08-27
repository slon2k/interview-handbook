# Testing Asynchronous Code

## Definition

Testing `async` methods mostly works the same as testing synchronous ones — test frameworks natively support `async Task`-returning test methods — but a few specific patterns (testing cancellation, testing concurrent behavior, avoiding accidental deadlocks) need deliberate attention.

```csharp
[Fact]
public async Task GetOrderAsync_ReturnsOrder_WhenItExists()
{
    var service = new OrderService(new FakeOrderRepository());
    var order = await service.GetOrderAsync(42);
    Assert.NotNull(order);
}
```

## Alternatives & Trade-offs

Writing the test method itself as `async Task` and awaiting normally is simple and safe — it's the same cooperative model covered in Module 6, and test frameworks fully support it. The risk only appears when test code blocks synchronously on async work (`.Result`, `.Wait()`) instead of awaiting, which can reintroduce the exact deadlock risk from Module 6's `synchronization-context-and-configureawait.md`, or when testing cancellation/timing behavior without properly controlling the clock or token.

## How It Works

### Never block on async code in tests, either

```csharp
// WRONG: blocking with .Result inside a test can deadlock in certain test-runner contexts,
// and always loses the specific exception type (wrapped in AggregateException) if the call fails
var order = service.GetOrderAsync(42).Result;

// RIGHT: the test method itself is async, and awaits normally
[Fact]
public async Task GetOrderAsync_ReturnsOrder()
{
    var order = await service.GetOrderAsync(42);
}
```

### Testing that cancellation is honored

```csharp
[Fact]
public async Task ProcessAsync_StopsWhenCancelled()
{
    using var cts = new CancellationTokenSource();
    cts.Cancel(); // cancel immediately, before the operation starts

    await Assert.ThrowsAsync<OperationCanceledException>(
        () => service.ProcessAsync(cts.Token));
}
```

### Testing a timeout without actually waiting for it

```csharp
// Slow: actually waits out the real timeout duration
[Fact]
public async Task ProcessAsync_TimesOutAfterConfiguredDuration()
{
    var service = new SlowService(timeout: TimeSpan.FromSeconds(30));
    await Assert.ThrowsAsync<TimeoutException>(() => service.ProcessAsync());
    // this test takes 30 real seconds to run
}

// Fast: inject a fake/abstracted time source so the test can simulate elapsed time instantly
[Fact]
public async Task ProcessAsync_TimesOutAfterConfiguredDuration()
{
    var fakeClock = new FakeClock();
    var service = new SlowService(fakeClock, timeout: TimeSpan.FromSeconds(30));
    var task = service.ProcessAsync();
    fakeClock.Advance(TimeSpan.FromSeconds(31)); // simulate time passing instantly
    await Assert.ThrowsAsync<TimeoutException>(() => task);
}
```

### Testing exceptions from an async method

```csharp
[Fact]
public async Task Withdraw_WhenInsufficientFunds_ThrowsAsync()
{
    var account = new BankAccount(balance: 50m);
    await Assert.ThrowsAsync<InvalidOperationException>(() => account.WithdrawAsync(100m));
}
```

## Application

Write test methods as `async Task` and await dependencies normally, the same discipline as production code from Module 6. Test cancellation explicitly by pre-cancelling a token and asserting the expected exception. Abstract time behind a controllable interface for any test that would otherwise need to wait out a real timeout or delay.

## Common Mistakes

- Blocking on async code with `.Result`/`.Wait()` inside test methods instead of making the test method itself `async Task`.
- Writing a test that actually waits out a real delay/timeout, making the test suite slower than necessary for no added confidence.
- Not testing cancellation behavior at all for methods that accept a `CancellationToken`, missing a whole category of correctness bugs.
- Using `async void` for a test method (unsupported/silently ignored by most test frameworks) instead of `async Task`.

## Common Interview Questions

### Basic
- How do you write a test for an `async Task`-returning method?
- Why shouldn't you block on async code with `.Result` inside a test?

### Intermediate
- How would you test that a method correctly honors a `CancellationToken`?
- How would you test timeout behavior without making the test slow?

### Advanced
- How would you design a controllable, injectable time abstraction to make timeout/delay-dependent code fast to test?
- What test-framework-specific pitfall exists with `async void` test methods?

### Follow-up Questions
- Does `Assert.ThrowsAsync` differ from `Assert.Throws` for testing exceptions from async methods?
- Can a test method safely run multiple awaited operations concurrently using `Task.WhenAll`?

### Code Prediction
A test calls `service.GetOrderAsync(42).Result` instead of awaiting it, inside a test framework context that happens to have a synchronization context. What's the risk, referencing the deadlock pattern from Module 6?

## Practical Tasks

- Write a test verifying that a method correctly throws `OperationCanceledException` when its token is pre-cancelled.
- Refactor a slow, real-timeout-waiting test into a fast one using an injected, controllable clock abstraction.
- Identify and fix a test blocking on async code with `.Result` instead of awaiting properly.

## Readiness Criteria

Write async test methods correctly using `await`, test cancellation and timeout behavior deliberately and quickly, and avoid the blocking-on-async-code deadlock risk inside tests.

## References

### Microsoft Learn

- [Unit testing C# with xUnit](https://learn.microsoft.com/dotnet/core/testing/unit-testing-with-dotnet-test)
