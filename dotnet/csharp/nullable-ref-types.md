# Nullable Reference Types

## Definition

Prior to C# 8, all reference types could contain `null`.

```csharp
string name = null;
```

This was completely valid.

As a result, many applications suffered from:

```text
NullReferenceException
```

at runtime.

Nullable Reference Types (NRTs) were introduced in C# 8 to help developers detect potential null-related bugs during compilation.

With nullable reference types enabled, reference types are divided into two categories:

- Non-nullable reference types
- Nullable reference types

Examples:

```csharp
string name;
```

```csharp
string? middleName;
```

A non-nullable reference type:

```csharp
string
```

indicates:

```text
This value should never be null.
```

A nullable reference type:

```csharp
string?
```

indicates:

```text
This value may be null.
```

Unlike nullable value types, nullable reference types do **not** change runtime behavior.

They are primarily a compiler-assisted static analysis feature.

## Alternatives & Trade-offs

### Non-Nullable Reference Type

```csharp
string Name
```

Pros:

- Expresses required data
- Reduces null-related bugs
- Provides stronger compile-time guarantees
- Self-documenting

Cons:

- Requires proper initialization
- May require additional annotations

Use when null should never be a valid state.

---

### Nullable Reference Type

```csharp
string? MiddleName
```

Pros:

- Explicitly models optional data
- Expresses intent clearly
- Works well for APIs and DTOs

Cons:

- Requires null handling
- Can lead to additional checks

Use when absence of a value is meaningful.

---

### Nullable Reference Type vs Empty String

Some code uses:

```csharp
string Name = "";
```

to represent a missing value.

Compare:

```csharp
string Name = "";
```

vs:

```csharp
string? Name = null;
```

These represent different concepts:

```text
""   -> value exists but is empty
null -> value is absent
```

Choose intentionally.

## How It Works

### Enabling Nullable Reference Types

Project-wide:

```xml
<Nullable>enable</Nullable>
```

Example:

```xml
<PropertyGroup>
    <Nullable>enable</Nullable>
</PropertyGroup>
```

This is the default for most modern .NET projects.

---

### Non-Nullable References

```csharp
string name = "Alice";
```

The compiler assumes `name` should never be null.

This generates a warning:

```csharp
string name = null;
```

Example warning:

```text
CS8625
Cannot convert null literal
to non-nullable reference type.
```

---

### Nullable References

```csharp
string? middleName = null;
```

Valid:

```csharp
middleName = null;
middleName = "John";
```

---

### Flow Analysis

The compiler tracks nullability.

Example:

```csharp
string? name = GetName();

Console.WriteLine(name.Length);
```

Produces a warning.

After checking:

```csharp
if (name != null)
{
    Console.WriteLine(name.Length);
}
```

The warning disappears.

The compiler knows the value cannot be null inside the block.

---

### Null-Conditional Operator

```csharp
string? name = GetName();

var length = name?.Length;
```

If `name` is null:

```text
length = null
```

No exception occurs.

---

### Null-Coalescing Operator

```csharp
string? name = GetName();

string displayName = name ?? "Unknown";
```

Provides a fallback value.

---

### Null-Forgiving Operator

```csharp
string? name = GetName();

Console.WriteLine(name!.Length);
```

The `!` operator tells the compiler:

```text
Trust me, this value is not null.
```

It suppresses warnings.

It does **not** perform a runtime check.

If the value is actually null:

```text
NullReferenceException
```

can still occur.

## Application

### Domain Models

Required data:

```csharp
public class User
{
    public string Name { get; set; } = string.Empty;
}
```

Optional data:

```csharp
public string? MiddleName { get; set; }
```

---

### DTOs

Request data often contains optional properties.

```csharp
public record UpdateUserRequest
{
    public string? Email { get; init; }
}
```

---

### Configuration

Some configuration values are optional.

```csharp
public string? ConnectionString { get; init; }
```

---

### API Design

Nullable annotations communicate intent.

```csharp
User? FindById(int id)
```

This makes it clear that no user may be found.

---

### Defensive Programming

Compiler warnings help identify:

- Missing validation
- Null dereferences
- Unsafe property access

before code reaches production.

## Common Mistakes

### Confusing Nullable Reference Types with Nullable Value Types

Nullable value type:

```csharp
int?
```

Nullable reference type:

```csharp
string?
```

These are different language features.

---

### Assuming Runtime Behavior Changes

Incorrect:

```text
string? automatically prevents NullReferenceException
```

False.

Nullable reference types only provide compiler analysis.

Runtime behavior remains unchanged.

---

### Overusing the Null-Forgiving Operator

Problematic:

```csharp
user!.Manager!.Name!
```

This suppresses warnings without solving the underlying issue.

Use sparingly.

---

### Making Everything Nullable

Poor:

```csharp
public string? Name { get; set; }
public string? Email { get; set; }
public string? Address { get; set; }
```

If values are required, make them non-nullable.

---

### Ignoring Compiler Warnings

The compiler often identifies potential bugs.

Treat warnings seriously rather than suppressing them.

---

### Using Empty Strings and Null Interchangeably

```text
"" != null
```

These represent different states.

Model them intentionally.

## Common Interview Questions

### Basic

- What are nullable reference types?
- Why were nullable reference types introduced?
- What does `string?` mean?
- What is the difference between `string` and `string?`?

### Intermediate

- How are nullable reference types enabled?
- What is compiler flow analysis?
- What is the purpose of the null-forgiving operator?
- What does the null-conditional operator do?

### Follow-up Questions

- Do nullable reference types affect runtime behavior?
- Can `NullReferenceException` still occur?
- When should a property be nullable?
- Why is `string.Empty` different from `null`?
- What should be returned if a method may not find a result?

### Code Prediction

Will this generate a warning?

```csharp
string name = null;
```

---

Will this generate a warning?

```csharp
string? name = null;
```

---

What happens?

```csharp
string? name = GetName();

Console.WriteLine(name.Length);
```

---

What is the result?

```csharp
string? name = null;

Console.WriteLine(name?.Length);
```

---

What does this do?

```csharp
name!.Length
```

Does it guarantee safety?

## Practical Tasks

### Refactoring

Review:

```csharp
public class User
{
    public string Name { get; set; }
    public string Email { get; set; }
}
```

Determine:

- Which properties should be nullable?
- Which properties should be non-nullable?

---

### API Design

Design:

```csharp
User FindById(int id)
```

Should the return type be:

```csharp
User
```

or

```csharp
User?
```

Explain your reasoning.

---

### Code Review

Review:

```csharp
public string? Email { get; set; }
```

The business rules state that every user must have an email address.

Would you change the design?

---

### Bug Investigation

Find the issue:

```csharp
string? name = GetName();

return name.Length;
```

---

### Domain Modeling

Determine which properties should be nullable:

```text
First Name
Middle Name
Date of Death
Email Address
National Identifier
```

Explain your choices.

## Readiness Criteria

You should be able to:

- Explain why nullable reference types were introduced.
- Distinguish nullable and non-nullable reference types.
- Explain the purpose of compiler nullability analysis.
- Use `?`, `?.`, `??`, and `!` correctly.
- Understand the limitations of nullable reference types.
- Recognize situations where null is a valid state.
- Design APIs that communicate nullability clearly.
- Interpret and resolve common nullability warnings.
- Avoid common misuse of the null-forgiving operator.
- Confidently answer common interview follow-up questions.

## References

### Microsoft Learn

- Nullable Reference Types
- Nullable Contexts
- Nullability Warnings
- Null-Conditional Operators
- Null-Coalescing Operators

### Additional Reading

- C# Language Specification
- C# in Depth
- Framework Design Guidelines
- Nullable Reference Types Design Notes
