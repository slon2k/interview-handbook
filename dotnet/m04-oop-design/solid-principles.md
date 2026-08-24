# SOLID Principles

## Definition

SOLID is a set of five object-oriented design guidelines that make code easier to change, test, and extend:

- **S** — Single Responsibility Principle (SRP)
- **O** — Open/Closed Principle (OCP)
- **L** — Liskov Substitution Principle (LSP)
- **I** — Interface Segregation Principle (ISP)
- **D** — Dependency Inversion Principle (DIP)

They are heuristics for managing change, not rules to maximize abstraction or class count.

## Alternatives & Trade-offs

SOLID trades a small amount of upfront structure for lower cost of change later. Over-applying it (an interface for every class, a strategy for every `if`) adds indirection without a real axis of change, which is its own maintenance cost. YAGNI and KISS are the natural counterweights — apply SOLID where change is expected or already happening, not everywhere by default.

## How It Works

### Single Responsibility

A class should have one reason to change.

```csharp
// Violates SRP: persistence, formatting, and business logic mixed together
public class InvoiceService
{
    public void Save(Invoice invoice) { /* EF Core code */ }
    public string FormatAsPdf(Invoice invoice) { /* PDF layout code */ }
    public decimal CalculateTotal(Invoice invoice) { /* business math */ }
}

// SRP applied: each class changes for a different reason
public class InvoiceRepository { public void Save(Invoice invoice) { } }
public class InvoicePdfFormatter { public string Format(Invoice invoice) => ""; }
public class InvoiceCalculator { public decimal CalculateTotal(Invoice invoice) => 0m; }
```

### Open/Closed

Behavior should be extendable without modifying stable, tested code.

```csharp
public interface IDiscountPolicy
{
    decimal Apply(decimal amount);
}

public sealed class SeasonalDiscount : IDiscountPolicy
{
    public decimal Apply(decimal amount) => amount * 0.9m;
}

// Adding a new discount policy does not require editing PriceCalculator
public sealed class PriceCalculator
{
    public decimal Calculate(decimal amount, IDiscountPolicy policy)
        => policy.Apply(amount);
}
```

### Liskov Substitution

A subtype must be usable anywhere its base type is expected, without surprising behavior.

```csharp
public class Rectangle
{
    public virtual int Width { get; set; }
    public virtual int Height { get; set; }
}

// Violates LSP: setting Width silently changes Height, breaking base-type expectations
public class Square : Rectangle
{
    public override int Width
    {
        get => base.Width;
        set { base.Width = value; base.Height = value; }
    }
}
```

Any code that does `rectangle.Width = 5; Assert.Equal(previousHeight, rectangle.Height);` breaks when given a `Square`.

### Interface Segregation

Clients should not be forced to depend on members they do not use.

```csharp
// Violates ISP: a read-only report never needs Save
public interface IRepository<T>
{
    T? GetById(int id);
    void Save(T entity);
    void Delete(int id);
}

// Segregated: consumers depend only on what they need
public interface IReadRepository<T> { T? GetById(int id); }
public interface IWriteRepository<T> { void Save(T entity); void Delete(int id); }
```

### Dependency Inversion

High-level policy should depend on abstractions, not on low-level implementation details.

```csharp
public interface IEmailSender { Task SendAsync(string to, string body); }

public sealed class OrderConfirmationService
{
    private readonly IEmailSender _emailSender; // depends on the abstraction

    public OrderConfirmationService(IEmailSender emailSender) => _emailSender = emailSender;

    public Task NotifyAsync(Order order) => _emailSender.SendAsync(order.CustomerEmail, "Order confirmed");
}
```

`OrderConfirmationService` never references `SmtpClient` or a vendor SDK directly; that detail is injected.

## Application

- **SRP** during code review, when a class keeps growing new unrelated methods.
- **OCP** when a `switch` on a type/category keeps gaining new cases as the business grows.
- **LSP** when reviewing inheritance hierarchies, especially before adding a new subtype.
- **ISP** when defining service contracts consumed by multiple, differently-scoped clients.
- **DIP** at architectural boundaries: infrastructure, external services, persistence.

## Common Mistakes

- Treating SRP as "one method per class" instead of "one reason to change."
- Using OCP to justify speculative plugin systems for behavior that never actually varies.
- Checking LSP only at the type-signature level, ignoring behavioral contracts (pre/post-conditions, exceptions thrown).
- Creating a wide interface "for symmetry" that no single client fully consumes.
- Confusing Dependency Inversion (depend on abstractions) with Dependency Injection (a mechanism for supplying implementations) — DI is one way to achieve DIP, not the principle itself.

## Common Interview Questions

### Basic
- What does each SOLID letter stand for?
- What problem does the Dependency Inversion Principle solve?

### Intermediate
- Give a concrete example of an LSP violation and explain why it breaks callers.
- How does the Interface Segregation Principle differ from just splitting a class into two?
- Why is DIP not the same thing as dependency injection?

### Advanced
- How do SOLID principles create tension with simplicity, and how do you decide when to stop applying them?
- How would you detect an abstraction introduced "for OCP" that has no real second implementation?
- How do architectural boundaries (e.g., clean architecture) enforce DIP structurally rather than by convention?

### Follow-up Questions
- Is SOLID always applicable, or are there contexts where it adds unnecessary cost?
- How do records and immutability interact with SRP?

### Code Prediction
Given the `Square`/`Rectangle` example above, what happens if a unit test does `rect.Width = 4; rect.Height = 6; Assert.Equal(24, rect.Width * rect.Height);` and is passed a `Square`? Why does this fail, and what does it reveal about LSP?

## Practical Tasks

- Take a class with three unrelated responsibilities and split it along SRP lines without breaking its public API.
- Refactor a `switch` statement that dispatches by enum into a strategy-based design that satisfies OCP.
- Write a unit test that would pass for `Rectangle` but fail for `Square`, then discuss what design change fixes it.
- Identify one interface in a hypothetical codebase that should be split for ISP, and split it.

## Readiness Criteria

Explain each principle with a concrete code example, identify real (not textbook) violations in existing code, distinguish DIP from DI, and justify when *not* applying a SOLID principle is the better trade-off.

## References

### Microsoft Learn

- [Dependency injection guidelines](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection-guidelines)
- [Interfaces in C#](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/interfaces)
- [Object-oriented programming concepts](https://learn.microsoft.com/dotnet/csharp/fundamentals/tutorials/oop)
