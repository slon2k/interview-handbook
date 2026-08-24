# Polymorphism

## Definition

Polymorphism lets code work with objects of different types through a common interface, with the actual behavior determined by the object's runtime type. C# supports **runtime (subtype) polymorphism** via `virtual`/`override` and interfaces, and **compile-time (ad hoc) polymorphism** via method overloading and generics.

```csharp
public interface IShape { double Area(); }
public class Circle : IShape { public double Area() => 3.14; }
public class Square : IShape { public double Area() => 4; }

IShape[] shapes = { new Circle(), new Square() };
foreach (var shape in shapes)
    Console.WriteLine(shape.Area()); // each call dispatches to the correct implementation
```

## Alternatives & Trade-offs

Runtime polymorphism (interfaces/virtual methods) trades a small dispatch cost for open-ended extensibility — new types can be added without touching existing code. A `switch` on a type tag achieves similar branching without the abstraction cost, but every new case requires editing the switch, violating the Open/Closed Principle. Generics give compile-time polymorphism with no runtime dispatch cost, at the price of needing a shared constraint rather than full behavioral substitution.

## How It Works

### Runtime polymorphism through an interface

```csharp
public interface IDiscount { decimal Apply(decimal price); }
public class NoDiscount : IDiscount { public decimal Apply(decimal price) => price; }
public class PercentDiscount : IDiscount
{
    private readonly decimal _percent;
    public PercentDiscount(decimal percent) => _percent = percent;
    public decimal Apply(decimal price) => price * (1 - _percent);
}

decimal Checkout(decimal price, IDiscount discount) => discount.Apply(price);
```

`Checkout` never branches on discount type; the correct behavior is selected by which object was passed in.

### Compile-time polymorphism via overloading

```csharp
public static int Add(int a, int b) => a + b;
public static double Add(double a, double b) => a + b;
```

The compiler picks the overload based on argument types at compile time — no virtual dispatch involved.

### The classic `switch` alternative and why it doesn't scale

```csharp
double Area(object shape) => shape switch
{
    Circle c => 3.14 * c.Radius * c.Radius,
    Square s => s.Side * s.Side,
    _ => throw new NotSupportedException()
};
```

Every new shape requires editing this method. The polymorphic version (each shape implementing its own `Area()`) does not.

## Application

Use runtime polymorphism when new variants are expected to be added over time and callers shouldn't need to change (payment methods, discount rules, notification channels). Use overloading/generics for compile-time variation where the operation is fundamentally the same shape across types.

## Common Mistakes

- Writing a type-checking `switch`/`if-else` chain over a class hierarchy instead of letting polymorphic dispatch handle it — a strong signal that Open/Closed is being violated.
- Assuming overloading is polymorphism in the same sense as virtual dispatch — it's resolved at compile time based on static types, not at runtime based on the actual object.
- Boxing value types unnecessarily when using polymorphism through a non-generic interface (e.g., `IComparable` vs. `IComparable<T>`).

## Common Interview Questions

### Basic
- What is polymorphism, and what are its two main forms in C#?
- What is the difference between method overloading and overriding?

### Intermediate
- Why does a type-checking `switch` over a class hierarchy usually indicate a missed opportunity for polymorphism?
- How is overload resolution decided — at compile time or runtime?

### Advanced
- How does polymorphism support the Open/Closed Principle in practice?
- How do generics provide "polymorphism" without runtime virtual dispatch, and what are the performance implications?

### Follow-up Questions
- Can a `static` method be polymorphic in the runtime-dispatch sense?
- How does polymorphism interact with Liskov substitution — what happens when a subtype doesn't honor the base contract?

### Code Prediction
```csharp
public class Base { public virtual void Speak() => Console.WriteLine("Base"); }
public class Derived : Base { public override void Speak() => Console.WriteLine("Derived"); }

Base obj = new Derived();
obj.Speak();
```
What is printed, and why does it differ from calling a `new`-hidden method through a base reference?

## Practical Tasks

- Refactor a type-checking `switch` over a shape hierarchy into a polymorphic design using an interface.
- Implement a discount system where new discount types can be added without modifying the checkout method.
- Explain, with code, the difference in dispatch timing between an overload and an override.

## Readiness Criteria

Distinguish compile-time and runtime polymorphism precisely, refactor type-switching code into polymorphic designs, and connect polymorphism to the Open/Closed Principle.

## References

### Microsoft Learn

- [Polymorphism](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/polymorphism)
- [Generics](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/generics)
