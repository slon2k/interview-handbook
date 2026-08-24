# Strategy Pattern

## Definition

Strategy encapsulates interchangeable algorithms behind a common contract.

```csharp
public interface IDiscount { decimal Apply(decimal total); }
```

## Alternatives & Trade-offs

Use a conditional for a small, stable set of cases. Use Strategy when behavior varies independently, grows, or must be selected and tested separately.

## How It Works

A context receives a strategy and delegates the variable operation. Strategies can be selected by configuration, input, or a composition root.

## Application

Pricing rules, serialization policies, retry policies, and validation algorithms are common examples.

## Common Mistakes

- Creating one class per trivial branch.
- Letting strategies depend on the context's internals.
- Selecting strategies through a hidden service locator.

## Common Interview Questions

### Basic
- What problem does Strategy solve?
- What is the context?

### Intermediate
- When is a switch clearer than Strategy?
- How does Strategy improve testing?

### Advanced
- How do you version and select strategies safely?
- How can strategy interfaces become too broad?
- How does Strategy differ from State?

### Follow-up Questions
- Can strategies be immutable?
- Where should strategy selection live?

### Code Prediction
Which strategy executes when the context delegates to its injected policy?

## Practical Tasks

- Replace a growing conditional with strategies.
- Test each strategy and the selection policy independently.

## Readiness Criteria

Identify independent variation, define a focused strategy contract, and explain when direct branching is preferable.

## References

### Microsoft Learn

- [Interfaces](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/interfaces)
