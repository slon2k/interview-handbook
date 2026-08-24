# Modern C# Syntax

## Definition

Modern C# adds concise syntax and stronger contracts for common programming patterns. This topic covers init-only properties, required members, using declarations, collection expressions, file-scoped namespaces, global usings, raw string literals, and spans.

```csharp
public sealed class User
{
    public required string Name { get; init; }
}

using StreamReader reader = File.OpenText(path);
int[] values = [1, 2, 3];
```

## Alternatives & Trade-offs

Modern syntax can reduce ceremony and express intent, but language version and target framework support must be considered. Prefer the clearest construct for the team and public API rather than adopting syntax solely because it is newer.

`init` and `required` improve construction contracts. `Span<T>` improves allocation-sensitive processing but introduces stack and lifetime restrictions. Collection expressions improve readability but do not guarantee a particular allocation strategy for every target type.

## How It Works

`init` accessors can be assigned during object initialization or construction but not afterward. `required` produces compile-time diagnostics when required members are not initialized.

Using declarations are lowered to `try/finally` disposal logic. Collection expressions are converted according to the target type. `Span<T>` and `ReadOnlySpan<T>` are stack-only ref structs that provide views over contiguous memory.

```csharp
ReadOnlySpan<char> slice = text.AsSpan(0, 3);
```

Raw string literals reduce escaping for JSON, regular expressions, and other structured text. File-scoped namespaces and global usings affect source organization rather than runtime behavior.

## Application

- Use `init` for immutable-after-construction object state.
- Use `required` when callers must provide a member.
- Use using declarations for deterministic resource cleanup with local scope.
- Use collection expressions for readable collection initialization.
- Use spans for measured, allocation-sensitive parsing and formatting.
- Use raw strings for embedded structured text.

## Common Mistakes

- Treating `required` as runtime validation.
- Assuming `init` makes referenced objects deeply immutable.
- Extending a disposable resource beyond the using declaration's scope.
- Using spans in fields, async methods, or closures where their lifetime is invalid.
- Assuming collection expressions always allocate minimally.
- Mixing language features without checking target compiler and runtime support.

## Common Interview Questions

### Basic

- What is an init-only property?
- What does `required` do?
- What is a using declaration?
- What is a collection expression?

### Intermediate

- How does `init` differ from a private setter?
- When does a required-member warning occur?
- What is a `Span<T>`?
- What are file-scoped namespaces and global usings?

### Advanced

- How are init-only setters enforced by the compiler and metadata?
- What limitations do `ref struct` types impose on async and iterator methods?
- How do collection expressions select their target representation?
- How can spans reduce allocations without copying data?
- What are the versioning risks of adding required members to a public type?
- How do raw string literal delimiter counts work?
- How does a using declaration lower into disposal control flow?
- How should modern syntax be evaluated for AOT and trimming scenarios?
- What are the trade-offs between source readability and language feature density?
- How would you migrate older syntax incrementally without changing behavior?

### Follow-up Questions

- Can a required member have a default value?
- Can an init-only property be assigned from a constructor?
- Can `Span<T>` be stored in a class field?
- Does a using declaration dispose at the end of the block?
- What determines the type of a collection expression?

### Code Prediction

What happens after initialization?

```csharp
var user = new User { Name = "Ada" };
// user.Name = "Grace"; // Does not compile when Name uses init.
```

## Practical Tasks

### Construction Contract

Define a type with required init-only members and test the compiler diagnostics for incomplete initialization.

### Resource Scope

Convert nested using blocks to using declarations while preserving disposal order.

### Span Parsing

Implement a small parser using `ReadOnlySpan<char>` and compare allocations with a substring-based implementation.

## Readiness Criteria

You should be able to explain the compile-time and runtime behavior of the covered features, choose them based on clarity and constraints, and identify where spans, disposal, or version compatibility affect design.

## References

### Microsoft Learn

- [Properties](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/properties)
- [Required members](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/required)
- [Using statement](https://learn.microsoft.com/dotnet/csharp/language-reference/statements/using)
- [Collection expressions](https://learn.microsoft.com/dotnet/csharp/language-reference/operators/collection-expressions)
- [Introduction to `Span<T>`](https://learn.microsoft.com/dotnet/fundamentals/runtime-libraries/system-span%7Bt%7D)
- [Raw string literals](https://learn.microsoft.com/dotnet/csharp/language-reference/tokens/raw-string)
