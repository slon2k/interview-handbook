# Parameterized Tests

## Definition

A parameterized test runs the same test logic against multiple sets of input/expected-output data, avoiding near-duplicate test methods that differ only in their values. xUnit provides `[Theory]` with `[InlineData]`, `[MemberData]`, or `[ClassData]`; NUnit provides `[TestCase]` and `[TestCaseSource]`.

```csharp
[Theory]
[InlineData(100, 0.1, 90)]
[InlineData(200, 0.5, 100)]
[InlineData(50, 0, 50)]
public void CalculateTotal_AppliesDiscountCorrectly(decimal subtotal, decimal discountRate, decimal expected)
{
    var order = new Order(subtotal, discountRate);
    Assert.Equal(expected, order.CalculateTotal());
}
```

## Alternatives & Trade-offs

Writing a separate test method for each input combination is more verbose and duplicates the arrange/act/assert structure for every case, but parameterized tests trade that duplication for a slightly less immediately obvious failure report (you see which *row* failed, but sometimes need to check the input values to understand which specific scenario that represents) — good test naming and clear parameter values largely offset this.

## How It Works

### `[InlineData]` — simple, compile-time-constant values

```csharp
[Theory]
[InlineData(0, true)]
[InlineData(-1, false)]
[InlineData(int.MaxValue, true)]
public void IsValid_ReturnsExpectedResult(int value, bool expected)
{
    Assert.Equal(expected, Validator.IsValid(value));
}
```

`[InlineData]` values must be compile-time constants — no complex objects, no computed values — which is exactly why `[MemberData]` exists for anything richer.

### `[MemberData]` — for complex or computed test data

```csharp
public static IEnumerable<object[]> DiscountCases()
{
    yield return new object[] { 100m, 0.1m, 90m };
    yield return new object[] { new Order(subtotal: 200m), 0.5m, 100m }; // can include real objects, unlike InlineData
}

[Theory]
[MemberData(nameof(DiscountCases))]
public void CalculateTotal_AppliesDiscountCorrectly(decimal subtotal, decimal discountRate, decimal expected)
{
    // ...
}
```

### NUnit equivalent

```csharp
[TestCase(100, 0.1, 90)]
[TestCase(200, 0.5, 100)]
public void CalculateTotal_AppliesDiscountCorrectly(decimal subtotal, decimal discountRate, decimal expected)
{
    var order = new Order(subtotal, discountRate);
    Assert.That(order.CalculateTotal(), Is.EqualTo(expected));
}
```

### Naming individual cases for a readable test report

```csharp
[Theory]
[InlineData(100, 0.1, 90)]
public void CalculateTotal_AppliesDiscountCorrectly(decimal subtotal, decimal discountRate, decimal expected)
{
    // xUnit reports each row with its parameter values in the test explorer/CI output by default,
    // e.g. "CalculateTotal_AppliesDiscountCorrectly(subtotal: 100, discountRate: 0.1, expected: 90)"
}
```

## Application

Use parameterized tests whenever the same logic needs to be verified against several input/output combinations — boundary values, edge cases (zero, negative, maximum), and representative "normal" cases. Choose `[InlineData]`/`[TestCase]` for simple literal values and `[MemberData]`/`[TestCaseSource]` once test data needs to be computed or include non-constant objects.

## Common Mistakes

- Writing many near-duplicate test methods differing only in input values, instead of consolidating them into one parameterized test.
- Trying to pass a complex object literal to `[InlineData]`, which only accepts compile-time constants, instead of switching to `[MemberData]`.
- Cramming unrelated test scenarios into one parameterized test just because they share a method signature, when they'd be clearer as separate, distinctly-named tests.
- Not choosing input values that actually exercise meaningful edge cases (boundaries, zero, negative, empty), settling for only "happy path" values.

## Common Interview Questions

### Basic
- What is a parameterized test, and what problem does it solve?
- What's the difference between `[InlineData]` and `[MemberData]` in xUnit?

### Intermediate
- Why can't `[InlineData]` accept a complex object as a parameter value?
- How does a parameterized test's failure report differ from a single test's failure report?

### Advanced
- How would you design a set of parameterized test cases that specifically target boundary conditions for a validation rule?
- When would consolidating several near-duplicate tests into one parameterized test actually reduce clarity rather than improve it?

### Follow-up Questions
- Can a parameterized test method have more than one data source attribute at once?
- Does xUnit report which specific parameter row failed when a `[Theory]` test fails?

### Code Prediction
Given the `CalculateTotal_AppliesDiscountCorrectly` theory with three `[InlineData]` rows, if the second row's expected value is wrong, does the entire theory fail as one unit, or does the test report distinguish which specific row failed?

## Practical Tasks

- Consolidate three near-duplicate test methods into one parameterized test using `[InlineData]`.
- Convert a parameterized test needing complex object parameters from `[InlineData]` (which would fail to compile) to `[MemberData]`.
- Design a parameterized test set specifically covering boundary conditions (zero, negative, maximum) for a validation method.

## Readiness Criteria

Use parameterized tests to consolidate related test cases, choose the correct data-source mechanism for the data's complexity, and design test cases that meaningfully cover edge conditions.

## References

### Microsoft Learn

- [Unit testing C# with xUnit](https://learn.microsoft.com/dotnet/core/testing/unit-testing-with-dotnet-test)

### Other

- [xUnit.net: Theory tests](https://xunit.net/docs/getting-started/netcore/cmdline#write-first-theory)
