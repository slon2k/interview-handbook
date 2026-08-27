# What Should and Should Not Be Mocked

## Definition

The general rule: mock (or fake/stub) dependencies that cross a real boundary — I/O, external services, time, randomness — and avoid mocking types you own that are cheap, deterministic, and simple to construct directly, since mocking those adds indirection without solving any real problem.

```csharp
// Worth mocking: crosses a real boundary (network call to an external payment provider)
var mockGateway = new Mock<IPaymentGateway>();

// NOT worth mocking: a plain value object with no external dependency
var money = new Money(100m, "USD"); // just construct it directly
```

## Alternatives & Trade-offs

Mocking everything a class depends on maximizes test isolation (a failure can only be caused by the unit under test itself) but can produce brittle, hard-to-read tests full of setup for things that never needed to vary. Using real objects wherever they're cheap and deterministic keeps tests simpler and closer to real usage, reserving mocking specifically for the handful of dependencies that genuinely can't or shouldn't run for real in a unit test.

## How It Works

### Good candidates for mocking — real boundaries

```csharp
IPaymentGateway        // external network call
IEmailSender            // external network call, real side effect (don't actually send emails in tests)
IClock / ISystemClock   // wall-clock time — needs to be controllable for deterministic tests (see reliable-tests.md)
IRandomNumberGenerator  // randomness — needs to be controllable for deterministic tests
IFileSystem             // real I/O
```

### Poor candidates for mocking — cheap, deterministic, owned types

```csharp
// Don't mock this — just construct a real one; it's a plain value object with no I/O and no side effects
var address = new Address("123 Test St", "Testville");

// Don't mock this — it's pure, deterministic logic; test it directly with real inputs
var calculator = new TaxCalculator();
var tax = calculator.Calculate(100m); // no need to fake or mock anything here
```

Mocking a pure, deterministic calculation adds ceremony without removing any actual nondeterminism or side effect — there's nothing unpredictable to isolate against.

### A more nuanced case — repository interfaces

```csharp
IOrderRepository // often faked (an in-memory implementation) rather than mocked with per-call setups,
                   // specifically because a fake behaves more realistically across a sequence of operations
```

Repository-style interfaces sit in a middle zone: not a "cheap owned type" you'd just construct directly, but also often better served by a fake than a heavily mock-verified stub, as discussed in `mocks-stubs-and-fakes.md`.

### Mocking too much makes a test brittle without adding real confidence

```csharp
// If EVERY collaborator of OrderService is mocked, including trivial value types,
// this test verifies almost nothing except that mocks were wired up correctly
var mockValidator = new Mock<IOrderValidator>();
var mockFormatter = new Mock<IOrderFormatter>();
var mockAddress = new Mock<IAddress>(); // mocking an interface for what should just be a plain value object
```

## Application

Mock dependencies that cross I/O boundaries, involve real side effects, or introduce nondeterminism (time, randomness). Construct real instances directly for cheap, deterministic, owned types with no external dependency. Use judgment for the middle ground (repositories, application services) based on whether a fake or a mock better serves the specific test's purpose.

## Common Mistakes

- Mocking every single dependency of a class by habit, including plain value objects and pure calculation logic that could just be constructed and used directly.
- Not mocking something that genuinely needs to be controlled for determinism (system clock, random number generator), leading to flaky tests (see `reliable-and-deterministic-tests.md`).
- Treating "mock everything" as a proxy for "good isolation," when over-mocking can produce a test that verifies almost nothing meaningful about actual behavior.
- Confusing "this type is an interface" with "this type should be mocked" — plenty of interfaces represent cheap, deterministic behavior that's fine to use for real in a test.

## Common Interview Questions

### Basic
- What kinds of dependencies are good candidates for mocking?
- Why shouldn't a plain value object typically be mocked?

### Intermediate
- Why does system time (`DateTime.Now`) usually need to be abstracted and mocked/faked for reliable tests?
- What's the risk of mocking every dependency a class has, even trivial ones?

### Advanced
- How would you decide, for a repository-style interface, whether a fake or a per-test mock setup better serves a specific test's purpose?
- How does over-mocking reduce a test's actual value, even if it technically passes?

### Follow-up Questions
- Should randomness always be abstracted behind an interface for testability?
- Is it ever appropriate to mock a pure, deterministic calculation just to isolate a test further?

### Code Prediction
A test for a tax calculation service mocks `ITaxRateProvider` (a real external boundary, reasonable to mock) but also mocks `IMoneyFormatter`, which just formats a decimal as a string with no side effects or nondeterminism. If `IMoneyFormatter`'s real implementation has a bug, would this over-mocked test ever catch it?

## Practical Tasks

- Identify a test that mocks a plain, deterministic value type unnecessarily, and refactor it to construct a real instance instead.
- Introduce an `IClock` abstraction for a piece of logic depending on `DateTime.Now`, and mock/fake it for deterministic testing.
- Review a test class's mock setups and classify each as a reasonable real-boundary mock or an unnecessary over-mock.

## Readiness Criteria

Distinguish dependencies worth mocking (I/O, external services, time, randomness) from those better used directly (cheap, deterministic, owned types), and recognize when over-mocking reduces a test's real value.

## References

### Other

- [Martin Fowler: Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
