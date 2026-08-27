# Arrange-Act-Assert and Test Naming

## Definition

**Arrange-Act-Assert (AAA)** is a structural convention for organizing a test into three clear phases: set up the preconditions, perform the action under test, then verify the outcome. Consistent **test naming** describes what's being tested, under what condition, and what's expected — so a failing test's name alone tells you roughly what broke, without opening the test body.

```csharp
[Fact]
public void CalculateTotal_WhenDiscountApplies_ReturnsDiscountedAmount() // MethodUnderTest_Condition_ExpectedResult
{
    // Arrange
    var order = new Order(subtotal: 100m, discountRate: 0.1m);

    // Act
    var total = order.CalculateTotal();

    // Assert
    Assert.Equal(90m, total);
}
```

## Alternatives & Trade-offs

AAA's clear separation makes a test easy to scan — you always know where to look for setup versus the actual check. Some teams use "Given-When-Then" (the same three-phase idea, common in BDD-style tools) instead, which reads more like a specification in plain language. Either is fine; what matters is consistency within a codebase, not which specific label is used.

## How It Works

### AAA keeps each test's intent scannable

```csharp
[Fact]
public void Withdraw_WhenAmountExceedsBalance_ThrowsInvalidOperationException()
{
    // Arrange
    var account = new BankAccount(balance: 50m);

    // Act
    Action act = () => account.Withdraw(100m);

    // Assert
    Assert.Throws<InvalidOperationException>(act);
}
```

Mixing setup, action, and assertion together without visual separation makes a test harder to read at a glance, especially as it grows — AAA (even just as comment markers or blank-line separation) keeps the structure legible regardless of test length.

### Naming conventions — several valid patterns, consistency matters more than the specific one

```csharp
// MethodUnderTest_Condition_ExpectedResult
public void CalculateTotal_WhenDiscountApplies_ReturnsDiscountedAmount()

// Should_ExpectedResult_When_Condition
public void Should_ReturnDiscountedAmount_When_DiscountApplies()

// Given_When_Then style
public void GivenADiscountedOrder_WhenCalculatingTotal_ThenReturnsDiscountedAmount()
```

### Why a good name matters more than it seems

```csharp
// A poor name — a failing test tells you almost nothing without opening it
[Fact]
public void Test1() { }

// A good name — the CI failure list alone tells you what broke and under what condition
[Fact]
public void CalculateTotal_WhenDiscountApplies_ReturnsDiscountedAmount() { }
```

When a CI run reports 40 failing tests, scannable names let you triage at a glance which failures are related and which represent one root cause versus many.

### One assertion concept per test — not necessarily one `Assert` call

```csharp
[Fact]
public void CalculateTotal_ReturnsCorrectAmountAndAppliedDiscount()
{
    var result = order.CalculateTotalWithBreakdown();
    Assert.Equal(90m, result.Total);       // multiple asserts are fine if they check
    Assert.Equal(10m, result.DiscountApplied); // one cohesive outcome, not unrelated behaviors
}
```

## Application

Use AAA (or an equivalent Given-When-Then structure) consistently across a codebase's test suite, and adopt one naming convention consistently rather than mixing several. Favor names that describe behavior and condition over names that just restate the method being called.

## Common Mistakes

- Mixing arrange, act, and assert logic together without clear separation, making longer tests hard to scan.
- Using uninformative names (`Test1`, `OrderTest`) that give no information when a test fails in a CI report.
- Testing multiple unrelated behaviors in one test method under one vague name, making a failure ambiguous about which behavior actually broke.
- Switching naming conventions inconsistently across a codebase, making test reports harder to scan as a whole.

## Common Interview Questions

### Basic
- What does Arrange-Act-Assert mean?
- Why does test naming matter beyond just satisfying a compiler?

### Intermediate
- What's the difference between AAA and Given-When-Then, and does it matter which one a team uses?
- What makes a test name "good" versus "uninformative"?

### Advanced
- How would you refactor a poorly-named, poorly-structured legacy test suite without rewriting all the underlying test logic?
- Why might testing multiple unrelated behaviors in one test method with multiple assertions be a bad idea, even if each assertion is individually correct?

### Follow-up Questions
- Is it acceptable to have more than one `Assert` call in a single test?
- Does AAA structure apply the same way to integration tests as unit tests?

### Code Prediction
Given a test named `Test1` that fails in a CI run alongside 30 other failing tests, how much information does the test name alone provide about what to investigate, compared to a test named `CalculateTotal_WhenDiscountApplies_ReturnsDiscountedAmount`?

## Practical Tasks

- Refactor a poorly-structured, poorly-named test into clear AAA sections with a descriptive name.
- Establish and apply one consistent naming convention across a small set of existing tests using different styles.
- Identify a test asserting multiple unrelated behaviors and split it into focused, individually-named tests.

## Readiness Criteria

Structure tests clearly using AAA (or an equivalent), and name tests so that a failure is diagnosable from the test report alone.

## References

### Microsoft Learn

- [Unit testing best practices](https://learn.microsoft.com/dotnet/core/testing/unit-testing-best-practices)
