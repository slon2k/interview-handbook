# Code Coverage and Its Limitations

## Definition

Code coverage measures what percentage of code was executed by a test run — line coverage, branch coverage (did each side of every `if` execute), and other variants. It's a useful signal for finding *untested* code, but a high coverage number does not mean the tests are actually good or that the code is correct.

```bash
dotnet test --collect:"XPlat Code Coverage"
```

## Alternatives & Trade-offs

Coverage tools are cheap to run and immediately highlight code with zero test exposure — genuinely useful for finding gaps. The trade-off is that coverage measures *execution*, not *verification* — a line can be executed by a test that asserts nothing meaningful about it, counting as "covered" while providing zero actual confidence. Treating a coverage percentage as a quality target risks incentivizing tests written to hit lines rather than to verify behavior.

## How It Works

### High coverage with a worthless test

```csharp
[Fact]
public void CalculateTotal_DoesNotThrow()
{
    var order = new Order(subtotal: 100m, discountRate: 0.1m);
    order.CalculateTotal(); // executes the line, counts as "covered" — but asserts NOTHING about the result
}
```

This test achieves 100% line coverage of `CalculateTotal()` while verifying absolutely nothing about its correctness — a bug that changed the discount calculation entirely would sail through this test undetected.

### Branch coverage vs. line coverage

```csharp
public decimal CalculateTotal(decimal subtotal, bool isVip)
{
    if (isVip) return subtotal * 0.8m;   // branch A
    return subtotal;                      // branch B
}
```

A test calling this only with `isVip = true` achieves high *line* coverage (most lines executed) but 0% coverage of branch B — line coverage alone can hide entire untested code paths that branch coverage would reveal.

### Using coverage correctly — as a gap-finder, not a target

```
Coverage report shows: PaymentService.RefundAsync() — 0% coverage
```

This is coverage doing its actual job well: flagging a method with zero test exposure, prompting a developer to investigate and add real tests — not to hit an arbitrary percentage target, but because a genuinely untested method was found.

### Mutation testing as a stronger signal (brief awareness)

Mutation testing tools (e.g., Stryker.NET) deliberately introduce small bugs ("mutants") into the code and check whether the test suite catches them — a stronger signal than coverage, since it actually measures whether tests would notice if the code broke, not just whether the code ran.

## Application

Use coverage reports to find genuinely untested code, not as a quality target to hit by any means. Pay attention to branch coverage, not just line coverage, for code with meaningful conditional logic. Treat 100% coverage as neither a guarantee of correctness nor a genuinely achievable or even desirable universal target — some code (trivial getters, generated code) isn't worth the effort to specifically cover.

## Common Mistakes

- Treating a coverage percentage target (e.g., "80% coverage required") as a proxy for test quality, incentivizing tests that execute code without meaningfully verifying it.
- Looking only at line coverage and missing untested branches within lines that technically executed.
- Chasing 100% coverage on trivial or generated code that provides little real risk, at the expense of time that could improve coverage or quality where it actually matters.
- Assuming high coverage means the code is well-tested, rather than checking whether the tests actually assert meaningful outcomes.

## Common Interview Questions

### Basic
- What does code coverage measure?
- Why doesn't 100% coverage guarantee correct code?

### Intermediate
- What's the difference between line coverage and branch coverage?
- Give an example of a test that achieves high coverage while verifying almost nothing.

### Advanced
- How would you use coverage reports productively — as a gap-finder rather than a target — in a real team workflow?
- What is mutation testing, and how does it address a specific weakness of plain code coverage?

### Follow-up Questions
- Should code coverage targets be enforced as a hard gate in CI?
- Is coverage a good metric for comparing test quality across two different codebases?

### Code Prediction
Given `CalculateTotal_DoesNotThrow()` above, which asserts nothing about the returned value, if a bug is introduced that makes `CalculateTotal` always return `0`, does this test's coverage number change? Does the test itself fail?

## Practical Tasks

- Run a coverage report on an existing test suite and identify a method with unexpectedly low or zero coverage.
- Identify a test achieving high coverage while asserting little, and rewrite it to actually verify meaningful behavior.
- Compare line coverage and branch coverage for a method with conditional logic, identifying an untested branch.

## Readiness Criteria

Use code coverage as a tool for finding gaps rather than a quality target, distinguish line from branch coverage, and recognize tests that achieve coverage without real verification.

## References

### Microsoft Learn

- [Use code coverage for unit testing](https://learn.microsoft.com/dotnet/core/testing/unit-testing-code-coverage)

### Other

- [Stryker.NET (mutation testing)](https://stryker-mutator.io/docs/stryker-net/introduction/)
