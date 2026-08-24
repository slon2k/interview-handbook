# Composition vs. Inheritance

## Definition

**Inheritance** models an "is-a" relationship: a derived type reuses and extends a base type's implementation. **Composition** models a "has-a" relationship: a type achieves reuse by holding a reference to another type and delegating to it. "Favor composition over inheritance" is a common design guideline, not an absolute rule.

```csharp
// Inheritance
public class FileLogger : LoggerBase { }

// Composition
public class OrderService
{
    private readonly ILogger _logger; // OrderService "has a" logger
    public OrderService(ILogger logger) => _logger = logger;
}
```

## Alternatives & Trade-offs

| | Inheritance | Composition |
|---|---|---|
| Coupling | Tight — derived class depends on base implementation details | Loose — depends only on an interface |
| Flexibility | Fixed at compile time, single base class | Can swap the composed object at runtime |
| Reuse granularity | Whole-class reuse, hard to reuse partially | Reuse exactly the behavior you need |
| Fragility | Base class changes can silently break derived classes ("fragile base class") | Implementation changes are isolated behind the interface |
| Natural fit | Genuine taxonomies (`Square is-a Shape`) | Cross-cutting or swappable behavior (logging, caching, notification) |

## How It Works

### The fragile base class problem

```csharp
public class Collection
{
    private readonly List<int> _items = new();
    public virtual void Add(int item) => _items.Add(item);
    public void AddRange(IEnumerable<int> items)
    {
        foreach (var item in items) Add(item); // calls the virtual Add
    }
}

public class CountingCollection : Collection
{
    public int AddCount { get; private set; }
    public override void Add(int item) { AddCount++; base.Add(item); }
}
```

Calling `AddRange` on a `CountingCollection` increments `AddCount` once per item — a detail of `Collection`'s internal implementation (`AddRange` calling the virtual `Add`) that `CountingCollection` had to know about to behave correctly. If `Collection` is later changed to bypass `Add` inside `AddRange` for performance, `CountingCollection` silently breaks with no compiler error.

### Composition avoids that coupling

```csharp
public sealed class CountingCollection
{
    private readonly List<int> _items = new();
    public int AddCount { get; private set; }

    public void Add(int item) { AddCount++; _items.Add(item); }
    public void AddRange(IEnumerable<int> items)
    {
        foreach (var item in items) Add(item); // explicit, owned logic
    }
}
```

There is no hidden dependency on another class's internal call pattern.

## Application

Use inheritance for genuine type taxonomies where Liskov substitution holds cleanly (`Circle`/`Square` as `Shape`). Use composition for cross-cutting concerns, swappable strategies, and anything you expect to change or mock independently — logging, caching, notification channels, payment providers.

## Common Mistakes

- Using inheritance purely to reuse a method, for types that aren't conceptually related ("a `ReportGenerator` extends `List<string>` to reuse its collection methods").
- Deep inheritance chains (4+ levels) that make it hard to reason about which class actually implements a given behavior.
- Overriding a virtual method without understanding how the base class calls it internally, recreating the fragile base class problem shown above.
- Assuming "favor composition" means *never* use inheritance — genuine is-a relationships with a stable base are still a good fit.

## Common Interview Questions

### Basic
- What's the difference between "is-a" and "has-a"?
- What does "favor composition over inheritance" mean in practice?

### Intermediate
- What is the fragile base class problem, and how does composition avoid it?
- When is inheritance still the better choice than composition?

### Advanced
- How does composition support the Open/Closed Principle better than deep inheritance hierarchies?
- How would you refactor a 4-level inheritance chain into a composition-based design without breaking existing callers?
- How does composition interact with dependency injection to enable runtime behavior swapping?

### Follow-up Questions
- Can composition and inheritance be combined in the same design?
- Does using an interface plus composition always eliminate fragile base class risk?

### Code Prediction
Using the `Collection`/`CountingCollection` example above, what does `AddCount` equal after `AddRange(new[] {1, 2, 3})`? What would happen to that value if `Collection.AddRange` were changed to add directly to `_items` without calling `Add`?

## Practical Tasks

- Refactor a class that inherits from a base class purely for code reuse into one that uses composition and an injected interface instead.
- Identify the fragile base class risk in a small inheritance hierarchy and rewrite it safely.
- Design a notification system (email, SMS, push) using composition so channels can be added without modifying existing code.

## Readiness Criteria

Explain is-a vs. has-a with examples, identify the fragile base class problem in code, and justify a composition-vs-inheritance choice for a given design scenario rather than defaulting to one or the other.

## References

### Microsoft Learn

- [Inheritance in C#](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/inheritance)
- [Object-oriented programming concepts](https://learn.microsoft.com/dotnet/csharp/fundamentals/tutorials/oop)
