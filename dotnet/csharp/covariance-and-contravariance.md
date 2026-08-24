# Covariance and Contravariance

## Definition

Variance describes when a generic type can be converted to another constructed type based on a reference conversion between their type arguments.

- Covariance (`out`) preserves the direction of a relationship: a producer of a more-derived type can be used as a producer of a base type.
- Contravariance (`in`) reverses the direction: a consumer of a base type can be used as a consumer of a derived type.

```csharp
IEnumerable<string> names = ["Ada"];
IEnumerable<object> objects = names; // covariance

Action<object> consumeObject = value => { };
Action<string> consumeString = consumeObject; // contravariance
```

## Alternatives & Trade-offs

Variance improves substitutability and API flexibility, but it applies only where the type safely produces or consumes values. Invariant generic types are appropriate when a type both reads and writes values.

Prefer interfaces and delegates with clear variance annotations. Do not add variance merely to force conversions that make an API harder to understand.

## How It Works

An interface or delegate can declare a type parameter as `out` when it appears only in output positions, or `in` when it appears only in input positions.

```csharp
interface IProducer<out T>
{
    T Get();
}

interface IConsumer<in T>
{
    void Put(T value);
}
```

Variance conversions are reference conversions. `IEnumerable<int>` cannot become `IEnumerable<object>` because `int` is a value type, even though the interface is covariant.

A mutable collection such as `List<T>` is invariant because allowing both reads and writes would be unsafe.

## Application

- Accept `IEnumerable<Base>` when callers may provide sequences of derived objects.
- Accept `Action<Derived>` when a consumer can safely consume derived values.
- Define producer and consumer interfaces with `out` and `in`.
- Use invariant collections when both reading and writing are required.

## Common Mistakes

- Reversing covariance and contravariance.
- Expecting variance for classes such as `List<T>`.
- Expecting variance conversions for value types.
- Marking a type parameter `out` while using it as a method input.
- Casting a covariant interface back to a mutable collection.
- Assuming variance changes the objects or their runtime types.

## Common Interview Questions

### Basic

- What is covariance?
- What is contravariance?
- What do `in` and `out` mean on generic type parameters?
- Why is `List<T>` invariant?

### Intermediate

- Why is `IEnumerable<string>` assignable to `IEnumerable<object>`?
- Why can an `Action<object>` be assigned to an `Action<string>`?
- Which interfaces and delegates support variance?
- Why does variance apply only to reference types?

### Advanced

- How do variance rules preserve type safety?
- Why can a covariant type parameter not be used as a method input?
- How do nested generic types affect variance conversions?
- How does delegate variance interact with method-group conversion?
- What API design rules determine whether an interface should be variant?
- How do nullable reference types affect variant assignments?
- Why are arrays covariant and what runtime failure can that permit?
- How would you adapt an invariant collection to a read-only producer API?
- How does variance affect binary compatibility when changing interfaces?
- How would you test a public API for unsafe or surprising variance conversions?

### Follow-up Questions

- Is `ICollection<T>` covariant?
- Can a value type participate in variance?
- What is the difference between variance and casting?
- Why is `Action<T>` contravariant?
- What exception can an invalid array write produce?

### Code Prediction

What is printed?

```csharp
IEnumerable<string> names = ["Ada"];
IEnumerable<object> values = names;
Console.WriteLine(values.First());
```

Is this assignment valid?

```csharp
Action<object> handler = value => Console.WriteLine(value);
Action<string> stringHandler = handler;
```

## Practical Tasks

### API Design

Create `IReader<out T>` and `IWriter<in T>` interfaces and demonstrate valid assignments.

### Safety Review

Explain why `IEnumerable<string>` can be exposed as `IEnumerable<object>` but `List<string>` cannot be exposed as `List<object>`.

### Delegate Exercise

Write examples showing covariance in return values and contravariance in delegate parameters.

## Readiness Criteria

You should be able to explain producer/consumer variance, identify valid conversions, design variant interfaces safely, and distinguish variance from ordinary casting and inheritance.

## References

### Microsoft Learn

- [Covariance and contravariance in generics](https://learn.microsoft.com/dotnet/standard/generics/covariance-and-contravariance)
- [Creating variant generic interfaces](https://learn.microsoft.com/dotnet/csharp/programming-guide/concepts/covariance-contravariance/creating-variant-generic-interfaces)
- [Using variance in generic interfaces](https://learn.microsoft.com/dotnet/csharp/programming-guide/concepts/covariance-contravariance/using-variance-in-generic-interfaces)
- [Variance in delegates](https://learn.microsoft.com/dotnet/csharp/programming-guide/concepts/covariance-contravariance/variance-in-delegates)
