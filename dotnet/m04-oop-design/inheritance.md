# Inheritance

## Definition

Inheritance lets a derived class reuse and extend the members of a base class, modeling an "is-a" relationship. C# supports single inheritance of implementation (one base class) plus multiple interface implementation.

```csharp
public class Employee
{
    public string Name { get; init; } = "";
    public virtual decimal CalculatePay() => 0m;
}

public class SalariedEmployee : Employee
{
    public decimal MonthlySalary { get; init; }
    public override decimal CalculatePay() => MonthlySalary;
}
```

## Alternatives & Trade-offs

Inheritance gives free reuse of base-class members and polymorphic dispatch, but couples derived classes to the base class's implementation details and its evolution (see the fragile base class problem). Composition is often preferable for reuse alone; inheritance is best reserved for genuine taxonomies where Liskov substitution holds.

## How It Works

### Virtual, override, and base

```csharp
public class Shape
{
    public virtual double Area() => 0;
}

public class Circle : Shape
{
    private readonly double _radius;
    public Circle(double radius) => _radius = radius;
    public override double Area() => Math.PI * _radius * _radius;
}

Shape shape = new Circle(2);
Console.WriteLine(shape.Area()); // 12.57 — dispatches to Circle.Area at runtime
```

### `new` hides instead of overrides

```csharp
public class Base { public void Show() => Console.WriteLine("Base"); }
public class Derived : Base { public new void Show() => Console.WriteLine("Derived"); }

Base b = new Derived();
b.Show(); // prints "Base" — non-virtual calls are resolved at compile time by the declared type
```

This is a classic interview trap: `new` hides a member for callers using the derived type directly, but does not participate in polymorphic dispatch through a base-typed reference.

### Sealed classes and methods

```csharp
public class SalariedEmployee : Employee
{
    public sealed override decimal CalculatePay() => MonthlySalary; // no further overriding allowed
}
```

## Application

Use inheritance when subclasses genuinely are specializations of the base type and can honor its contract (Liskov substitution) — a family of `PaymentMethod` types, a hierarchy of `Shape`s with a real `Area()` contract. Avoid it purely to reuse code between otherwise unrelated types.

## Common Mistakes

- Confusing `new` (member hiding) with `override` (polymorphic dispatch) — a very common source of "why did the wrong method run" bugs.
- Building deep hierarchies (4+ levels) that obscure where behavior actually comes from.
- Overriding a method without honoring the base class's implicit contract, breaking Liskov substitution.
- Forgetting that non-virtual methods cannot be overridden — only hidden — leading to unexpected dispatch through base-typed references.

## Common Interview Questions

### Basic
- What is the difference between `virtual`, `override`, and `new`?
- Can a C# class inherit from more than one class?

### Intermediate
- What happens when a non-virtual method is called through a base-class reference to a derived object?
- What does `sealed` do when applied to a class vs. a method?

### Advanced
- How does the fragile base class problem manifest through inheritance specifically?
- How would you decide between inheritance and composition for a new type hierarchy?

### Follow-up Questions
- Can a derived class call a base class's constructor explicitly? How?
- What happens if a base class constructor calls a virtual method?

### Code Prediction
Given the `Base`/`Derived`/`new Show()` example above, what is printed by `b.Show()` when `b` is declared as `Base` but holds a `Derived` instance? What would print instead if `Show` were declared `virtual`/`override`?

## Practical Tasks

- Build a small `Employee` hierarchy with `virtual`/`override` and demonstrate polymorphic dispatch through a `List<Employee>`.
- Reproduce the `new` vs. `override` behavior difference in code and explain the output.
- Identify a Liskov violation in a given inheritance hierarchy and refactor it.

## Readiness Criteria

Explain virtual/override/new precisely with correct dispatch behavior, recognize the fragile base class problem, and judge when inheritance is the right tool versus composition.

## References

### Microsoft Learn

- [Inheritance in C#](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/inheritance)
- [Polymorphism](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/polymorphism)
- [Knowing when to use Override and New keywords](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/knowing-when-to-use-override-and-new-keywords)
