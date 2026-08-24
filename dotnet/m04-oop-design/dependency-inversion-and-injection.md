# Dependency Inversion and Dependency Injection

## Definition

Dependency Inversion makes high-level policy depend on stable abstractions rather than low-level details. Dependency injection supplies those dependencies from outside the dependent class.

```csharp
public sealed class OrderService(IClock clock)
{
    public bool IsOpen() => clock.UtcNow.Hour < 17;
}
```

## Alternatives & Trade-offs

Constructor injection makes dependencies visible and testable. Factories, method injection, or direct construction can be appropriate for short-lived or stable dependencies. Abstraction has a cost and should reflect a real boundary.

## How It Works

A composition root creates concrete implementations and passes them into application services. The service depends on the contract, not the construction mechanism.

## Application

Use injection for external systems, clocks, policies, persistence, and behavior that must vary or be replaced in tests.

## Common Mistakes

- Injecting a service locator instead of dependencies.
- Creating huge constructors instead of splitting responsibilities.
- Hiding required dependencies in optional properties.
- Confusing DIP with a specific container.

## Common Interview Questions

### Basic
- What is dependency injection?
- What is dependency inversion?

### Intermediate
- Why is constructor injection usually preferred?
- What is a composition root?

### Advanced
- How can injection reveal poor class cohesion?
- How do lifetimes and scopes affect injected dependencies?
- When is an abstraction unnecessary?

### Follow-up Questions
- What is the difference between DIP and DI?
- Why avoid service locator?

### Code Prediction
Which class is easier to test when its clock is injected?

## Practical Tasks

- Replace direct construction of an external client with constructor injection.
- Design a composition root without leaking container APIs into domain classes.

## Readiness Criteria

Explain DIP, DI, composition roots, constructor injection, lifetime concerns, and the cost of unnecessary abstractions.

## References

### Microsoft Learn

- [Dependency injection in .NET](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection)
- [Dependency injection guidelines](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection-guidelines)
