# Immutability in Object Design

## Definition

An immutable object cannot change observable state after construction. Operations return new values instead of mutating the existing instance.

## Alternatives & Trade-offs

Immutability simplifies sharing and concurrency but can create allocations or copying. Mutable objects can be efficient for controlled local state but require ownership discipline.

## How It Works

Use private or init-only setters, readonly fields, defensive copies, and immutable element types. A readonly reference prevents reassignment, not mutation of the referenced object.

## Application

Use immutable value objects, configuration snapshots, messages, and shared state that crosses thread boundaries.

## Common Mistakes

- Calling a type immutable while exposing mutable collections.
- Confusing shallow and deep immutability.
- Assuming `readonly` makes an object immutable.

## Common Interview Questions

### Basic
- What is immutability?
- What is the benefit of immutable objects?

### Intermediate
- How do you make a collection property safe?
- What is the difference between shallow and deep immutability?

### Advanced
- How do immutable snapshots support concurrency?
- When do persistent collections outperform defensive copying?
- How do records help and where do they fall short?

### Follow-up Questions
- Can an immutable object contain a mutable dependency?
- Is `readonly` enough?

### Code Prediction
Does changing a referenced list violate deep immutability?

## Practical Tasks

- Refactor a mutable DTO-like class into an immutable value object.
- Review a read-only API for mutable state leaks.

## Readiness Criteria

Explain ownership, shallow/deep immutability, copying, records, and the performance trade-offs of immutable design.

## References

### Microsoft Learn

- [Immutability in C#](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/records)
- [Init-only setters](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/init)
