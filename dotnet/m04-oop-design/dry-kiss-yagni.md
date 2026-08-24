# DRY, KISS, and YAGNI

## Definition

Three complementary design heuristics:

- **DRY** (Don't Repeat Yourself) — every piece of knowledge should have a single, unambiguous representation in the system.
- **KISS** (Keep It Simple, Stupid) — prefer the simplest design that solves the actual problem.
- **YAGNI** (You Aren't Gonna Need It) — don't build functionality for requirements that don't exist yet.

They pull in a consistent direction: avoid both duplication and speculative complexity.

## Alternatives & Trade-offs

These are heuristics, not laws, and they can conflict with each other and with SOLID. Aggressively applying DRY to superficially similar code that represents different business concepts can create the wrong abstraction, which is often costlier than a small amount of duplication. KISS and YAGNI push against over-engineering (see `pattern-overuse-and-overengineering.md`), but taken too far they can justify never refactoring toward better structure at all.

## How It Works

### DRY misapplied

```csharp
// Two validation rules that look similar today but represent different business concepts
public bool IsValidEmail(string s) => s.Contains('@') && s.Length <= 254;
public bool IsValidUsername(string s) => s.Contains('@') == false && s.Length <= 254;

// "DRY-ing" these into one shared method couples two independent business rules
public bool IsValidLength(string s, int max) => s.Length <= max; // fine to share
public bool IsValidField(string s, bool mustContainAt, int max) => // forced, artificial sharing
    (mustContainAt ? s.Contains('@') : !s.Contains('@')) && s.Length <= max;
```

The shared `IsValidLength` helper is genuine, harmless DRY. Forcing the two different business rules into one parameterized method just because they look similar creates an artificial coupling — a change to username rules risks touching email validation.

### KISS in practice

```csharp
// Over-engineered for a rule that will not change
public interface IAgeValidationStrategy { bool IsValid(int age); }
public class AdultAgeValidationStrategy : IAgeValidationStrategy
{
    public bool IsValid(int age) => age >= 18;
}

// Simple and sufficient
public bool IsAdult(int age) => age >= 18;
```

### YAGNI in practice

```csharp
// Built speculatively for a multi-currency feature that isn't planned or requested
public interface ICurrencyConverter { decimal Convert(decimal amount, string from, string to); }

// What the actual current requirement needs
public decimal Amount { get; init; } // single currency, add conversion when actually required
```

## Application

Apply DRY to genuine, single-source business rules and calculations. Apply KISS when choosing between a simple direct implementation and a more "flexible" one with no current justification. Apply YAGNI when tempted to add configuration options, abstraction layers, or extensibility hooks for hypothetical future requirements.

## Common Mistakes

- Using DRY to justify merging two rules that coincidentally look similar but represent different, independently-changing business concepts (see "wrong abstraction" above).
- Treating YAGNI as a reason to skip necessary error handling or validation that the *current* requirement actually needs.
- Using KISS to avoid any structure at all, producing a single giant method that's simple to write but hard to read or test.
- Refactoring duplicated code into a shared abstraction before a second real, differing use case actually exists to validate the abstraction's shape.

## Common Interview Questions

### Basic
- What do DRY, KISS, and YAGNI stand for?
- Give an example of unnecessary complexity that YAGNI would avoid.

### Intermediate
- What is "the wrong abstraction," and how does DRY applied carelessly create it?
- How do you decide whether two similar pieces of code should be merged or kept separate?

### Advanced
- How do DRY and YAGNI create tension with SOLID's Open/Closed Principle, which sometimes calls for anticipatory abstraction?
- How would you recognize, in code review, that a "DRY" refactor actually coupled two unrelated business rules?

### Follow-up Questions
- Is some duplication ever preferable to a shared abstraction?
- How does YAGNI interact with building a public library API versus internal application code?

### Code Prediction
Given the `IsValidField` example above, if the username rule changes to also disallow spaces, what does that force to also be reviewed in `IsValidEmail`'s call sites, and why does that reveal a DRY misapplication?

## Practical Tasks

- Identify two "duplicated" methods in a hypothetical codebase that actually represent different business rules, and argue against merging them.
- Simplify an over-engineered strategy-pattern implementation for a rule that has never varied and isn't expected to.
- Remove speculative extensibility (unused configuration options, unused interface parameters) from a small service.

## Readiness Criteria

Apply DRY without creating a wrong abstraction, recognize over-engineering that KISS/YAGNI would avoid, and articulate the trade-off between these heuristics and anticipatory design.

## References

### Microsoft Learn

- [Common architectural principles](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/architectural-principles)
