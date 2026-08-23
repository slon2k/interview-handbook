# Built-in Numeric Types and Conversions

## Definition

C# provides a set of built-in numeric types for representing whole numbers, floating-point numbers, and high-precision decimal values.

### Integral Types

| Type | Size | Range |
|--------|--------|--------|
| `sbyte` | 8-bit | -128 to 127 |
| `byte` | 8-bit | 0 to 255 |
| `short` | 16-bit | -32,768 to 32,767 |
| `ushort` | 16-bit | 0 to 65,535 |
| `int` | 32-bit | ±2.1 billion |
| `uint` | 32-bit | 0 to 4.2 billion |
| `long` | 64-bit | Very large |
| `ulong` | 64-bit | Very large positive |

### Floating-Point Types

| Type | Size | Typical Usage |
|--------|--------|--------|
| `float` | 32-bit | Approximate values |
| `double` | 64-bit | Default floating-point type |
| `decimal` | 128-bit | Financial calculations |

### Special Types

```csharp
char
nint
nuint
```

In practice, the most commonly used numeric types are:

```csharp
int
long
double
decimal
```

Understanding numeric types and conversions is important for preventing data loss, overflow, precision issues, and unexpected behavior.

## Alternatives & Trade-offs

### int vs long

#### int

Pros:

- Smaller memory footprint
- Faster in some scenarios
- Most APIs use it by default

Cons:

- Limited range

Use when values comfortably fit within a 32-bit range.

#### long

Pros:

- Much larger range

Cons:

- Larger memory footprint

Use for identifiers, timestamps, counters, or values that may exceed the range of `int`.

---

### float vs double

#### float

Pros:

- Lower memory usage

Cons:

- Less precision

#### double

Pros:

- Higher precision
- Default floating-point type in .NET

Cons:

- Larger memory usage

Use `double` unless there is a specific reason to choose `float`.

---

### double vs decimal

#### double

Pros:

- Faster arithmetic
- Smaller memory footprint

Cons:

- Binary representation causes precision issues

Example:

```csharp
Console.WriteLine(0.1 + 0.2);
```

Output:

```text
0.30000000000000004
```

#### decimal

Pros:

- Higher decimal precision
- Preferred for financial calculations

Cons:

- Slower arithmetic
- Larger memory footprint

Example:

```csharp
decimal price = 10.99m;
```

Use `decimal` for money and other values where precision is critical.

## How It Works

### Numeric Literals

```csharp
int number = 42;
long total = 42L;
float ratio = 1.5F;
double value = 1.5;
decimal amount = 1.5M;
```

Literal suffixes determine the resulting type.

---

### Implicit Conversions

Safe conversions occur automatically.

```csharp
int value = 100;
long total = value;
```

No data loss is possible.

---

### Explicit Conversions

Potentially unsafe conversions require a cast.

```csharp
long total = 1000;
int value = (int)total;
```

The compiler requires acknowledgement that data may be lost.

---

### Overflow

Overflow occurs when a value exceeds the target type's range.

```csharp
int max = int.MaxValue;
max++;
```

In unchecked contexts this wraps around.

Result:

```text
-2147483648
```

---

### checked and unchecked

```csharp
checked
{
    int value = int.MaxValue;
    value++;
}
```

Throws:

```text
OverflowException
```

Unchecked behavior is the default for most runtime operations.

---

### Type Promotion

Arithmetic operations may promote operands.

```csharp
byte a = 10;
byte b = 20;

var result = a + b;
```

`result` becomes:

```csharp
int
```

not:

```csharp
byte
```

This is a common interview question.

## Application

### Counters

```csharp
int processedItems = 0;
```

### Large Identifiers

```csharp
long fileSize = 4_000_000_000;
```

### Measurements

```csharp
double temperature = 22.5;
```

### Financial Data

```csharp
decimal totalPrice = 99.99m;
```

### Scientific Calculations

```csharp
double gravity = 9.81;
```

Selecting the wrong numeric type can introduce bugs, precision loss, or unexpected overflow.

## Common Mistakes

### Using double for Money

Incorrect:

```csharp
double price = 10.99;
```

Preferred:

```csharp
decimal price = 10.99m;
```

---

### Ignoring Overflow

```csharp
int total = int.MaxValue;
total++;
```

Many candidates forget that this does not necessarily throw an exception.

---

### Assuming Casting Is Always Safe

```csharp
long large = 5_000_000_000;
int small = (int)large;
```

Data is lost.

---

### Forgetting Numeric Literal Suffixes

Incorrect:

```csharp
decimal amount = 10.5;
```

Correct:

```csharp
decimal amount = 10.5m;
```

---

### Assuming Floating-Point Numbers Are Exact

Incorrect expectation:

```csharp
0.1 + 0.2 == 0.3
```

Floating-point arithmetic may produce surprising results.

---

### Forgetting Arithmetic Promotions

Many developers expect:

```csharp
byte + byte
```

to produce:

```csharp
byte
```

but the result is:

```csharp
int
```

## Common Interview Questions

### Basic

- What numeric types are available in C#?
- What is the difference between `int` and `long`?
- What is the difference between `double` and `decimal`?
- When would you use `decimal`?

### Intermediate

- What is an implicit conversion?
- What is an explicit conversion?
- What is overflow?
- What is the purpose of `checked`?
- Why does `byte + byte` produce an `int`?

### Follow-up Questions

- Why is floating-point arithmetic imprecise?
- Is `decimal` faster than `double`?
- What happens when data is lost during conversion?
- Does `checked` affect performance?
- Why does C# require casts for some conversions but not others?

### Code Prediction

What is the output?

```csharp
Console.WriteLine(0.1 + 0.2);
```

Why?

---

What is the value of `result`?

```csharp
byte a = 10;
byte b = 20;

var result = a + b;
```

What is the type of `result`?

---

What happens?

```csharp
checked
{
    int value = int.MaxValue;
    value++;
}
```

---

What is printed?

```csharp
int value = (int)1.9;

Console.WriteLine(value);
```

## Practical Tasks

### Type Selection

Choose the most appropriate type for:

- User age
- Bank account balance
- GPS coordinate
- Number of processed records
- File size in bytes

Explain your choice.

---

### Conversion Review

Review:

```csharp
long total = GetValue();
int count = (int)total;
```

Identify potential issues.

---

### Overflow Investigation

Find and fix the problem:

```csharp
int totalUsers = int.MaxValue;
totalUsers++;
```

---

### Financial Calculation

Implement:

```csharp
decimal CalculateTotal(
    decimal price,
    decimal taxRate)
```

and explain why `decimal` is preferable to `double`.

---

### Code Review

Review:

```csharp
double amount = 100.10;
double tax = 0.15;

var total = amount + tax;
```

Would you keep `double` here?

## Readiness Criteria

You should be able to:

- Explain the purpose of all commonly used numeric types.
- Explain the differences between `int`, `long`, `double`, and `decimal`.
- Distinguish between implicit and explicit conversions.
- Explain overflow and the purpose of `checked`.
- Predict the result of common conversion operations.
- Recognize precision problems involving floating-point values.
- Choose appropriate numeric types for real-world scenarios.
- Answer common interview follow-up questions confidently.

## References

### Microsoft Learn

- Integral numeric types
- Floating-point numeric types
- Numeric conversions
- checked and unchecked statements
- Arithmetic operators

### Additional Reading

- C# Language Specification
- CLR via C#
- C# in Depth
