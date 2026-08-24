# Strategy Pattern

## Definition

The Strategy pattern extracts an interchangeable algorithm or business rule into its own type implementing a shared interface, so the algorithm can be selected and swapped at runtime without changing the code that uses it.

```csharp
public interface IShippingCostStrategy
{
    decimal Calculate(Order order);
}

public sealed class FlatRateShipping : IShippingCostStrategy
{
    public decimal Calculate(Order order) => 5.99m;
}

public sealed class WeightBasedShipping : IShippingCostStrategy
{
    public decimal Calculate(Order order) => order.TotalWeightKg * 1.5m;
}
```

## Alternatives & Trade-offs

Strategy replaces conditional branching (`if`/`switch` on a type or flag) with polymorphic dispatch, satisfying the Open/Closed Principle — new strategies don't require editing existing code. For a rule that genuinely never varies, a plain method is simpler; introducing a strategy interface for a single, permanent implementation adds indirection without benefit.

## How It Works

### Before: conditional branching

```csharp
decimal CalculateShipping(Order order, string method) => method switch
{
    "flat" => 5.99m,
    "weight" => order.TotalWeightKg * 1.5m,
    _ => throw new NotSupportedException(method)
};
```

Every new shipping method requires editing this function.

### After: strategy injected

```csharp
public sealed class CheckoutService
{
    private readonly IShippingCostStrategy _shippingStrategy;
    public CheckoutService(IShippingCostStrategy shippingStrategy) => _shippingStrategy = shippingStrategy;

    public decimal CalculateTotal(Order order) =>
        order.Subtotal + _shippingStrategy.Calculate(order);
}

var checkout = new CheckoutService(new WeightBasedShipping());
```

`CheckoutService` never branches on shipping type; a new `ExpressShipping` strategy can be added and injected without modifying `CheckoutService` at all.

### Selecting a strategy at runtime

```csharp
var strategies = new Dictionary<string, IShippingCostStrategy>
{
    ["flat"] = new FlatRateShipping(),
    ["weight"] = new WeightBasedShipping()
};

IShippingCostStrategy selected = strategies[order.ShippingMethod];
```

## Application

Use Strategy for business rules that vary by configuration, customer tier, region, or A/B test — pricing rules, shipping calculations, validation policies, sorting/ranking algorithms — anywhere a `switch` on a category is likely to keep growing.

## Common Mistakes

- Introducing a strategy interface for a rule with exactly one implementation and no expected variation.
- Reintroducing a `switch` to *select* the strategy at the composition root and calling that "removing the switch" — the branching moved, it wasn't eliminated; the win is that business logic no longer branches, only the wiring does.
- Giving each strategy hidden dependencies on mutable shared state, making them behave inconsistently depending on call order.
- Confusing Strategy with Template Method: Strategy swaps the whole algorithm via composition; Template Method varies *steps* of a fixed algorithm via inheritance.

## Common Interview Questions

### Basic
- What problem does the Strategy pattern solve?
- How does Strategy relate to the Open/Closed Principle?

### Intermediate
- How is Strategy typically composed with Dependency Injection in an ASP.NET Core app?
- What's the difference between Strategy and Template Method?

### Advanced
- Where does the "selection" branching go when you remove branching from business logic via Strategy, and why is that still an improvement?
- How would you test each strategy implementation in isolation, and how would you test that `CheckoutService` correctly delegates to whichever strategy it's given?

### Follow-up Questions
- Can Strategy be implemented with a delegate (`Func<Order, decimal>`) instead of an interface?
- Is a `Dictionary<string, IStrategy>` lookup itself a violation of Open/Closed?

### Code Prediction
Given `CalculateTotal` above, what changes are needed to add a new `ExpressShipping` strategy — does `CheckoutService` need to change? What does that tell you about which class satisfies the Open/Closed Principle here?

## Practical Tasks

- Refactor the `CalculateShipping` switch statement into an injectable Strategy-based design.
- Add a new shipping strategy without modifying `CheckoutService`, and write a unit test proving `CheckoutService` correctly delegates.
- Implement the same Strategy example using a `Func<Order, decimal>` delegate instead of an interface, and compare readability and testability.

## Readiness Criteria

Refactor a conditional-branching business rule into an interchangeable Strategy, correctly explain where selection logic lives after the refactor, and distinguish Strategy from Template Method and from Factory.

## References

### Microsoft Learn

- [Strategy pattern](https://learn.microsoft.com/dotnet/standard/design-patterns/strategy)
- [Dependency injection](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection)
