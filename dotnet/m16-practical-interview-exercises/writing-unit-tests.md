# Writing Unit Tests

## What This Assesses

Given a piece of untested logic, can you design a meaningful, well-structured test suite for it — covering edge cases, not just the happy path — following Module 11's AAA structure, naming, and behavior-focused testing principles, live and under time pressure.

## Format and Time Expectations

Usually a small class or method (10-20 lines) with a prompt to "write tests for this" — expect to be asked "what else would you test?" after your first pass, so don't stop at just the obvious case.

## Exercise 1: A Discount Calculation

**Problem:** Write tests for this method.

```csharp
public decimal CalculateTotal(decimal subtotal, decimal discountRate)
{
    if (discountRate < 0 || discountRate > 1) throw new ArgumentOutOfRangeException(nameof(discountRate));
    return subtotal * (1 - discountRate);
}
```

**What a strong answer demonstrates:** A parameterized test (Module 11) covering: a normal discount case, a zero discount (no change), a 100% discount (total becomes zero), and — critically — the exception path for an out-of-range `discountRate` using `Assert.Throws<T>` (Module 11's testing-exceptions content), not just happy-path values.

**Common mistakes:** Only testing one "normal" case and never exercising the boundary values (0, 1) or the invalid-input exception path, missing exactly the edge cases the method's own guard clause exists to protect.

## Exercise 2: A Method with a Hidden Dependency

**Problem:** Write tests for this method.

```csharp
public class OrderService
{
    private readonly IOrderRepository _repository;
    public OrderService(IOrderRepository repository) => _repository = repository;

    public async Task<bool> CancelOrderAsync(int orderId)
    {
        var order = await _repository.GetByIdAsync(orderId);
        if (order is null || order.Status == "Shipped") return false;
        order.Status = "Cancelled";
        await _repository.SaveAsync(order);
        return true;
    }
}
```

**What a strong answer demonstrates:** Using a fake or mock `IOrderRepository` (Module 11) rather than a real database, and testing at least three distinct paths: order not found (returns `false`), order already shipped (returns `false`, and — a good candidate to explicitly verify — `SaveAsync` is *not* called), and a normal cancellable order (returns `true`, status updated, `SaveAsync` called).

**Common mistakes:** Only testing the successful cancellation path, missing the two early-return branches entirely — a classic branch-coverage gap (Module 11's code-coverage content) that a real interviewer will specifically probe by asking "what about an already-shipped order?"

## Exercise 3: Testing an Async Method with Cancellation

**Problem:** Write a test verifying this method respects cancellation.

```csharp
public async Task ProcessItemsAsync(IEnumerable<int> items, CancellationToken cancellationToken)
{
    foreach (var item in items)
    {
        cancellationToken.ThrowIfCancellationRequested();
        await ProcessAsync(item, cancellationToken);
    }
}
```

**What a strong answer demonstrates:** Pre-cancelling a `CancellationTokenSource` before calling the method and asserting `OperationCanceledException` is thrown (Module 6's testing-async-code content, applied), rather than only testing the successful, non-cancelled path.

**Common mistakes:** Never testing the cancellation path at all, treating cancellation support as something that doesn't need its own explicit test.

## Readiness Criteria

Design test suites covering edge cases and error paths, not just the happy path, use fakes/mocks appropriately for dependencies, and proactively identify untested branches when asked "what else would you test?" rather than needing to be walked through each one.

## References

- [Arrange-Act-Assert and test naming (Module 11)](../m11-testing-and-testability/arrange-act-assert-and-test-naming.md)
- [Parameterized tests (Module 11)](../m11-testing-and-testability/parameterized-tests.md)
- [Mocks, stubs, and fakes (Module 11)](../m11-testing-and-testability/mocks-stubs-and-fakes.md)
- [Testing asynchronous code (Module 11)](../m11-testing-and-testability/testing-async-code.md)
