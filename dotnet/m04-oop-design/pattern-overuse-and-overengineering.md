# Pattern Overuse and Overengineering

## Definition

Overengineering is adding structure, abstraction, or flexibility beyond what the current requirements justify — often by applying a design pattern because it's known, not because the problem calls for it. The result compiles and "looks professional" while making the code harder to read and change than a simpler version would.

```csharp
// A single, permanent tax rule wrapped in unnecessary Strategy + Factory ceremony
public interface ITaxStrategy { decimal Calculate(decimal amount); }
public sealed class StandardTaxStrategy : ITaxStrategy { public decimal Calculate(decimal amount) => amount * 0.2m; }
public static class TaxStrategyFactory { public static ITaxStrategy Create() => new StandardTaxStrategy(); }

// What the actual requirement needs
public decimal CalculateTax(decimal amount) => amount * 0.2m;
```

## Alternatives & Trade-offs

Patterns solve real recurring problems, but each one adds indirection that must be understood by every future reader. The trade-off is between flexibility for change that may never come and simplicity for the change that's actually needed today. YAGNI is the direct counterweight: prefer the simplest design that satisfies current, known requirements, and introduce a pattern when a second real variation actually appears.

## How It Works

### Signs of overengineering

- An interface with exactly one implementation, introduced "for testability" but never actually used in a test.
- A factory or builder for an object with two constructor parameters and no validation logic.
- A generic, configurable plugin system built for a business rule that has never changed in the system's history.
- A layer of DTOs mapping to other DTOs mapping to other DTOs, with no behavioral difference between them.

### A pattern earning its keep vs. not

```csharp
// Earns its keep: two real implementations exist today, and DI wiring lets tests substitute a fake
public interface IPaymentGateway { Task<bool> ChargeAsync(decimal amount); }
public sealed class StripeGateway : IPaymentGateway { /* ... */ }
public sealed class FakePaymentGateway : IPaymentGateway { /* used in tests */ }

// Doesn't earn its keep: no second implementation exists, no test uses the abstraction,
// and the interface simply forwards to one class
public interface ITaxCalculator { decimal Calculate(decimal amount); }
public sealed class TaxCalculator : ITaxCalculator { public decimal Calculate(decimal amount) => amount * 0.2m; }
```

The `IPaymentGateway` abstraction is justified by an actual second implementation and actual test usage. `ITaxCalculator` provides neither — it's pure ceremony until a second tax rule or a genuine test-substitution need appears.

## Application

Before introducing a pattern, ask: does a second real variation exist or is one concretely planned? Is there an actual test that needs to substitute an implementation? If the honest answer to both is no, prefer the plain, direct implementation and revisit once real variation appears.

## Common Mistakes

- Reaching for a pattern because it was recently learned, rather than because the problem exhibits the variation the pattern addresses.
- Treating "might need it later" as equivalent to "need it now" (this is exactly what YAGNI warns against).
- Justifying an abstraction with "testability" without ever writing the test that would need it.
- Confusing pattern *vocabulary* (being able to name Strategy, Factory, Decorator) with pattern *judgment* (knowing when not to use them) — interviewers often probe the latter specifically.

## Common Interview Questions

### Basic
- What is overengineering?
- Give an example of a pattern applied where it wasn't needed.

### Intermediate
- How do you decide whether an abstraction is justified or premature?
- What's the cost of an interface with a single implementation and no test using it?

### Advanced
- How would you simplify an overengineered plugin-style system back down to what the business actually needs, without losing the ability to extend it later if a real need arises?
- How do you push back, in a design discussion, on a proposed abstraction that isn't yet justified by real requirements?

### Follow-up Questions
- Is it ever acceptable to add an abstraction purely for anticipated future flexibility?
- How does pattern overuse relate to YAGNI and to the Open/Closed Principle?

### Code Prediction
Given the `ITaxCalculator`/`TaxCalculator` example above, what does removing the interface entirely change about the code's actual behavior or test coverage today? What would justify re-introducing it later?

## Practical Tasks

- Review a small codebase (or the OOP-design module itself) for interfaces with exactly one implementation and no test usage, and argue for or against simplifying each.
- Simplify an overengineered factory+strategy combination for a rule that has never varied.
- In a mock design discussion, argue against a proposed abstraction that isn't yet justified, and state what evidence would change your mind.

## Readiness Criteria

Recognize overengineering by pattern (single-implementation interfaces, unused flexibility, layered indirection with no behavioral difference), and articulate the difference between anticipatory design that's earned and design that's premature.

## References

### Microsoft Learn

- [Common architectural principles](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/architectural-principles)
