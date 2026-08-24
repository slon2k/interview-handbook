# Strings and Immutability

## Definition

`string` is a reference type that represents an immutable sequence of UTF-16 characters.

```csharp
string first = "Ada";
string second = first;

first = "Grace";
```

The assignment creates a new reference to a different string value. It does not modify the original string object.

Because strings are immutable, operations such as concatenation and `Replace` return a new string.

## Alternatives & Trade-offs

### `string` vs `StringBuilder`

Use `string` for ordinary text and a small number of transformations. Use `StringBuilder` when repeatedly changing text in a loop or building a large result.

```csharp
var builder = new StringBuilder();
builder.Append("Hello");
builder.Append(' ');
builder.Append("world");
string result = builder.ToString();
```

`StringBuilder` can reduce temporary allocations, but it is not automatically faster for every operation.

### String Comparisons

Choose comparison rules deliberately:

- `StringComparison.Ordinal` for non-linguistic identifiers
- `StringComparison.OrdinalIgnoreCase` for case-insensitive keys
- Culture-aware comparisons for user-facing language-sensitive text

## How It Works

String literals may be interned so identical literals can share storage. Runtime-created strings are not necessarily interned.

```csharp
string a = "cat";
string b = "cat";
Console.WriteLine(object.ReferenceEquals(a, b));
```

`==` compares string contents, while `ReferenceEquals` compares object identity.

Interpolation and concatenation can allocate new strings. `Span<char>` and related APIs can help process text without creating intermediate strings, but the final string still requires an allocation when materialized.

## Application

- Use strings for immutable text values and protocol data.
- Use ordinal comparison for identifiers, tokens, file names where appropriate, and security-sensitive values.
- Normalize or validate input at boundaries.
- Use `StringBuilder` for repeated appends.
- Use `ReadOnlySpan<char>` for allocation-sensitive parsing paths.

## Common Mistakes

- Treating a string as a mutable character array.
- Using culture-sensitive comparison for a machine identifier.
- Using `ToLower()` before comparison instead of an explicit comparison option.
- Assuming every concatenation is expensive without measuring the actual code.
- Building SQL, HTML, or shell commands by concatenating untrusted input.
- Forgetting that `null` and `string.Empty` represent different states.

## Common Interview Questions

### Basic

- Why is `string` immutable in C#?
- Is `string` a value type or a reference type?
- What does string interning mean?
- What is the difference between `string.Empty` and `null`?

### Intermediate

- How does `==` work for strings?
- When should you use `StringBuilder`?
- What is the difference between ordinal and culture-aware comparison?
- Why can repeated concatenation allocate many objects?

### Advanced

- How does string interning affect identity and memory usage?
- How should comparison rules be selected for security-sensitive values?
- How do UTF-16 code units differ from Unicode scalar values?
- When can `ReadOnlySpan<char>` improve text-processing performance?
- What allocations can string interpolation introduce?
- How would you design a case-insensitive dictionary for string keys?
- How can normalization affect equality and security checks?
- What are the trade-offs of pooling or caching strings?
- How would you benchmark `string` concatenation against `StringBuilder`?
- How can immutable strings simplify concurrent code?

### Follow-up Questions

- Does `ReferenceEquals` prove two strings have equal content?
- What does `StringComparison.OrdinalIgnoreCase` guarantee?
- When is `StringBuilder` unnecessary?
- What is the difference between a character, a UTF-16 code unit, and a Unicode scalar value?
- How can untrusted text be safely placed into an output format?

### Code Prediction

What is printed?

```csharp
string value = "A";
string copy = value;
value += "B";
Console.WriteLine(copy);
```

What is printed?

```csharp
string left = new string('x', 1);
string right = new string('x', 1);
Console.WriteLine(left == right);
Console.WriteLine(object.ReferenceEquals(left, right));
```

## Practical Tasks

### Comparison Design

Choose the correct `StringComparison` option for a user name, an HTTP header, and a database key. Explain each choice.

### Allocation Review

Compare a loop using `result += value` with one using `StringBuilder`. Measure allocations and runtime rather than relying on assumptions.

### Input Validation

Design a parser that distinguishes `null`, empty, whitespace-only, and valid input.

## Readiness Criteria

You should be able to explain string immutability, interning, comparison rules, and allocation behavior. You should also know when to use `StringBuilder` or span-based processing and how to handle untrusted text safely.

## References

### Microsoft Learn

- [The `string` type](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/reference-types#the-string-type)
- [String comparison](https://learn.microsoft.com/dotnet/standard/base-types/best-practices-strings)
- [StringBuilder class](https://learn.microsoft.com/dotnet/api/system.text.stringbuilder)
- [String interning](https://learn.microsoft.com/dotnet/fundamentals/runtime-libraries/system-string-intern)
