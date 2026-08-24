# Generics and Constraints

## Definition

Generics allow types and methods to operate on a type parameter while preserving compile-time type safety.

```csharp
static T First<T>(IReadOnlyList<T> values) => values[0];
```

A constraint limits which types may be used for a type parameter.

```csharp
static T Create<T>() where T : new() => new T();
```

## Alternatives & Trade-offs

Generics are usually preferable to `object`-based APIs because they avoid casts and can avoid boxing for value types. Interfaces are better when behavior must be substituted at runtime without making the consumer generic.

Generic APIs can become complex when many constraints or type parameters are involved. Keep constraints minimal and design for readable call sites.

## How It Works

The runtime creates specialized generic code for value types and can share code for many reference-type instantiations. Generic constraints give the compiler and runtime guarantees about available members and conversions.

Common constraints include `class`, `struct`, `notnull`, `unmanaged`, a base class, an interface, and `new()`.

```csharp
static bool Equal<T>(T left, T right) where T : IEquatable<T>
    => left.Equals(right);
```

Generic variance applies only to reference types and only to interface and delegate type parameters declared `in` or `out`.

## Application

- Build reusable collections, algorithms, and services.
- Preserve type safety across APIs.
- Use constraints to express required capabilities.
- Use generic math interfaces for numeric algorithms where appropriate.
- Use static abstract interface members for extensible generic operators.

## Common Mistakes

- Using `where T : class` when value types should also be valid.
- Assuming `default(T)` is always a meaningful value.
- Boxing a value type through an unconstrained interface or `object` conversion.
- Adding broad constraints only to make an implementation compile.
- Confusing generic type parameters with runtime reflection types.
- Assuming generic variance applies to classes or value types.

## Common Interview Questions

### Basic

- What problem do generics solve?
- What is a generic type parameter?
- What is a generic constraint?
- What does `default(T)` return?

### Intermediate

- What is the difference between `class` and `struct` constraints?
- Why use `IEquatable<T>` in a generic algorithm?
- What does `new()` require?
- Why can generics reduce boxing?

### Advanced

- How does CLR generic code sharing work?
- What are the performance implications of constrained calls?
- How do generic constraints affect overload resolution?
- Why are static abstract interface members useful for generic math?
- How do `notnull` and nullable analysis interact with generics?
- What constraints are required for `Span<T>`-based generic APIs?
- How do generic type parameters affect trimming and AOT compilation?
- When does an unconstrained generic operation box a value type?
- How do generic variance and constraints interact?
- How would you design a generic API that remains readable as capabilities grow?

### Follow-up Questions

- Can a generic type parameter be a pointer type?
- What is the difference between `T?` for value and reference types?
- Why is `new()` not a general dependency-injection strategy?
- Can a generic method be overloaded only by constraints?
- What is a reified generic type?

### Code Prediction

What is printed?

```csharp
static T Echo<T>(T value) => value;
Console.WriteLine(Echo(42).GetType().Name);
```

What happens here?

```csharp
static T Create<T>() where T : new() => new T();
// Create<int>() is valid; Create<string>() is not.
```

## Practical Tasks

### Generic Collection

Implement a type-safe `Pair<TFirst, TSecond>` with read-only properties and deconstruction.

### Constraint Design

Write a generic method that accepts only equatable values and explain why the constraint matters.

### Performance Review

Compare an `object[]` algorithm with a generic equivalent and measure boxing and allocations.

## Readiness Criteria

You should be able to write generic methods and types, select appropriate constraints, explain boxing and runtime specialization, and recognize when a non-generic abstraction is clearer.

## References

### Microsoft Learn

- [Generics in .NET](https://learn.microsoft.com/dotnet/standard/generics/)
- [Generic type parameter constraints](https://learn.microsoft.com/dotnet/csharp/programming-guide/generics/constraints-on-type-parameters)
- [Generic classes and methods](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/generics)
- [Generic math](https://learn.microsoft.com/dotnet/standard/generics/math)
