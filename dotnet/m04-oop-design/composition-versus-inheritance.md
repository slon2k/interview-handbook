# Composition versus Inheritance

## Definition

Composition builds behavior by combining objects. Inheritance specializes a base type.

## Alternatives & Trade-offs

Composition limits coupling and supports replacement. Inheritance offers direct substitutability and shared implementation, but base changes affect derived types.

## How It Works

A composed class delegates work to collaborators. An inherited class receives base behavior and participates in virtual dispatch.

```csharp
public interface IShippingPolicy
{
	decimal Calculate(decimal weight);
}

public sealed class OrderService(IShippingPolicy shippingPolicy)
{
	public decimal ShippingFor(decimal weight) => shippingPolicy.Calculate(weight);
}
```

## Application

Prefer composition when behavior varies independently or the relationship is has-a rather than is-a.

## Common Mistakes

- Treating every reuse relationship as inheritance.
- Hiding collaborators behind an overly broad facade.
- Using inheritance where substitutability is false.

## Common Interview Questions

### Basic
- What is composition?
- What is an is-a relationship?

### Intermediate
- Why is composition often preferred?
- When is inheritance appropriate?

### Advanced
- How does composition improve testing and versioning?
- How can composition become excessive indirection?
- How do decorators use composition?

### Follow-up Questions
- Is delegation the same as composition?
- Can composition provide polymorphism?

### Code Prediction
Which design allows replacing behavior without changing the owning class?

## Practical Tasks

- Refactor a subclass hierarchy into collaborators.
- Defend inheritance for a stable, genuinely substitutable domain hierarchy.

## Readiness Criteria

Evaluate relationship semantics, coupling, substitution, testing, and changeability before choosing either approach.

## References

### Microsoft Learn

- [Inheritance](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/inheritance)
- [Polymorphism](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/polymorphism)
