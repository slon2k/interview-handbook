# Tuples and Deconstruction

## Definition

Tuples group multiple values into one value without requiring a named type. C# tuple syntax is backed by `System.ValueTuple`.

```csharp
(string Name, int Age) person = ("Ada", 36);
Console.WriteLine(person.Name);

var (name, age) = person;
```

Tuple elements can have names, but those names are compile-time metadata and are not a substitute for a stable domain contract.

## Alternatives & Trade-offs

Use a tuple for short-lived, local combinations of related values. Use a record, class, or dedicated struct when the result crosses a public API boundary, needs validation, or has domain meaning.

Tuples are value types and can avoid object allocation, but large tuples can be expensive to copy and unnamed tuple APIs can reduce readability.

## How It Works

Tuple syntax maps to `ValueTuple` types such as `ValueTuple<T1, T2>`. Tuple assignment and equality compare elements in order.

```csharp
var first = (1, 2);
var second = (1, 2);
Console.WriteLine(first == second);
```

Deconstruction calls a `Deconstruct` method when one is available. It can target existing variables, discard values with `_`, or participate in pattern matching.

## Application

- Return two or three local results from a private method.
- Deconstruct records and domain types for readable extraction.
- Use discards when only some values matter.
- Use named tuple elements for local clarity.

## Common Mistakes

- Exposing unnamed tuples as long-lived public contracts.
- Assuming tuple element names affect runtime identity.
- Confusing tuple syntax with `System.Tuple`, which is a reference type.
- Accidentally copying large tuples.
- Reusing `_` in a way that obscures which value is discarded.

## Common Interview Questions

### Basic

- What is a tuple?
- How do you create and deconstruct a tuple?
- What is a discard?
- Are C# tuples reference types or value types?

### Intermediate

- What is the difference between `Tuple` and `ValueTuple`?
- How are tuple names represented?
- How does a custom `Deconstruct` method work?
- Can tuples be returned from methods?

### Advanced

- What compatibility risks exist when changing tuple element names in a public API?
- How does tuple equality work for nested or nullable elements?
- How does overload resolution use tuple conversions?
- When can tuple copying affect performance?
- How do tuples interact with generic type inference?
- How are tuples represented in metadata and across language boundaries?
- How would you preserve domain invariants that a tuple cannot enforce?
- How do deconstruction patterns differ from ordinary method calls?
- What are the trade-offs between tuples and record structs?
- How would you design a versionable multi-value API?

### Follow-up Questions

- Can a tuple contain another tuple?
- Can you deconstruct a class?
- What happens to tuple element names after assignment?
- Why might a record be better than a tuple?
- What does `_` mean in deconstruction?

### Code Prediction

What is printed?

```csharp
var result = (Name: "Ada", Score: 10);
var (name, _) = result;
Console.WriteLine(name);
```

What is printed?

```csharp
var left = (1, 2);
var right = (1, 2);
Console.WriteLine(left == right);
```

## Practical Tasks

### Return Design

Write a method that returns a parsed value and an error message. Decide whether a tuple or a result record is more appropriate.

### Deconstruction

Add a `Deconstruct` method to a `Point` type and use it in a switch expression.

### API Review

Review a public method returning `(bool Success, string Message, object? Value)` and propose a more maintainable contract.

## Readiness Criteria

You should be able to create, compare, return, and deconstruct tuples, explain their value-type behavior, and choose between a tuple and a named domain type.

## References

### Microsoft Learn

- [Tuple types](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/value-tuples)
- [Deconstructing tuples and other types](https://learn.microsoft.com/dotnet/csharp/fundamentals/functional/deconstruct)
- [System.ValueTuple](https://learn.microsoft.com/dotnet/api/system.valuetuple)
