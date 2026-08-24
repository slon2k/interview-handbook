# API and Class Design

## Definition

API and class design define responsibilities, contracts, invariants, dependencies, and change boundaries.

## Alternatives & Trade-offs

Small focused types are easier to test and change, while overly fragmented types increase navigation and wiring cost. Public APIs deserve stronger compatibility discipline than internal code.

## How It Works

Constructors establish required state; methods express meaningful operations; interfaces expose capabilities. Visibility, mutability, parameter choices, and return types become part of the contract.

## Application

Design around caller needs, stable domain concepts, explicit ownership, and predictable failure behavior.

## Common Mistakes

- Anemic types with no invariant protection.
- Leaking concrete dependencies or mutable collections.
- Adding options and overloads without a coherent contract.

## Common Interview Questions

### Basic
- What makes a class cohesive?
- What belongs in a public API?

### Intermediate
- How do you design for testability?
- How do you choose method and property visibility?

### Advanced
- How do API decisions affect binary and source compatibility?
- How do you evolve a contract without breaking callers?
- When should a result type replace exceptions or tuples?

### Follow-up Questions
- What makes a good constructor?
- When should a class be sealed?

### Code Prediction
Which public member creates the strongest long-term compatibility obligation?

## Practical Tasks

- Review and simplify a service API.
- Design a class around invariants, ownership, and failure behavior.

## Readiness Criteria

Defend responsibilities, visibility, dependencies, mutability, compatibility, and failure choices in a class design.

## References

### Microsoft Learn

- [Classes](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/classes)
- [Design guidelines](https://learn.microsoft.com/dotnet/standard/design-guidelines/)
