# Abstraction

## Definition

Abstraction means exposing only the essential behavior of a concept and hiding the implementation details behind a simpler interface. Where encapsulation protects state, abstraction manages *complexity* by letting a caller depend on "what" something does without knowing "how."

```csharp
public interface IPaymentGateway
{
    Task<PaymentResult> ChargeAsync(decimal amount, string cardToken);
}
```

A caller of `IPaymentGateway` never needs to know whether it calls Stripe, a bank API, or a mock.

## Alternatives & Trade-offs

More abstraction layers reduce coupling to specific implementations and make testing/swapping easier, but each layer adds a level of indirection that must be understood and navigated when debugging. Under-abstraction leaks implementation details everywhere (SQL strings scattered through business logic); over-abstraction (an interface, a factory, and a strategy for a single, never-varying implementation) makes simple code harder to follow for no real gain.

## How It Works

### Abstraction via interface

```csharp
public interface INotifier
{
    Task NotifyAsync(string userId, string message);
}

public sealed class EmailNotifier : INotifier
{
    public Task NotifyAsync(string userId, string message) => Task.CompletedTask; // SMTP details hidden here
}

public sealed class OrderService
{
    private readonly INotifier _notifier;
    public OrderService(INotifier notifier) => _notifier = notifier;

    public Task CompleteOrderAsync(string userId) =>
        _notifier.NotifyAsync(userId, "Your order has shipped"); // no idea how notification actually happens
}
```

### Abstraction via abstract class

```csharp
public abstract class ReportGenerator
{
    public string Generate(Report report)
    {
        var header = BuildHeader(report);
        var body = BuildBody(report);
        return header + body; // orchestration is abstracted away from subclasses
    }

    protected abstract string BuildHeader(Report report);
    protected abstract string BuildBody(Report report);
}
```

Subclasses implement only the varying parts; the overall algorithm shape is abstracted into the base class.

## Application

Abstract at architectural seams — persistence, external services, notification channels — and around genuinely varying business rules. Don't abstract implementation details that have exactly one realistic implementation and no test-isolation need.

## Common Mistakes

- Introducing an interface with a single implementation and no plan for a second one or for testing, purely out of habit ("program to an interface" taken too literally).
- Abstracting at the wrong level — hiding *what* something does behind a name so generic (`IManager`, `IHelper`) that the abstraction communicates nothing.
- Leaking implementation details through the abstraction's shape, e.g., an `IRepository` whose method returns an EF Core `IQueryable`, exposing the ORM through the "abstraction."
- Confusing abstraction with abstract classes specifically — abstraction is the design idea; abstract classes and interfaces are two of its mechanisms.

## Common Interview Questions

### Basic
- What is abstraction, and how is it different from encapsulation?
- Give an example of abstraction in the .NET base class library.

### Intermediate
- What's wrong with an interface that has exactly one implementation and no tests using it?
- How does abstraction support testability?

### Advanced
- How do you decide the right "shape" for an abstraction so it doesn't leak implementation details?
- How does abstraction interact with the Dependency Inversion Principle?

### Follow-up Questions
- Is a `DbContext` itself an abstraction over the database?
- Can too much abstraction make code harder to read?

### Code Prediction
Given `IRepository<T> { IQueryable<T> Query(); }`, what implementation detail does this abstraction still leak to its callers, and why does that undermine the point of abstracting the repository at all?

## Practical Tasks

- Design an abstraction (interface) for a "notification channel" concept that could later support email, SMS, and push without changing calling code.
- Take a service directly calling `HttpClient` and abstract the external call behind a purpose-named interface.
- Critique a given interface for leaking implementation details and propose a fix.

## Readiness Criteria

Distinguish abstraction from encapsulation, design interfaces that hide implementation rather than merely renaming it, and judge when an abstraction is adding value versus adding ceremony.

## References

### Microsoft Learn

- [Object-oriented programming concepts](https://learn.microsoft.com/dotnet/csharp/fundamentals/tutorials/oop)
- [Interfaces](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/interfaces)
