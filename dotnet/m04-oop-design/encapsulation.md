# Encapsulation

## Definition

Encapsulation keeps an object's state behind a controlled public API. The type protects its invariants instead of exposing unrestricted mutation.

## Alternatives & Trade-offs

Private fields with methods or restricted properties improve safety and changeability. Fully open data objects are simpler but allow invalid states.

## How It Works

C# access modifiers, properties, constructors, and methods define the boundary. Validation belongs at the boundary where invalid state could enter.

## Application

Use encapsulation for domain rules, lifecycle transitions, and state that must remain consistent.

## Common Mistakes

- Exposing public fields.
- Adding getters and setters without protecting invariants.
- Allowing constructors to create invalid objects.

## Common Interview Questions

### Basic
- What is encapsulation?
- Why prefer properties over public fields?

### Intermediate
- How do access modifiers support encapsulation?
- Where should invariant validation occur?

### Advanced
- How can encapsulation reduce coupling and versioning risk?
- When is a read-only property insufficient?
- How do immutable objects enforce encapsulation?

### Follow-up Questions
- Is a private field enough to guarantee a valid object?
- How does encapsulation differ from abstraction?

### Code Prediction
What prevents invalid state in a well-designed encapsulated type?

## Practical Tasks

- Refactor a public-field model into an invariant-protecting class.
- Design a state transition API that rejects invalid transitions.

## Readiness Criteria

Explain how a type protects state, identify leaks in an API, and design boundaries around invariants.

## References

### Microsoft Learn

- [Encapsulation](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/encapsulation)
- [Access modifiers](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/access-modifiers)
