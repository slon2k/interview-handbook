# Factory Pattern

## Definition

The Factory pattern centralizes object creation behind a method or class, so callers depend on what is created rather than how. It covers both the simple **Factory Method** (a method that returns different implementations based on input) and **Abstract Factory** (a family of related factories).

```csharp
public interface INotifier { void Notify(string message); }

public static class NotifierFactory
{
    public static INotifier Create(string channel) => channel switch
    {
        "email" => new EmailNotifier(),
        "sms" => new SmsNotifier(),
        _ => throw new NotSupportedException(channel)
    };
}
```

## Alternatives & Trade-offs

A factory hides construction complexity (which constructor, which configuration, which conditional logic) from callers, and is a natural seam for swapping implementations in tests. For simple, single-implementation types, a factory is unnecessary indirection — just call the constructor. Dependency injection containers often replace hand-written factories for object graphs resolved at startup; factories remain useful when the choice of implementation depends on a runtime value (like `channel` above) rather than being fixed by configuration.

## How It Works

```csharp
public sealed class EmailNotifier : INotifier
{
    public void Notify(string message) => Console.WriteLine($"Email: {message}");
}

public sealed class SmsNotifier : INotifier
{
    public void Notify(string message) => Console.WriteLine($"SMS: {message}");
}

// Caller depends only on INotifier, never on the concrete classes or the switch logic
INotifier notifier = NotifierFactory.Create(user.PreferredChannel);
notifier.Notify("Your order shipped");
```

### Factory injected as a dependency

```csharp
public interface INotifierFactory { INotifier Create(string channel); }

public sealed class OrderService
{
    private readonly INotifierFactory _factory;
    public OrderService(INotifierFactory factory) => _factory = factory;

    public void ShipOrder(Order order) =>
        _factory.Create(order.Customer.PreferredChannel).Notify("Shipped");
}
```

Wrapping the factory itself behind an interface allows `OrderService` to be unit tested with a fake factory.

## Application

Use a factory when the concrete type to create depends on runtime data (user preference, configuration value, feature flag) rather than being statically known, or when construction involves nontrivial setup you don't want scattered across call sites.

## Common Mistakes

- Adding a factory for a type with exactly one implementation and no runtime branching — pure ceremony.
- Letting the factory's `switch`/`if` chain grow unmanaged instead of registering implementations (e.g., via a dictionary or DI container) as new types are added.
- Confusing the Factory pattern with a DI container — a container resolves a fixed dependency graph, while a factory typically picks among implementations based on a runtime value.
- Making the factory a static class that is hard to substitute in tests, instead of injecting it behind an interface.

## Common Interview Questions

### Basic
- What problem does the Factory pattern solve?
- What's the difference between Factory Method and Abstract Factory?

### Intermediate
- When would you use a factory instead of just injecting the concrete implementation via DI?
- How do you keep a factory's branching logic from becoming unmanageable as types are added?

### Advanced
- How would you replace a `switch`-based factory with a registration-based design (e.g., `Dictionary<string, Func<INotifier>>`) to satisfy the Open/Closed Principle?
- How does a factory interact with a DI container when both are used together?

### Follow-up Questions
- Can a factory itself be injected as a dependency for testability?
- Is a static factory method always a bad idea?

### Code Prediction
Given the `NotifierFactory.Create` switch above, what happens when `channel` is `"push"` and no case matches? What design change would let a new `"push"` case be added without modifying `NotifierFactory`?

## Practical Tasks

- Refactor the `switch`-based `NotifierFactory` into a registration-based design using `Dictionary<string, Func<INotifier>>`.
- Wrap a factory behind an interface and write a unit test for a service that depends on it, using a fake factory.
- Identify a place in a hypothetical codebase where object construction logic is duplicated across call sites, and centralize it into a factory.

## Readiness Criteria

Explain when a factory earns its keep versus when it's unnecessary indirection, refactor a growing switch-based factory into an open/closed design, and use a factory as an injectable, testable dependency.

## References

### Microsoft Learn

- [Factory pattern guidance](https://learn.microsoft.com/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/implement-value-objects#factories)
- [Dependency injection](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection)
