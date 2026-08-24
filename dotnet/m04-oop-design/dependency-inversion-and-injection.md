# Dependency Inversion and Dependency Injection

## Definition

**Dependency Inversion Principle (DIP)** is a design principle: high-level modules should depend on abstractions, not on low-level implementation details, and abstractions should not depend on details.

**Dependency Injection (DI)** is a technique for supplying an object's dependencies from the outside (constructor, method, or property) instead of the object creating them itself. DI is one common way to satisfy DIP, but they are not the same thing — you can apply DIP without a DI container, and you can inject concrete classes without inverting anything.

## Alternatives & Trade-offs

Compared to `new`-ing dependencies directly, DI adds indirection and a composition step, but enables substitutability (swap the real implementation for a test double), decoupling, and centralized lifetime management. For small scripts or truly stable, unlikely-to-change dependencies (e.g., `DateTime`), the overhead may not be worth it — but in application services, external I/O, and anything under test, it almost always is.

## How It Works

### Without inversion

```csharp
public class OrderService
{
    private readonly SqlOrderRepository _repository = new(); // hard dependency, hard to test
}
```

### With inversion

```csharp
public interface IOrderRepository
{
    Task SaveAsync(Order order);
}

public sealed class OrderService
{
    private readonly IOrderRepository _repository;

    public OrderService(IOrderRepository repository) => _repository = repository;
}
```

`OrderService` now depends on an abstraction it owns conceptually; `SqlOrderRepository` implements that abstraction, so the dependency arrow points *inward*, toward the high-level policy — hence "inversion."

### Registering with the built-in container

```csharp
services.AddScoped<IOrderRepository, SqlOrderRepository>();
services.AddScoped<OrderService>();
```

### Constructor vs. method vs. property injection

```csharp
// Constructor injection: dependency required for the object to be valid — preferred default
public OrderService(IOrderRepository repository) { }

// Method injection: dependency only needed for one operation
public Task ExportAsync(IOrderRepository repository, int orderId) => Task.CompletedTask;

// Property injection: optional dependency with a sensible default — used sparingly
public ILogger Logger { get; set; } = NullLogger.Instance;
```

### Service Locator anti-pattern

```csharp
// Hides dependencies inside the method body instead of the constructor signature
public class OrderService
{
    public void Process()
    {
        var repository = ServiceLocator.Current.GetService<IOrderRepository>();
    }
}
```

This compiles and "works," but it hides the class's real dependencies from its public contract, making the class harder to test and its requirements harder to see.

## Application

Use DIP/DI at the boundary between business logic and anything that varies independently or is expensive/impure to test directly: persistence, external HTTP calls, email/SMS providers, clock/time sources, file systems, and third-party SDKs.

## Common Mistakes

- Injecting a concrete class (`SqlOrderRepository`) instead of an abstraction, which achieves DI's wiring convenience without DIP's decoupling.
- Constructor over-injection: five or more dependencies in one constructor usually signals an SRP violation, not a DI problem.
- Using the service locator pattern to "inject" dependencies at call time.
- Registering a dependency with the wrong lifetime (e.g., a singleton holding a scoped `DbContext`), causing captive-dependency bugs.
- Creating an interface with exactly one implementation and no plausible second one "just in case," adding indirection with no real benefit.

## Common Interview Questions

### Basic
- What is the difference between Dependency Inversion and Dependency Injection?
- What are the three common forms of injection?

### Intermediate
- Why is constructor injection generally preferred over property injection?
- What problem does the service locator anti-pattern hide?
- How does DI improve unit testability?

### Advanced
- What is a captive dependency, and how does mismatched service lifetime cause it?
- How would you refactor a class with eight constructor parameters?
- How does DIP apply at an architectural level (e.g., domain layer not referencing infrastructure)?

### Follow-up Questions
- Can you apply DIP without using a DI container?
- What happens if you inject a singleton service into a scoped one?

### Code Prediction
```csharp
services.AddSingleton<ICache, MemoryCache>();
services.AddScoped<AppDbContext>();
// ICache implementation internally holds a reference to AppDbContext
```
What breaks here, and why is this a lifetime mismatch rather than a DI syntax error?

## Practical Tasks

- Refactor a class that directly instantiates `HttpClient` and `SmtpClient` to depend on injected abstractions instead.
- Write a unit test for `OrderService` using a fake `IOrderRepository`, demonstrating why the interface-based version is testable and the `new SqlOrderRepository()` version is not.
- Given a constructor with seven dependencies, identify which responsibilities should be split out.
- Diagnose a captive-dependency bug from a described symptom (stale data from a supposedly scoped service).

## Readiness Criteria

Explain DIP and DI as distinct but related concepts, choose the right injection style for a given scenario, recognize the service locator anti-pattern, and diagnose lifetime-mismatch bugs.

## References

### Microsoft Learn

- [Dependency injection in .NET](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection)
- [Dependency injection guidelines](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection-guidelines)
- [Service lifetimes](https://learn.microsoft.com/aspnet/core/fundamentals/dependency-injection#service-lifetimes)
