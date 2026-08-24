# Interfaces versus Abstract Classes

## Definition

An interface defines a contract. An abstract class defines a contract and may provide shared implementation or state.

## Alternatives & Trade-offs

Interfaces support multiple contracts and loose coupling. Abstract classes support controlled shared behavior but allow only one base class.

## How It Works

A class implements interfaces and inherits one base class. Interfaces can include default implementations, but shared state and constructor behavior remain class concerns.

## Application

Use interfaces for capabilities and external boundaries; use abstract classes for a cohesive family with invariant shared behavior.

## Common Mistakes

- Choosing an abstract class solely for convenience.
- Putting unrelated responsibilities into one interface.
- Treating default interface members as a replacement for clear design.

## Common Interview Questions

### Basic
- What is an interface?
- What is an abstract class?

### Intermediate
- When would you choose one over the other?
- Can a class implement multiple interfaces?

### Advanced
- How do interface evolution and default members affect compatibility?
- How can abstract base classes violate substitutability?
- How do explicit implementations control API exposure?

### Follow-up Questions
- Can an abstract class be instantiated?
- Can an interface contain implementation?

### Code Prediction
Which members are available through an interface-typed reference?

## Practical Tasks

- Split a large interface into focused capabilities.
- Design an abstract base only where shared invariants are real.

## Readiness Criteria

Compare contracts, shared behavior, state, substitution, multiple implementation, and versioning trade-offs.

## References

### Microsoft Learn

- [Interfaces](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/interfaces)
- [Abstract and sealed classes](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/abstract)
