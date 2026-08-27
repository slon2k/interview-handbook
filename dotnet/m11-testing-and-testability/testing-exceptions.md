# Testing Exceptions

## Definition

Testing that a method throws the correct exception under the correct condition — not just "an exception happens," but the specific type, and ideally that its message/properties carry the expected information. xUnit and NUnit both provide dedicated assertion methods (`Assert.Throws<T>`, `Assert.ThrowsAsync<T>`) rather than relying on a manual try/catch.

```csharp
[Fact]
public void Withdraw_WhenAmountExceedsBalance_ThrowsInvalidOperationException()
{
    var account = new BankAccount(balance: 50m);
    var exception = Assert.Throws<InvalidOperationException>(() => account.Withdraw(100m));
    Assert.Equal("Insufficient funds", exception.Message);
}
```

## Alternatives & Trade-offs

Using `Assert.Throws<T>` is explicit about which exception type is expected and fails clearly if a different exception (or none at all) is thrown. A manual try/catch that swallows any exception and asserts something afterward is more error-prone — it's easy to accidentally write a test that passes whether or not the expected exception actually occurred, because the catch block is too broad or the assertion logic has a bug.

## How It Works

### The dedicated assertion, not a manual try/catch

```csharp
// WRONG: passes even if a completely different exception type is thrown, or subtly, might pass with no exception at all
// depending on exactly how the try/catch and flag logic is written
[Fact]
public void Withdraw_ThrowsWhenInsufficientFunds_BadVersion()
{
    var threw = false;
    try { account.Withdraw(100m); }
    catch { threw = true; }
    Assert.True(threw); // doesn't verify WHICH exception type, or its message
}

// RIGHT: precise about the expected exception type, and can inspect its details
[Fact]
public void Withdraw_ThrowsWhenInsufficientFunds()
{
    var exception = Assert.Throws<InvalidOperationException>(() => account.Withdraw(100m));
    Assert.Equal("Insufficient funds", exception.Message);
}
```

### Testing async exceptions

```csharp
[Fact]
public async Task WithdrawAsync_WhenInsufficientFunds_ThrowsAsync() =>
    await Assert.ThrowsAsync<InvalidOperationException>(() => account.WithdrawAsync(100m));
```

### Verifying the exception's specific properties, not just its type

```csharp
[Fact]
public void PlaceOrder_WhenStockInsufficient_ThrowsWithCorrectProductId()
{
    var exception = Assert.Throws<InsufficientStockException>(() => service.PlaceOrder(order));
    Assert.Equal(order.ProductId, exception.ProductId); // the exception type alone might not be specific enough
}
```

Just asserting "an `InsufficientStockException` was thrown" might not distinguish between "the right product ran out" and "the exception logic itself has a bug and always reports the wrong product" — inspecting relevant properties closes that gap.

### Testing that an exception is NOT thrown for a valid case

```csharp
[Fact]
public void Withdraw_WhenSufficientFunds_DoesNotThrow()
{
    var account = new BankAccount(balance: 100m);
    account.Withdraw(50m); // if this throws, the test fails naturally — no special assertion needed
}
```

A test simply calling the method without wrapping it in a try/catch already fails if an unexpected exception is thrown — there's rarely a need for an explicit "does not throw" assertion.

## Application

Use `Assert.Throws<T>`/`Assert.ThrowsAsync<T>` rather than manual try/catch for exception tests. Inspect the exception's message or specific properties when the type alone doesn't fully verify the correct failure occurred. Rely on natural test failure (an uncaught exception fails the test) rather than an explicit "does not throw" assertion for the happy path.

## Common Mistakes

- Using a manual try/catch with a boolean flag instead of the framework's dedicated exception-assertion method, risking a test that passes for the wrong reason.
- Asserting only the exception's type without checking relevant properties, missing cases where the right exception type is thrown for the wrong underlying reason.
- Catching a broader exception type than intended in the assertion (e.g., `Exception` instead of the specific `InvalidOperationException`), letting an unrelated bug masquerade as the expected failure.
- Writing an explicit "should not throw" test with an unnecessary try/catch, when simply calling the method without one already achieves the same verification.

## Common Interview Questions

### Basic
- How do you test that a method throws a specific exception?
- What's the difference between `Assert.Throws` and `Assert.ThrowsAsync`?

### Intermediate
- Why is a manual try/catch with a boolean flag a worse pattern than `Assert.Throws<T>`?
- When would you inspect an exception's specific properties rather than just its type?

### Advanced
- How would you design a test that distinguishes "the correct exception was thrown for the correct reason" from "the correct exception type happened to be thrown by coincidence"?
- What's the risk of asserting against too broad an exception type in a test?

### Follow-up Questions
- Does `Assert.Throws<T>` also match subclasses of the specified exception type?
- Is an explicit "does not throw" test ever necessary?

### Code Prediction
Given the "bad version" try/catch test above with a boolean flag, what happens if `account.Withdraw(100m)` throws a completely unrelated `NullReferenceException` due to an unrelated bug — does the test still pass?

## Practical Tasks

- Convert a manual try/catch exception test into one using `Assert.Throws<T>`.
- Write a test verifying not just an exception's type but a specific property carrying diagnostic information.
- Identify a test asserting against an overly broad exception type and narrow it to the specific expected type.

## Readiness Criteria

Use framework-provided exception assertions correctly, verify exception details beyond just type where it matters, and avoid manual try/catch patterns that can mask incorrect behavior.

## References

### Microsoft Learn

- [Unit testing C# with xUnit](https://learn.microsoft.com/dotnet/core/testing/unit-testing-with-dotnet-test)
