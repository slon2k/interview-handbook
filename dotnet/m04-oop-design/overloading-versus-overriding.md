# Overloading vs. Overriding

## Definition

**Overloading** defines multiple methods with the same name but different parameter lists in the same type; the compiler picks one at compile time based on the argument types. **Overriding** replaces a base class's `virtual`/`abstract` method implementation in a derived class; the runtime picks the implementation based on the object's actual type.

```csharp
// Overloading
public void Print(int value) { }
public void Print(string value) { }

// Overriding
public class Shape { public virtual double Area() => 0; }
public class Circle : Shape { public override double Area() => 3.14; }
```

## Alternatives & Trade-offs

Overloading gives a convenient, uniform API name for conceptually similar operations on different input types, resolved with no runtime cost. Overriding gives polymorphic behavior across a type hierarchy at a small runtime dispatch cost. Confusing the two — expecting overload resolution to behave polymorphically based on runtime type — is a common source of subtle bugs.

## How It Works

### Overload resolution happens at compile time, based on static type

```csharp
void Show(object o) => Console.WriteLine("object");
void Show(string s) => Console.WriteLine("string");

object x = "hello";
Show(x); // prints "object" — the compiler picks the overload based on x's *declared* type (object), not its runtime type
```

This surprises people who expect "the more specific overload for the runtime type" to run — that is override behavior, not overload behavior.

### Overriding dispatches at runtime, based on the actual object

```csharp
Shape shape = new Circle();
Console.WriteLine(shape.Area()); // prints Circle's Area, even though the variable is typed as Shape
```

### Overload resolution ambiguity

```csharp
void Process(int a, double b) { }
void Process(double a, int b) { }

Process(1, 2); // ambiguous? No — Process(int, double) requires one implicit conversion (int->double for b),
                // Process(double, int) requires one implicit conversion (int->double for a): genuinely ambiguous, compile error
```

## Application

Use overloading for the same logical operation on different, unrelated input shapes (`Console.WriteLine(int)`, `Console.WriteLine(string)`). Use overriding for a base class defining a contract that subclasses specialize (`Shape.Area()`).

## Common Mistakes

- Expecting overload resolution to use an object's runtime type, when it actually uses the compile-time (static) type of the expression.
- Adding an overload that behaves inconsistently with existing overloads (different exception behavior, different null-handling) for the "same" conceptual operation.
- Confusing method hiding (`new`) with overriding when a subclass redefines a non-virtual method with the same signature — this is neither overloading nor true overriding.
- Creating ambiguous overloads that only resolve differently through implicit conversions, making the caller's actual selected overload non-obvious.

## Common Interview Questions

### Basic
- What's the key difference between overloading and overriding?
- Does overload resolution happen at compile time or runtime?

### Intermediate
- Why does `Show(x)` above print "object" instead of "string" even though `x` holds a string at runtime?
- Can you overload a method by return type alone?

### Advanced
- How does the compiler resolve overload ambiguity when multiple overloads require implicit conversions?
- How does overriding combined with `base.Method()` calls affect the execution order in a multi-level hierarchy?

### Follow-up Questions
- Can a method be both overloaded and overridden at the same time?
- What does `sealed override` mean, and why would you use it?

### Code Prediction
Given the `Show(object)`/`Show(string)` example above, what prints if `x` is declared as `string x = "hello";` directly (not through an `object` variable) and passed to `Show(x)`?

## Practical Tasks

- Write a small program demonstrating that overload resolution uses static type, not runtime type.
- Build a `Shape` hierarchy using `override` and show polymorphic dispatch through a `List<Shape>`.
- Identify and fix an ambiguous overload pair in a given code sample.

## Readiness Criteria

Explain the compile-time vs. runtime resolution difference precisely, predict output for tricky overload/override examples, and avoid designing ambiguous or inconsistent overload sets.

## References

### Microsoft Learn

- [Overload resolution](https://learn.microsoft.com/dotnet/csharp/language-reference/operators/member-access-operators#overload-resolution)
- [Polymorphism](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/polymorphism)
