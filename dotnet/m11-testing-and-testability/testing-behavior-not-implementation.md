# Testing Behavior, Not Implementation

## Definition

A test should verify *what* a unit does (its observable behavior, from a caller's perspective) rather than *how* it does it internally. Tests that assert on private implementation details break every time the implementation is refactored, even when behavior hasn't changed — defeating one of the main reasons to have tests in the first place.

```csharp
// Tests implementation: breaks if the internal algorithm changes, even if behavior is identical
[Fact]
public void CalculateTotal_CallsApplyDiscountInternalMethod() { /* asserts a private method was called */ }

// Tests behavior: only breaks if the actual observable outcome changes
[Fact]
public void CalculateTotal_WhenDiscountApplies_ReturnsDiscountedAmount()
{
    var total = order.CalculateTotal();
    Assert.Equal(90m, total);
}
```

## Alternatives & Trade-offs

Behavior-focused tests are more resilient to refactoring — you can rewrite a method's internals freely as long as its observable contract stays the same, and the tests keep passing without modification. Implementation-focused tests (checking private state, verifying exact internal call sequences) give more precise failure localization in the moment they're written, but become a maintenance burden that actively discourages refactoring, since every internal change risks breaking tests that were never really about correctness.

## How It Works

### The refactoring test — does a test survive a safe internal change?

```csharp
// Before refactoring: CalculateTotal calls a private ApplyDiscount method
public decimal CalculateTotal() => ApplyDiscount(Subtotal);
private decimal ApplyDiscount(decimal amount) => amount * (1 - DiscountRate);

// After refactoring: inlined, same behavior, different implementation
public decimal CalculateTotal() => Subtotal * (1 - DiscountRate);
```

A test asserting the *result* of `CalculateTotal()` survives this refactor unchanged. A test that somehow asserted `ApplyDiscount` was called (via reflection, or by making it non-private just to test it) would break — for a change that introduced no actual bug.

### Testing through the public interface, not private internals

```csharp
public class OrderService
{
    public decimal CalculateTotal(Order order) => ApplyDiscount(order); // private helper is an implementation detail
    private decimal ApplyDiscount(Order order) => order.Subtotal * 0.9m;
}

[Fact]
public void CalculateTotal_AppliesDiscount()
{
    var service = new OrderService();
    var total = service.CalculateTotal(order); // only the public method is exercised
    Assert.Equal(90m, total);
}
```

### Mock verification can slip into testing implementation if overused

```csharp
// Overly implementation-focused: cares about exactly how the result was produced internally
mockRepository.Verify(r => r.GetByIdAsync(42), Times.Once); // sometimes appropriate, but easy to overdo

// Behavior-focused: cares about the actual outcome the caller experiences
var result = await service.ProcessOrderAsync(42);
Assert.Equal(OrderStatus.Processed, result.Status);
```

Verifying that a specific dependency method was called *can* be legitimate (see `mocks-stubs-and-fakes.md` and `what-to-mock.md`) but overusing verification-style assertions instead of outcome-style assertions tends to couple tests to implementation details rather than behavior.

## Application

Write tests against a unit's public contract and observable outcomes. Avoid making private members public, or reaching into internals via reflection, purely to make them "testable" — if a piece of logic genuinely deserves independent testing, consider whether it should be its own, separately-testable public unit instead.

## Common Mistakes

- Exposing private methods as `internal`/`public` purely so tests can call them directly, coupling tests to implementation and discouraging future refactoring.
- Over-verifying mock interactions (exact call counts, argument details) for behavior that would be better checked via the actual observable result.
- Treating "100% of private methods have a direct test" as a goal, when private methods are implementation details that should be exercised indirectly through the public behavior that uses them.
- Writing a test that passes today but would fail after any reasonable refactor with no behavior change — a strong signal the test is checking implementation, not behavior.

## Common Interview Questions

### Basic
- What does it mean to test behavior rather than implementation?
- Why might a test that checks private implementation details be a problem?

### Intermediate
- What is "the refactoring test" for evaluating whether a test is behavior-focused?
- Why might exposing a private method as `internal` purely for testing purposes be a bad idea?

### Advanced
- How would you decide whether a piece of logic buried in a private method deserves to be extracted into its own testable unit, versus tested only indirectly?
- How does overusing mock verification (as opposed to outcome assertions) tend to couple tests to implementation?

### Follow-up Questions
- Is it ever appropriate to verify that a mocked dependency was called a specific number of times?
- Does testing behavior rather than implementation mean private methods should never be tested at all?

### Code Prediction
A test asserts, via reflection, that a private field `_cachedDiscount` equals a specific value after calling `CalculateTotal()`. If a developer later refactors the method to compute the discount without caching it in that field at all (but returns the identical final result), does this test pass or fail? What does that reveal about what the test was actually checking?

## Practical Tasks

- Refactor a test that asserts on private implementation details into one that asserts only on public, observable behavior.
- Apply "the refactoring test" to a set of existing tests: for each, imagine a safe internal refactor and check whether the test would survive it unchanged.
- Identify overused mock-verification assertions in a test file and convert appropriate ones to outcome-based assertions instead.

## Readiness Criteria

Write tests that survive safe refactors, recognize implementation-coupled tests via "the refactoring test," and use mock verification judiciously rather than as the default assertion style.

## References

### Microsoft Learn

- [Unit testing best practices](https://learn.microsoft.com/dotnet/core/testing/unit-testing-best-practices)
