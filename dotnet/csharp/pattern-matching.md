# Pattern Matching

## Definition

Pattern matching tests a value against a shape or type and can extract information when the match succeeds.

```csharp
if (value is int number and > 0)
{
    Console.WriteLine(number);
}
```

C# supports type, constant, relational, logical, property, positional, list, and var patterns.

## Alternatives & Trade-offs

Pattern matching often makes branching over types and shapes concise. Traditional `if` statements remain clearer for complex imperative workflows. A polymorphic design may be better than repeatedly switching on runtime types when behavior belongs to the types themselves.

Switch expressions are concise and return a value, but complex nested patterns can become difficult to read.

## How It Works

Patterns are checked from left to right in logical combinations. A `switch` expression selects the first matching arm.

```csharp
static string Describe(object value) => value switch
{
    null => "null",
    int n when n > 0 => "positive integer",
    string { Length: 0 } => "empty string",
    _ => "other"
};
```

The compiler performs exhaustiveness analysis for many known domains. Guards introduced with `when` refine a match but do not generally make the compiler prove exhaustiveness.

## Application

- Validate and classify input values.
- Handle discriminated-union-like object hierarchies.
- Parse tokens and command arguments.
- Match records and immutable data shapes.
- Use list patterns for fixed or prefix/suffix array shapes.

## Common Mistakes

- Ordering a broad pattern before a more specific pattern.
- Using `var` patterns when a clearer type or property pattern exists.
- Assuming a `when` guard makes a switch exhaustive.
- Casting repeatedly instead of using one type pattern.
- Hiding business rules in an unreadable pattern expression.
- Forgetting that a null value does not match an ordinary type pattern.

## Common Interview Questions

### Basic

- What is pattern matching?
- What is a type pattern?
- What does `_` mean in a switch expression?
- What is the difference between a switch statement and expression?

### Intermediate

- How do property and relational patterns work?
- What is a positional pattern?
- How do `and`, `or`, and `not` patterns combine?
- How does the compiler check exhaustiveness?

### Advanced

- How do recursive patterns traverse nested object shapes?
- How does pattern variable scope work across `or` patterns?
- What are the runtime costs of type and property patterns?
- How can pattern ordering create unreachable-arm compiler errors?
- When should a polymorphic design replace a type switch?
- How do list patterns interact with spans and arrays?
- How do nullable flow states interact with pattern matching?
- How would you model an exhaustive domain union in current C#?
- What versioning risks exist when adding derived types to a switch?
- How would you test a complex pattern set for overlapping cases?

### Follow-up Questions

- Does `is` compare object identity?
- How does `is null` differ from `== null`?
- Can a property pattern match null properties?
- What happens when no switch arm matches?
- Why is a discard arm often useful?

### Code Prediction

What is printed?

```csharp
object value = 12;
Console.WriteLine(value is int number and > 10 ? number : 0);
```

What is printed?

```csharp
int[] values = [1, 2, 3];
Console.WriteLine(values is [1, .., 3]);
```

## Practical Tasks

### Classifier

Write a switch expression that classifies an object as null, positive integer, non-empty string, or other.

### Validation

Use property and relational patterns to validate a request object without nested null checks.

### Review

Find overlapping or unreachable arms in a switch expression and rewrite it for clarity.

## Readiness Criteria

You should be able to use the major pattern forms, explain ordering and exhaustiveness, recognize null behavior, and choose between pattern matching and polymorphism.

## References

### Microsoft Learn

- [Pattern matching overview](https://learn.microsoft.com/dotnet/csharp/fundamentals/functional/pattern-matching)
- [Patterns](https://learn.microsoft.com/dotnet/csharp/language-reference/operators/patterns)
- [Pattern matching tutorial](https://learn.microsoft.com/dotnet/csharp/fundamentals/tutorials/pattern-matching)
- [Switch expression](https://learn.microsoft.com/dotnet/csharp/language-reference/operators/switch-expression)
