# Inheritance

## Definition

Inheritance lets a derived type reuse and extend a base type. It models an is-a relationship.

## Alternatives & Trade-offs

Inheritance supports polymorphism and shared behavior, but creates tight coupling to the base class. Composition is often easier to evolve.

## How It Works

A derived class inherits accessible members and can override virtual members. Constructors run from base to derived; disposal and cleanup require deliberate design.

## Application

Use inheritance for stable taxonomies and substitutable base contracts, not merely to reuse implementation.

## Common Mistakes

- Inheriting for code reuse alone.
- Calling overridable members during construction.
- Breaking base-class invariants in derived classes.

## Common Interview Questions

### Basic
- What is inheritance?
- What is a base class?

### Intermediate
- How does overriding differ from hiding?
- What is the Liskov Substitution Principle?

### Advanced
- How can a fragile base class affect derived types?
- When should a hierarchy be sealed?
- How do versioning and binary compatibility affect base classes?

### Follow-up Questions
- Can C# inherit from multiple classes?
- What does `base` do?

### Code Prediction
Which constructor executes first when creating a derived object?

## Practical Tasks

- Review a hierarchy for an LSP violation.
- Replace inheritance-based reuse with composition where appropriate.

## Readiness Criteria

Explain substitutability, virtual dispatch, constructor order, and the coupling costs of hierarchies.

## References

### Microsoft Learn

- [Inheritance](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/inheritance)
- [Object-oriented programming](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/)
