# Reliable and Deterministic Tests

## Definition

A reliable (non-flaky) test produces the same result every time it runs against unchanged code — it should never intermittently pass or fail for reasons unrelated to an actual bug. Common sources of flakiness: real wall-clock time, uncontrolled randomness, test-order dependencies, shared mutable state, and race conditions in the test itself (especially around async code).

```csharp
// Flaky: depends on real system time, which changes between runs
Assert.Equal(DateTime.Now.Date, order.CreatedAt.Date); // could fail right at midnight, or under system clock skew

// Reliable: time is injected and controlled
Assert.Equal(_fakeClock.Now.Date, order.CreatedAt.Date);
```

## Alternatives & Trade-offs

Testing against the real system clock, real randomness, or real timing is sometimes unavoidable in true end-to-end scenarios, but for the vast majority of tests, injecting and controlling these sources of nondeterminism costs a small amount of extra abstraction (an `IClock`, a seeded random generator) in exchange for a test suite that never produces confusing, unrelated failures — which is what actually erodes a team's trust in its own tests over time.

## How It Works

### Abstracting time for determinism

```csharp
public interface IClock { DateTime UtcNow { get; } }
public class SystemClock : IClock { public DateTime UtcNow => DateTime.UtcNow; }
public class FakeClock : IClock { public DateTime UtcNow { get; set; } = new(2026, 1, 1); }

public class OrderService
{
    private readonly IClock _clock;
    public Order CreateOrder() => new Order { CreatedAt = _clock.UtcNow }; // deterministic and controllable in tests
}
```

### Seeding randomness for reproducibility

```csharp
// Flaky-ish: different random values on every run make some scenarios hard to reliably assert against
var random = new Random();

// Reproducible: a fixed seed produces the exact same sequence every time, useful for tests needing "random-like" input
var random = new Random(seed: 42);
```

### Test-order independence

```csharp
// Flaky if tests share static/shared mutable state and their execution order changes
private static int _sharedCounter = 0;

[Fact]
public void Test1() { _sharedCounter++; Assert.Equal(1, _sharedCounter); } // passes only if run first

[Fact]
public void Test2() { _sharedCounter++; Assert.Equal(1, _sharedCounter); } // fails if Test1 already ran
```

Tests should never depend on a specific execution order — each test should set up everything it needs and never rely on state left behind by another test (this connects directly to the fixture-isolation discussion earlier in this module).

### Async race conditions inside tests themselves

```csharp
// Flaky: assumes a background operation completes within an arbitrary delay
await Task.Delay(100);
Assert.True(backgroundTask.IsCompleted); // may or may not have finished in exactly 100ms, depending on system load

// Reliable: actually await the completion signal instead of guessing at timing
await backgroundOperation.CompletionTask;
Assert.True(backgroundOperation.CompletionTask.IsCompletedSuccessfully);
```

## Application

Abstract and inject any source of nondeterminism (time, randomness, background-task completion) behind a controllable interface for unit and most integration tests. Ensure tests never depend on execution order or shared mutable state between them. Prefer explicitly awaiting a real completion signal over guessing with an arbitrary `Task.Delay`.

## Common Mistakes

- Asserting against real wall-clock time without any tolerance or abstraction, causing intermittent failures around midnight, DST changes, or under system clock skew.
- Using unseeded randomness in a test that needs reproducible behavior across runs.
- Writing tests that only pass when run in a specific order, due to shared mutable static state.
- Using `Task.Delay` as a substitute for actually awaiting a real completion signal, producing tests that are flaky under system load or CI environment slowness.

## Common Interview Questions

### Basic
- What makes a test "flaky," and why is that a problem beyond just an occasional inconvenience?
- Name a few common sources of test flakiness.

### Intermediate
- How would you make code depending on `DateTime.Now` reliably testable?
- Why is `Task.Delay` combined with an assumption about completion a common source of flaky async tests?

### Advanced
- How would you diagnose an intermittently failing test in a large CI suite, given that it can't be reliably reproduced locally?
- How does test-order dependency via shared static state cause flakiness, and how would you detect it systematically?

### Follow-up Questions
- Does seeding a random number generator guarantee identical behavior across different .NET versions?
- Should flaky tests be quarantined/skipped, or fixed immediately?

### Code Prediction
Given the shared `_sharedCounter` example above, if `Test2` happens to run before `Test1` in a given test run, does it pass or fail? What does that reveal about a fundamental requirement for reliable tests?

## Practical Tasks

- Refactor code depending directly on `DateTime.Now` to use an injectable `IClock`, then write a deterministic test for time-dependent behavior.
- Identify and fix a test with a `Task.Delay`-based assumption about async completion timing.
- Identify test-order dependency caused by shared static state and refactor to eliminate it.

## Readiness Criteria

Identify and eliminate common sources of test flakiness (time, randomness, order dependency, timing assumptions), and design tests that are deterministic regardless of when or in what order they run.

## References

### Microsoft Learn

- [Unit testing best practices](https://learn.microsoft.com/dotnet/core/testing/unit-testing-best-practices)
