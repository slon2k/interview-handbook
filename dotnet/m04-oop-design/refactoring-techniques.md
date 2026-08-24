# Refactoring Techniques

## Definition

Refactoring is changing the internal structure of code without changing its observable behavior, usually to fix a code smell or make the next change easier. It is only safe with a reliable way to verify behavior is unchanged — ideally automated tests.

```csharp
// Before
public decimal Calc(decimal p, int q, decimal d) => p * q * (1 - d);

// After: Extract Method + Rename, same behavior, clearer intent
public decimal CalculateTotal(decimal unitPrice, int quantity, decimal discountRate)
    => ApplyDiscount(unitPrice * quantity, discountRate);

private decimal ApplyDiscount(decimal subtotal, decimal discountRate) => subtotal * (1 - discountRate);
```

## Alternatives & Trade-offs

Refactoring in small, verifiable steps (supported by tests) keeps risk low and lets you stop at any point with working code. The alternative — a large rewrite — can produce a better end result but carries much higher risk of introducing regressions and taking longer than planned, especially without a strong test suite to lean on.

## How It Works

### Extract Method

```csharp
// Before: one method doing three things
public void ProcessOrder(Order order)
{
    if (order.Items.Count == 0) throw new InvalidOperationException("Empty order");
    var total = order.Items.Sum(i => i.Price * i.Quantity);
    Console.WriteLine($"Total: {total}");
}

// After: each step named and independently testable
public void ProcessOrder(Order order)
{
    ValidateOrder(order);
    var total = CalculateTotal(order);
    PrintReceipt(total);
}

private void ValidateOrder(Order order) { if (order.Items.Count == 0) throw new InvalidOperationException("Empty order"); }
private decimal CalculateTotal(Order order) => order.Items.Sum(i => i.Price * i.Quantity);
private void PrintReceipt(decimal total) => Console.WriteLine($"Total: {total}");
```

### Replace Conditional with Polymorphism

```csharp
// Before
decimal Area(string shapeType, double a, double b) => shapeType switch
{
    "rectangle" => a * b,
    "triangle" => 0.5 * a * b,
    _ => throw new NotSupportedException()
};

// After
public interface IShape { double Area(); }
public sealed class Rectangle : IShape { public double A, B; public double Area() => A * B; }
public sealed class Triangle : IShape { public double A, B; public double Area() => 0.5 * A * B; }
```

### Introduce Parameter Object

```csharp
// Before: five related parameters passed around together everywhere
void Search(string query, int page, int pageSize, string sortBy, bool descending) { }

// After
public sealed record SearchOptions(string Query, int Page, int PageSize, string SortBy, bool Descending);
void Search(SearchOptions options) { }
```

### Safety net: tests before refactoring

```csharp
[Fact]
public void CalculateTotal_SumsItemPricesTimesQuantity()
{
    var order = new Order(new[] { new OrderItem(10m, 2), new OrderItem(5m, 1) });
    Assert.Equal(25m, CalculateTotal(order));
}
```
Running this test before and after each refactoring step confirms behavior hasn't changed.

## Application

Refactor incrementally as part of normal development ("leave the code better than you found it"), not as a separate, large, unplanned project. Always refactor with test coverage as a safety net where possible; if tests don't exist yet, consider writing characterization tests first.

## Common Mistakes

- Refactoring and adding new behavior in the same commit, making it hard to tell which change caused a regression.
- Doing a large, risky refactor without any tests in place first.
- Renaming or restructuring public APIs during a refactor without considering downstream callers — a true refactor preserves observable behavior for all callers, not just internal ones.
- Refactoring code that's about to be deleted or replaced, spending effort with no payoff.

## Common Interview Questions

### Basic
- What is refactoring, and how is it different from adding a feature or fixing a bug?
- Why is test coverage important before refactoring?

### Intermediate
- Name a few common refactoring techniques (Extract Method, Introduce Parameter Object, Replace Conditional with Polymorphism).
- What's the risk of mixing refactoring with behavior changes in the same commit?

### Advanced
- How would you safely refactor a method with no existing tests?
- How do you decide the right scope for a refactor — a single method, a class, or a whole module — given limited time?

### Follow-up Questions
- Is renaming a public method a refactor if it breaks external callers?
- How does refactoring relate to fixing the code smells covered earlier?

### Code Prediction
Given the "Replace Conditional with Polymorphism" example above, what has to change at the call site when a new `"circle"` shape type is added — before the refactor versus after?

## Practical Tasks

- Refactor a method with an untested but well-understood side effect, first adding a characterization test, then applying Extract Method.
- Apply Replace Conditional with Polymorphism to a given `switch`-based method.
- Introduce a parameter object for a method with five or more related parameters.

## Readiness Criteria

Apply common refactoring techniques by name, explain why tests should precede risky refactors, and keep refactoring commits separate from behavior-changing commits.

## References

### Microsoft Learn

- [Unit testing best practices](https://learn.microsoft.com/dotnet/core/testing/unit-testing-best-practices)
- [Refactoring guidance in Visual Studio](https://learn.microsoft.com/visualstudio/ide/refactoring-in-visual-studio)
