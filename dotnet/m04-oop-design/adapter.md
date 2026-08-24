# Adapter Pattern

## Definition

Adapter converts one interface into another interface expected by a client.

## Alternatives & Trade-offs

An adapter isolates an incompatible dependency. A direct change may be simpler when the dependency is owned and stable; wrappers add a type and mapping boundary.

## How It Works

The adapter implements the target interface and delegates to the adaptee, translating calls, values, and errors.

## Application

Use adapters around third-party SDKs, legacy APIs, transport clients, or incompatible domain models.

## Common Mistakes

- Leaking the adaptee through the target interface.
- Mixing unrelated business logic into the adapter.
- Hiding incompatible error semantics.

## Common Interview Questions

### Basic
- What problem does Adapter solve?
- What is the adaptee?

### Intermediate
- How does Adapter differ from Decorator?
- Why isolate third-party APIs?

### Advanced
- How should adapters translate cancellation and failures?
- How can an adapter preserve compatibility across SDK versions?
- When is an anti-corruption layer more appropriate?

### Follow-up Questions
- Is Adapter inheritance or composition?
- Can an adapter be bidirectional?

### Code Prediction
Which interface does the client depend on after adaptation?

## Practical Tasks

- Wrap a legacy client behind a domain interface.
- Test mapping, failures, and cancellation at the adapter boundary.

## Readiness Criteria

Explain interface conversion, dependency isolation, mapping, error semantics, and adapter versus wrapper trade-offs.

## References

### Microsoft Learn

- [Interfaces](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/interfaces)
