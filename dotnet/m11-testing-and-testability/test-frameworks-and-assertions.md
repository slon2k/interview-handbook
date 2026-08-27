# Test Frameworks and Assertions (xUnit / NUnit)

## Definition

**xUnit** and **NUnit** are the two most common .NET test frameworks, providing test discovery, execution, and assertion libraries. xUnit is the more common default in modern .NET projects; NUnit is older and still widely used, especially in codebases predating xUnit's rise. Both provide `Assert`-style methods for verifying expected outcomes.

```csharp
// xUnit
public class OrderTests
{
    [Fact]
    public void CalculateTotal_ReturnsExpectedAmount()
    {
        Assert.Equal(90m, order.CalculateTotal());
    }
}

// NUnit
[TestFixture]
public class OrderTests
{
    [Test]
    public void CalculateTotal_ReturnsExpectedAmount()
    {
        Assert.That(order.CalculateTotal(), Is.EqualTo(90m));
    }
}
```

## Alternatives & Trade-offs

xUnit creates a new test class instance per test method by default, encouraging test isolation without shared mutable state by accident. NUnit's `Assert.That(actual, Is.EqualTo(expected))` constraint-based syntax reads more fluently for some assertion types and offers richer built-in constraints. Neither is "better" outright — most teams pick one based on project convention, ecosystem tooling fit, or historical codebase choice, and consistency within a codebase matters more than which one is chosen.

## How It Works

### xUnit — new instance per test, `[Fact]`/`[Theory]`

```csharp
public class OrderTests
{
    private readonly Order _order = new(subtotal: 100m); // fresh instance for every single test method

    [Fact]
    public void CalculateTotal_ReturnsExpectedAmount() => Assert.Equal(90m, _order.CalculateTotal());
}
```

A fresh class instance per test means fields like `_order` can't accidentally leak state between tests — each test starts from the same clean constructor-run state, without needing an explicit `[SetUp]` method the way NUnit does.

### NUnit — shared instance per fixture by default, explicit `[SetUp]`

```csharp
[TestFixture]
public class OrderTests
{
    private Order _order = null!;

    [SetUp]
    public void SetUp() => _order = new Order(subtotal: 100m); // must be explicit, or state could leak between tests

    [Test]
    public void CalculateTotal_ReturnsExpectedAmount() => Assert.That(_order.CalculateTotal(), Is.EqualTo(90m));
}
```

### Assertion styles

```csharp
// xUnit
Assert.Equal(expected, actual);
Assert.True(condition);
Assert.Throws<InvalidOperationException>(() => account.Withdraw(1000m));

// NUnit constraint-based
Assert.That(actual, Is.EqualTo(expected));
Assert.That(condition, Is.True);
Assert.Throws<InvalidOperationException>(() => account.Withdraw(1000m));

// FluentAssertions (works with either framework, popular add-on library)
actual.Should().Be(expected);
account.Invoking(a => a.Withdraw(1000m)).Should().Throw<InvalidOperationException>();
```

FluentAssertions' more English-like syntax and richer failure messages (showing exactly what was expected vs. actual, for complex objects) is why many teams layer it on top of either xUnit or NUnit rather than relying only on the base framework's assertions.

## Application

Use whichever framework the codebase already standardizes on; introducing a second framework into an existing codebase adds tooling and mental-context overhead without much real benefit. Be deliberate about `[SetUp]`-style shared state in NUnit specifically, since it doesn't get xUnit's automatic per-test isolation for free.

## Common Mistakes

- Assuming NUnit's `[SetUp]` behaves like xUnit's constructor-per-test isolation, when it actually requires explicit setup logic to avoid shared mutable state leaking between tests within the same fixture.
- Mixing xUnit and NUnit in the same codebase without a strong reason, adding unnecessary tooling and mental overhead.
- Relying purely on base-framework assertions for complex object comparisons, producing unhelpful failure messages instead of using something like FluentAssertions for richer diagnostics.
- Assuming one framework is objectively superior rather than recognizing the choice is largely about convention and ecosystem fit.

## Common Interview Questions

### Basic
- What's the difference between xUnit and NUnit at a basic usage level?
- What is FluentAssertions, and why would a team add it on top of xUnit or NUnit?

### Intermediate
- How does xUnit's per-test instantiation model differ from NUnit's shared-fixture model, and what does that imply for state isolation?
- What's a `[Theory]` in xUnit, and how does it relate to parameterized tests (covered separately)?

### Advanced
- What test-isolation bug could occur in an NUnit fixture with mutable shared state and no explicit `[SetUp]`, that xUnit's model would prevent by default?
- How would you decide whether introducing FluentAssertions to an existing test suite is worth the added dependency?

### Follow-up Questions
- Does xUnit support fixture-level setup shared across multiple test classes?
- Can NUnit and xUnit tests coexist in the same test project?

### Code Prediction
An NUnit test fixture has a mutable `private List<int> _items = new();` field with no `[SetUp]` resetting it between tests, and one test adds items to it. What happens to a second test in the same fixture that assumes `_items` starts empty?

## Practical Tasks

- Write the same set of tests using both xUnit and NUnit syntax and compare readability and setup requirements.
- Reproduce a state-leakage bug in an NUnit fixture missing explicit `[SetUp]`, then fix it.
- Add FluentAssertions to an existing test file and compare the failure message quality for a complex object comparison.

## Readiness Criteria

Write tests fluently in at least one framework, understand the isolation-model difference between xUnit and NUnit, and know when a richer assertion library is worth adding.

## References

### Microsoft Learn

- [Unit testing C# with xUnit](https://learn.microsoft.com/dotnet/core/testing/unit-testing-with-dotnet-test)

### Other

- [xUnit.net documentation](https://xunit.net/)
- [NUnit documentation](https://docs.nunit.org/)
- [FluentAssertions documentation](https://fluentassertions.com/)
