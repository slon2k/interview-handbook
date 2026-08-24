# Abstraction

## Definition

Abstraction exposes essential behavior while hiding implementation details.

## Alternatives & Trade-offs

Interfaces and abstract classes provide explicit contracts. Concrete classes are simpler when substitution is not required. Too many abstractions add indirection without value.

## How It Works

An interface defines capabilities; an abstract class can provide shared state or implementation. Callers depend on the contract rather than the implementation.

## Application

Use abstraction at change boundaries such as persistence, external services, and policies.

## Common Mistakes

- Creating interfaces for every class automatically.
- Leaking implementation details through an abstraction.
- Confusing abstraction with encapsulation.

## Common Interview Questions

### Basic
- What is abstraction?
- How do interfaces provide abstraction?

### Intermediate
- When is an abstract class better than an interface?
- What makes an abstraction useful?

### Advanced
- How can a leaky abstraction increase coupling?
- How do abstractions affect testing and versioning?
- When should you remove an abstraction?

### Follow-up Questions
- Is an interface always an abstraction?
- Can abstraction exist without inheritance?

### Code Prediction
Which implementation can replace an interface dependency without changing the caller?

## Practical Tasks

- Extract a focused interface from a concrete dependency.
- Identify and remove an abstraction that exposes database details.

## Readiness Criteria

Explain contracts, hidden implementation, substitution, and the cost of unnecessary abstraction.

## References

### Microsoft Learn

- [Interfaces](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/interfaces)
- [Abstract and sealed classes](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/abstract)
