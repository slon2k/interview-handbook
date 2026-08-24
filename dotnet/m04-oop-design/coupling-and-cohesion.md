# Coupling and Cohesion

## Definition

**Coupling** is the degree to which one module depends on the internal details of another. **Cohesion** is the degree to which the responsibilities inside a single module belong together. The goal is low coupling between modules and high cohesion within each one.

```csharp
// High coupling: OrderService reaches directly into SqlConnection details
public class OrderService
{
    public void Save(Order order)
    {
        using var conn = new SqlConnection("Server=...");
        // raw SQL here
    }
}
```

## Alternatives & Trade-offs

Low coupling (via interfaces, DI, events) makes modules independently changeable and testable, at the cost of an extra layer of indirection to navigate. High cohesion keeps a class's methods and fields focused on one concept, making it easier to name, understand, and test — the alternative, a "God class" with many unrelated responsibilities, is easy to reach for early but expensive to maintain.

## How It Works

### Reducing coupling

```csharp
public interface IOrderRepository { void Save(Order order); }

public class OrderService
{
    private readonly IOrderRepository _repository;
    public OrderService(IOrderRepository repository) => _repository = repository;
    public void Save(Order order) => _repository.Save(order);
}
```

`OrderService` no longer knows about `SqlConnection` or connection strings — it's coupled only to an abstraction it can substitute in tests.

### Low cohesion example

```csharp
public class UserManager
{
    public void CreateUser(User user) { }
    public void SendWelcomeEmail(User user) { }
    public decimal CalculateInvoiceTotal(Invoice invoice) { } // unrelated responsibility
    public void GenerateSalesReport() { } // also unrelated
}
```

`UserManager` mixes user management, invoicing, and reporting — three unrelated reasons to change bundled into one class.

### High cohesion after splitting

```csharp
public class UserService { public void CreateUser(User user) { } }
public class WelcomeEmailSender { public void Send(User user) { } }
public class InvoiceCalculator { public decimal CalculateTotal(Invoice invoice) => 0m; }
public class SalesReportGenerator { public void Generate() { } }
```

Each class now has one focused purpose and one reason to change.

## Application

Measure coupling and cohesion during code review: if a class needs to change for several unrelated reasons, cohesion is low; if a small internal change in one class forces edits across many unrelated classes, coupling is too high. Use interfaces, events, and DI to reduce coupling; use SRP to raise cohesion.

## Common Mistakes

- Conflating "coupling" with "using other classes" — depending on an abstraction is normal and healthy; depending on another class's internal implementation details is the problem.
- Assuming more interfaces automatically means lower coupling — an interface with a single, tightly-coupled implementation and leaking implementation details doesn't actually decouple much.
- Ignoring low cohesion because "it's just a utility class" — utility/manager classes are a common home for accumulated, unrelated responsibilities.

## Common Interview Questions

### Basic
- What is coupling, and what is cohesion?
- Why do we want low coupling and high cohesion?

### Intermediate
- How does depending on an interface reduce coupling compared to depending on a concrete class?
- What's a practical sign that a class has low cohesion?

### Advanced
- How do coupling and cohesion relate to the Single Responsibility Principle and to testability?
- How would you measure coupling in a real codebase (e.g., afferent/efferent coupling, dependency graphs)?

### Follow-up Questions
- Can a highly cohesive class still be tightly coupled to the rest of the system?
- Does introducing an interface always reduce coupling?

### Code Prediction
Given the `UserManager` example above, if `CalculateInvoiceTotal`'s logic changes, does anything about `CreateUser` or `SendWelcomeEmail` need to change? What does the answer reveal about why these responsibilities are bundled incorrectly?

## Practical Tasks

- Split a low-cohesion "manager" class into several single-purpose classes.
- Refactor a class with a hard dependency on a concrete database class into one coupled only to an interface.
- Given a dependency diagram, identify the most tightly coupled module and propose a decoupling strategy.

## Readiness Criteria

Explain coupling and cohesion with concrete examples, identify low-cohesion classes and high-coupling dependencies in real code, and refactor both using SRP and dependency inversion.

## References

### Microsoft Learn

- [Common architectural principles](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/architectural-principles)
- [Dependency injection guidelines](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection-guidelines)
