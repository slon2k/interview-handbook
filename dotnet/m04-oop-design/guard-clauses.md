# Guard Clauses

## Definition

A guard clause checks an invalid precondition early and exits or throws, keeping the main path less nested.

```csharp
ArgumentNullException.ThrowIfNull(request);
if (amount < 0) throw new ArgumentOutOfRangeException(nameof(amount));
```

## Alternatives & Trade-offs

Guard clauses improve local readability. Validation frameworks or domain constructors may be better when rules are numerous, reusable, or externally reported as a group.

## How It Works

A guard establishes a precondition at the boundary of a method or constructor. It should not replace business validation that belongs in a domain workflow.

## Application

Validate nulls, ranges, permissions, state, and argument relationships before performing work.

## Common Mistakes

- Validating only at the UI boundary.
- Throwing generic exceptions.
- Hiding important business rules in scattered guards.
- Performing side effects before validation completes.

## Common Interview Questions

### Basic
- What is a guard clause?
- Why reduce nesting?

### Intermediate
- Where should argument validation occur?
- How do guards differ from business validation?

### Advanced
- How do guard clauses affect error aggregation and API contracts?
- How would you centralize repeated invariants without hiding them?
- What are the trade-offs of validation at multiple boundaries?

### Follow-up Questions
- Should guards throw or return a result?
- What exception fits an invalid range?

### Code Prediction
Which invalid input should be rejected before any side effect?

## Practical Tasks

- Refactor nested validation into clear guards.
- Design a constructor that cannot create an invalid value object.

## Readiness Criteria

Place guards at appropriate boundaries, choose meaningful failures, and distinguish preconditions from domain validation.

## References

### Microsoft Learn

- [ArgumentNullException.ThrowIfNull](https://learn.microsoft.com/dotnet/api/system.argumentnullexception.throwifnull)
- [Exceptions](https://learn.microsoft.com/dotnet/standard/exceptions/)
