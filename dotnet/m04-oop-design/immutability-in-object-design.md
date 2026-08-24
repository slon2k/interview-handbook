# Immutability in Object Design

## Definition

An immutable object's state cannot change after construction. Instead of mutating an object, operations that would "change" it return a new instance with the updated state.

```csharp
public sealed record Money(decimal Amount, string Currency)
{
    public Money Add(Money other)
    {
        if (other.Currency != Currency) throw new InvalidOperationException("Currency mismatch");
        return this with { Amount = Amount + other.Amount }; // returns a new instance
    }
}
```

## Alternatives & Trade-offs

Immutable objects are inherently thread-safe (no locking needed for read access), simpler to reason about (no action-at-a-distance mutation), and safe as dictionary keys or in hash sets. The trade-off is allocation: every "change" creates a new object, which can matter in high-throughput or memory-sensitive code. Mutable objects avoid that allocation cost but require careful discipline around shared references and concurrent access.

## How It Works

### Mutable version — aliasing risk

```csharp
public class MutableMoney
{
    public decimal Amount { get; set; }
}

var price = new MutableMoney { Amount = 100 };
var discountedPrice = price;
discountedPrice.Amount -= 10; // also changes price.Amount, because both variables reference the same object
Console.WriteLine(price.Amount); // 90 — surprising if the caller expected `price` to be untouched
```

### Immutable version — no aliasing risk

```csharp
public sealed record ImmutableMoney(decimal Amount)
{
    public ImmutableMoney Discount(decimal amount) => this with { Amount = Amount - amount };
}

var price = new ImmutableMoney(100);
var discountedPrice = price.Discount(10);
Console.WriteLine(price.Amount);           // 100 — unchanged
Console.WriteLine(discountedPrice.Amount); // 90 — a distinct instance
```

### `readonly` and `init` for enforced immutability

```csharp
public sealed class Point
{
    public double X { get; init; } // settable only during object initialization
    public double Y { get; init; }
}

var p = new Point { X = 1, Y = 2 };
// p.X = 5; // compile error — init-only property cannot be set after construction
```

### Immutable collections

```csharp
public sealed class Order
{
    private readonly ImmutableList<OrderLine> _lines;
    public Order(ImmutableList<OrderLine> lines) => _lines = lines;
    public Order AddLine(OrderLine line) => new(_lines.Add(line)); // returns a new Order
}
```

## Application

Favor immutability for value objects (`Money`, `Address`, `DateRange`), objects shared across threads, dictionary/hash-set keys, and domain events (a fact that already happened shouldn't be mutable). Mutable state is often still the right choice for entities with genuine identity and lifecycle (an `Order` being built up through a UI form) or performance-critical hot paths.

## Common Mistakes

- Assuming `readonly` on a field makes the referenced object immutable — it only prevents reassigning the field itself; a `readonly List<T>` can still be mutated via `Add()`.
- Implementing "immutable" objects that expose a mutable collection property, defeating the immutability.
- Overusing immutability in hot paths where the allocation cost of constant new-object creation becomes measurable, without profiling first.
- Forgetting that mutable object equality (`Equals`) can become unstable if a mutable field used in `GetHashCode()` changes after the object is stored in a `HashSet`.

## Common Interview Questions

### Basic
- What does it mean for an object to be immutable?
- What is the difference between `readonly`, `init`, and a mutable `set`?

### Intermediate
- Why does `readonly List<T>` not make the list itself immutable?
- How does immutability help with thread safety?

### Advanced
- What are the performance trade-offs of immutable data structures under high allocation pressure, and how do immutable collection types mitigate some of that cost structurally?
- How does immutability interact with equality and hashing for objects used as dictionary keys?

### Follow-up Questions
- Are records always immutable?
- How would you make a class with a mutable internal list expose an immutable view of it?

### Code Prediction
Given the `MutableMoney`/`ImmutableMoney` example above, what values do `price.Amount` and `discountedPrice.Amount` hold after `discountedPrice.Amount -= 10` on the mutable version versus `price.Discount(10)` on the immutable version?

## Practical Tasks

- Convert a mutable `Money` class into an immutable record with a `with`-expression-based `Add` method.
- Fix a class that exposes a mutable `List<T>` property so external code cannot mutate its internal state.
- Reproduce and explain the aliasing bug in the `MutableMoney` example, then fix it using immutability.

## Readiness Criteria

Explain why immutability helps with thread safety and aliasing bugs, correctly distinguish `readonly`/`init`/mutable semantics, and judge when immutability's allocation cost is or isn't worth paying.

## References

### Microsoft Learn

- [Immutability in C#](https://learn.microsoft.com/dotnet/csharp/whats-new/tutorials/immutability)
- [Records](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/record)
- [System.Collections.Immutable](https://learn.microsoft.com/dotnet/api/system.collections.immutable)
