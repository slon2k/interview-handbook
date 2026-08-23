# Nullable Value Types

## Definition

Value types cannot normally contain `null`.

For example:

```csharp
int age = null;
```

This does not compile.

Nullable value types allow value types to represent both:

- A valid value
- No value (`null`)

Syntax:

```csharp
int?
decimal?
bool?
DateTime?
Guid?
```

Example:

```csharp
int? age = null;
```

The `?` syntax is shorthand for:

```csharp
Nullable<int>
```

The following declarations are equivalent:

```csharp
int? age = null;
```

```csharp
Nullable<int> age = null;
```

Nullable value types are commonly used when the absence of a value is meaningful.

Examples:

- Optional database fields
- Search filters
- External API requests
- User input
- Domain values that may be unknown

## Alternatives & Trade-offs

### Value Type vs Nullable Value Type

#### Value Type

```csharp
int age;
```

Pros:

- Simpler
- Always contains a valid value
- No null handling required

Cons:

- Cannot represent "unknown" or "not provided"

#### Nullable Value Type

```csharp
int? age;
```

Pros:

- Represents missing values
- Maps naturally to database NULLs
- Useful for optional parameters and filters

Cons:

- Additional null handling required
- Can introduce null-related bugs if ignored

---

### Sentinel Values vs Nullable Values

Some code uses special values:

```csharp
int age = -1;
```

to represent "unknown".

This approach is often less expressive than:

```csharp
int? age = null;
```

Benefits of nullable values:

- Clearer intent
- Easier validation
- Fewer magic numbers
- Better API design

## How It Works

### Nullable<T>

Nullable value types are implemented using:

```csharp
Nullable<T>
```

Example:

```csharp
Nullable<int> age = null;
```

Equivalent to:

```csharp
int? age = null;
```

---

### Assigning Values

```csharp
int? age = 25;
```

```csharp
int? age = null;
```

Both are valid.

---

### HasValue

Use `HasValue` to determine whether a value exists.

```csharp
int? age = null;

Console.WriteLine(age.HasValue);
```

Output:

```text
False
```

Example:

```csharp
if (age.HasValue)
{
    Console.WriteLine(age.Value);
}
```

---

### Value

The `Value` property returns the underlying value.

```csharp
int? age = 25;

Console.WriteLine(age.Value);
```

Output:

```text
25
```

However:

```csharp
int? age = null;

Console.WriteLine(age.Value);
```

Throws:

```text
InvalidOperationException
```

---

### Null-Coalescing Operator

Use `??` to provide a default value.

```csharp
int? age = null;

int result = age ?? 0;
```

Result:

```text
0
```

A very common interview topic.

---

### Null-Coalescing Assignment

```csharp
age ??= 18;
```

Equivalent to:

```csharp
if (age == null)
{
    age = 18;
}
```

---

### GetValueOrDefault

```csharp
int? age = null;

Console.WriteLine(age.GetValueOrDefault());
```

Output:

```text
0
```

Custom default:

```csharp
age.GetValueOrDefault(18);
```

---

### Nullable Arithmetic

```csharp
int? a = 10;
int? b = null;

var result = a + b;
```

Result:

```text
null
```

Most arithmetic operators propagate null values.

---

### Boxing Behavior

```csharp
int? value = 10;

object boxed = value;
```

The underlying value is boxed.

If the nullable value is null:

```csharp
int? value = null;

object boxed = value;
```

Result:

```text
boxed == null
```

This is a common follow-up interview question.

## Application

### Database Models

Database values often permit NULL.

```csharp
public DateTime? BirthDate { get; set; }
```

---

### Search Filters

```csharp
public class UserSearchRequest
{
    public int? MinAge { get; set; }
}
```

A missing filter can be represented as null.

---

### Optional Domain Values

```csharp
public decimal? Bonus { get; set; }
```

A bonus may not exist.

---

### API Contracts

```csharp
public record UpdateUserRequest(
    string Name,
    int? Age);
```

The consumer may omit a value.

---

### Calculations With Optional Values

```csharp
decimal total = discount ?? 0;
```

Provides a safe fallback when data is unavailable.

## Common Mistakes

### Accessing Value Without Checking

Unsafe:

```csharp
int? age = null;

Console.WriteLine(age.Value);
```

Results in:

```text
InvalidOperationException
```

---

### Forgetting Null Handling

Problematic:

```csharp
int? age = GetAge();

var result = age + 10;
```

The result may become null.

---

### Using Magic Numbers Instead of Nullable Values

Poor:

```csharp
int age = -1;
```

Preferred:

```csharp
int? age = null;
```

---

### Confusing Nullable Value Types and Nullable Reference Types

These are different features.

Nullable value type:

```csharp
int?
```

Nullable reference type:

```csharp
string?
```

They solve different problems.

---

### Assuming HasValue Checks the Numeric Value

```csharp
int? value = 0;
```

```csharp
value.HasValue == true
```

because a value exists.

Zero and null are different concepts.

## Common Interview Questions

### Basic

- What is a nullable value type?
- Why do nullable value types exist?
- What does `int?` mean?
- What is `Nullable<T>`?

### Intermediate

- What is the difference between `Value` and `HasValue`?
- What does `??` do?
- What does `GetValueOrDefault()` do?
- How are nullable value types represented internally?

### Follow-up Questions

- What exception can `Value` throw?
- What happens when nullable values participate in arithmetic?
- How does boxing work for nullable value types?
- What is the difference between `int?` and `string?`?
- Why are nullable value types common in EF Core entities?

### Code Prediction

What is the output?

```csharp
int? age = null;

Console.WriteLine(age.HasValue);
```

---

What is the output?

```csharp
int? age = null;

Console.WriteLine(age ?? 18);
```

---

What happens?

```csharp
int? age = null;

Console.WriteLine(age.Value);
```

---

What is the result?

```csharp
int? a = 10;
int? b = null;

Console.WriteLine(a + b);
```

---

What is printed?

```csharp
int? value = null;

object boxed = value;

Console.WriteLine(boxed == null);
```

## Practical Tasks

### Refactoring

Replace:

```csharp
public int Age = -1;
```

with a nullable design.

Explain the benefits.

---

### API Design

Design a search request:

```text
Minimum age
Maximum age
City
```

Use nullable values where appropriate.

---

### Code Review

Review:

```csharp
public DateTime BirthDate { get; set; }
```

A user may choose not to provide a birth date.

Would you change the design?

---

### Bug Investigation

Find the issue:

```csharp
int? amount = null;

var total = amount.Value * 2;
```

---

### Database Mapping

Design a model for:

```text
Middle name
Date of resignation
Bonus amount
```

Determine which properties should be nullable.

## Readiness Criteria

You should be able to:

- Explain why nullable value types exist.
- Explain the relationship between `int?` and `Nullable<int>`.
- Use `HasValue`, `Value`, and `GetValueOrDefault()`.
- Use the null-coalescing operator correctly.
- Understand nullable arithmetic behavior.
- Explain boxing behavior.
- Distinguish nullable value types from nullable reference types.
- Design APIs and models that correctly represent missing values.
- Recognize common null-handling mistakes.
- Confidently answer common interview follow-up questions.

## References

### Microsoft Learn

- Nullable Value Types
- Nullable<T>
- Null-Coalescing Operators
- Boxing and Unboxing

### Additional Reading

- C# Language Specification
- C# in Depth
- CLR via C#
- Framework Design Guidelines
