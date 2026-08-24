# Interfaces vs. Abstract Classes

## Definition

An **interface** defines a contract — a set of members implementers must provide — with no state and (traditionally) no implementation. An **abstract class** is a partially implemented base class: it can hold state, provide default method implementations, and still declare abstract members that derived classes must implement. Neither can be instantiated directly.

```csharp
public interface IShape
{
    double Area();
}

public abstract class ShapeBase
{
    public string Name { get; protected set; } = "";
    public abstract double Area();
    public string Describe() => $"{Name}: {Area():F2}";
}
```

## Alternatives & Trade-offs

| | Interface | Abstract class |
|---|---|---|
| Multiple inheritance | A class can implement many interfaces | A class can extend only one abstract class |
| Shared state | No fields; only members (default interface methods can carry logic, not state) | Can hold fields and shared state |
| Shared implementation | Only via default interface methods (C# 8+) | Naturally, via concrete methods |
| Versioning | Adding a member breaks all implementers unless given a default implementation | Adding a concrete method is safe for existing derived classes |
| Intent | "Can do X" — a capability/contract | "Is a kind of Y" — a shared identity and partial implementation |

Prefer interfaces when unrelated types need to share a capability (`IComparable`, `IDisposable`). Prefer an abstract class when types share genuine identity and meaningful common state/behavior, and single inheritance is not a constraint.

## How It Works

### Default interface methods (C# 8+)

```csharp
public interface ILogger
{
    void Log(string message);
    void LogError(string message) => Log($"ERROR: {message}"); // default implementation
}
```

Existing implementers of `ILogger` do not break when `LogError` is added, because it has a default body — but this still cannot hold instance state the way an abstract class field can.

### Combining both

```csharp
public interface IShape { double Area(); }

public abstract class ShapeBase : IShape
{
    public string Name { get; protected set; } = "";
    public abstract double Area(); // still abstract; concrete shapes implement it
}

public sealed class Circle : ShapeBase
{
    private readonly double _radius;
    public Circle(double radius) { _radius = radius; Name = "Circle"; }
    public override double Area() => Math.PI * _radius * _radius;
}
```

`Circle` gets shared state and behavior from `ShapeBase`, and is still substitutable anywhere `IShape` is expected.

## Application

Use an interface to define a role a type can play (`IEmailSender`, `IEquatable<T>`), especially across unrelated class hierarchies. Use an abstract class when several concrete types are genuinely variations of the same concept and benefit from sharing fields, constructors, or protected helper methods — e.g., a family of `PaymentProcessor` subclasses sharing retry logic.

## Common Mistakes

- Choosing an abstract class purely to share code, when the types involved aren't conceptually related — this couples unrelated types to one inheritance chain.
- Assuming default interface methods let interfaces fully replace abstract classes; they still can't hold instance fields.
- Making an interface "fat" with many unrelated members instead of following Interface Segregation.
- Forgetting that a class can extend only one abstract class, which can force an awkward hierarchy if two unrelated abstract classes both seem to apply.

## Common Interview Questions

### Basic
- Can you instantiate an interface or an abstract class directly?
- Can a class implement multiple interfaces? Multiple abstract classes?

### Intermediate
- What are default interface methods, and what problem do they solve?
- When would you choose an abstract class over an interface if both could technically work?

### Advanced
- Why can't default interface methods fully replace abstract classes?
- How does adding a member to a widely-implemented interface affect binary/source compatibility, and how do default implementations mitigate that?
- How would you redesign a hierarchy that mixes "is-a" and "can-do" concerns into one abstract class?

### Follow-up Questions
- Can an abstract class implement an interface without implementing all of its members?
- Can an interface extend another interface?

### Code Prediction
```csharp
public interface IShape { double Area(); }
public abstract class ShapeBase : IShape
{
    public abstract double Area();
}
```
Can `ShapeBase` be instantiated? Can a variable of type `IShape` reference a `ShapeBase`-derived instance?

## Practical Tasks

- Design both an interface-based and abstract-class-based solution for a family of payment processors, and justify which fits better.
- Add a new member to an existing interface using a default implementation, and verify existing implementers still compile.
- Refactor a class hierarchy that uses inheritance purely for code reuse (unrelated types forced into one base class) into an interface plus composition.

## Readiness Criteria

Explain the structural and semantic differences between interfaces and abstract classes, choose correctly between them for a given design problem, and explain how default interface methods change (and don't change) that trade-off.

## References

### Microsoft Learn

- [Interfaces](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/interfaces)
- [Abstract and sealed classes](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/inheritance)
- [Default interface methods](https://learn.microsoft.com/dotnet/csharp/whats-new/tutorials/default-interface-members-versions)
