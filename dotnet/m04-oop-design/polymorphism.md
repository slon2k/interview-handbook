# Polymorphism

## Definition

Polymorphism allows one contract to represent different implementations.

## Alternatives & Trade-offs

Virtual dispatch and interfaces support runtime substitution. Pattern matching and explicit branching can be clearer for closed data variants. Polymorphism adds flexibility but can make control flow less visible.

## How It Works

C# supports interface dispatch, virtual/override dispatch, overload resolution, and generic static polymorphism. Overloads are selected at compile time; overrides at runtime.

## Application

Use polymorphism for strategies, plugins, domain behaviors, and dependencies with multiple implementations.

## Common Mistakes

- Confusing overload resolution with overriding.
- Casting to concrete types throughout callers.
- Creating deep hierarchies for small variations.

## Common Interview Questions

### Basic
- What is polymorphism?
- What is virtual dispatch?

### Intermediate
- How do overloads and overrides differ?
- How do interfaces enable substitution?

### Advanced
- How does dispatch affect performance and versioning?
- When is pattern matching better than polymorphism?
- How do generic constraints provide static polymorphism?

### Follow-up Questions
- Can static methods be overridden?
- What is method hiding?

### Code Prediction
Which implementation runs when a virtual method is called through a base reference?

## Practical Tasks

- Implement a strategy using an interface.
- Compare a type switch with a polymorphic design.

## Readiness Criteria

Distinguish compile-time and runtime polymorphism and choose an appropriate substitution mechanism.

## References

### Microsoft Learn

- [Polymorphism](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/polymorphism)
- [Virtual and override](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/virtual)
