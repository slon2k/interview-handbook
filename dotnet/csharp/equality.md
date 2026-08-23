# Equality and Hashing

## Definition

Equality determines whether two values should be considered equivalent.

Hashing provides an efficient way to locate objects in hash-based collections such as:

```csharp
Dictionary<TKey, TValue>
HashSet<T>
ConcurrentDictionary<TKey, TValue>
```

Equality and hashing are closely related because hash-based collections rely on both.

In .NET, equality is primarily implemented through:

- `==`
- `!=`
- `Equals()`
- `IEquatable<T>`
- `GetHashCode()`

Understanding how these mechanisms work is essential for designing domain models, value objects, dictionary keys, and collection behavior.

## Alternatives & Trade-offs

### Reference Equality vs Value Equality

#### Reference Equality

Two variables are equal if they reference the same object.

```csharp
var p1 = new Person();
var p2 = p1;
```

```csharp
ReferenceEquals(p1, p2) == true
```

Pros:

- Simple
- Fast
- Natural for entities

Cons:

- Objects with identical data are not necessarily equal

---

#### Value Equality

Two objects are equal when their contents are equal.

```csharp
new Money(10) == new Money(10)
```

Pros:

- Better for value objects
- More intuitive for immutable data

Cons:

- Requires implementation effort
- May increase comparison cost

### Class vs Record Equality

Class:

```csharp
public class Person
{
    public string Name { get; set; }
}
```

Default behavior:

```csharp
Reference equality
```

Record:

```csharp
public record Person(string Name);
```

Default behavior:

```csharp
Value equality
```

Records automatically generate equality members.

### Equality Comparer vs GetHashCode Override

Sometimes equality logic should be type-wide:

```csharp
public override bool Equals(object? obj)
```

Sometimes it should be collection-specific:

```csharp
StringComparer.OrdinalIgnoreCase
```

or

```csharp
IEqualityComparer<T>
```

Custom comparers are often preferable when multiple equality rules exist.

## How It Works

### Object.Equals

Every type inherits:

```csharp
object.Equals()
```

Example:

```csharp
var a = "abc";
var b = "abc";

Console.WriteLine(a.Equals(b));
```

Output:

```text
True
```

---

### ReferenceEquals

Checks object identity.

```csharp
var a = new Person();
var b = new Person();

Console.WriteLine(
    ReferenceEquals(a, b));
```

Output:

```text
False
```

Even if all properties contain the same values.

---

### Equality Operator

The behavior of:

```csharp
==
```

depends on the type.

Example:

```csharp
string a = "abc";
string b = "abc";

Console.WriteLine(a == b);
```

Output:

```text
True
```

because `string` implements value equality.

Classes do not automatically behave this way.

---

### GetHashCode

A hash code is an integer used by hash-based collections.

Example:

```csharp
var value = "test";

Console.WriteLine(
    value.GetHashCode());
```

Hash codes are not unique.

Different objects may produce the same hash code.

This is called a collision.

---

### Equality Contract

A correct implementation should satisfy:

Reflexive:

```csharp
x.Equals(x) == true
```

Symmetric:

```csharp
x.Equals(y)
==
y.Equals(x)
```

Transitive:

```csharp
x == y
y == z
```

implies:

```csharp
x == z
```

Hashing contract:

```text
Equal objects must have equal hash codes.
```

The reverse is not required.

```text
Equal hash codes do not guarantee equality.
```

---

### Overriding Equals and GetHashCode

Example:

```csharp
public sealed class Money
{
    public decimal Amount { get; }

    public Money(decimal amount)
    {
        Amount = amount;
    }

    public override bool Equals(object? obj)
    {
        return obj is Money other
            && Amount == other.Amount;
    }

    public override int GetHashCode()
    {
        return Amount.GetHashCode();
    }
}
```

Whenever `Equals()` is overridden, `GetHashCode()` should also be overridden.

---

### IEquatable<T>

Strongly typed equality.

```csharp
public sealed class Money
    : IEquatable<Money>
{
}
```

Benefits:

- Better performance
- Avoids boxing
- Often used by collections

This is a frequent senior-level follow-up question.

### Record Equality

Records automatically generate:

- `Equals`
- `GetHashCode`
- `==`
- `!=`

Example:

```csharp
public record Money(decimal Amount);
```

```csharp
new Money(10)
==
new Money(10)
```

Result:

```text
True
```

## Application

### Value Objects

Good candidates:

```csharp
Money
Coordinate
Distance
Temperature
Percentage
```

These naturally use value equality.

---

### Dictionary Keys

```csharp
var users =
    new Dictionary<UserId, User>();
```

Key types should implement equality correctly.

---

### Hash Sets

```csharp
var tags =
    new HashSet<Tag>();
```

Uniqueness depends on equality and hashing behavior.

---

### Domain Design

Entities:

```csharp
Customer
Order
Invoice
```

typically rely on identity.

Value objects:

```csharp
Money
Address
Email
```

typically rely on value equality.

---

### Case-Insensitive Collections

```csharp
var users =
    new Dictionary<
        string,
        string>(
        StringComparer.OrdinalIgnoreCase);
```

Custom comparers often provide cleaner solutions than changing the type itself.

## Common Mistakes

### Overriding Equals Without GetHashCode

Incorrect:

```csharp
public override bool Equals(...)
{
}
```

without:

```csharp
public override int GetHashCode()
{
}
```

This can break dictionaries and hash sets.

---

### Using Mutable Properties in Hash Codes

Problematic:

```csharp
public string Name { get; set; }
```

If the value changes after insertion into a `HashSet`, lookups may stop working correctly.

---

### Assuming Equal Hash Codes Mean Equal Objects

Incorrect:

```text
hash1 == hash2
```

does not imply:

```text
object1 == object2
```

Collisions are expected.

---

### Forgetting String Equality Rules

Different comparison modes exist:

```csharp
Ordinal
OrdinalIgnoreCase
CurrentCulture
InvariantCulture
```

Using the wrong comparer can introduce bugs.

---

### Confusing Reference Equality and Value Equality

Many candidates incorrectly assume:

```csharp
new Person("Alice")
==
new Person("Alice")
```

returns:

```text
True
```

for ordinary classes.

By default it returns:

```text
False
```

unless equality is implemented.

---

### Using Complex Objects as Dictionary Keys

Objects used as keys should be:

- Logically immutable
- Consistent
- Have stable hash codes

## Common Interview Questions

### Basic

- What is equality?
- What is hashing?
- What is the purpose of `GetHashCode()`?
- What is the difference between `Equals()` and `==`?

### Intermediate

- Why must equal objects have equal hash codes?
- What is `ReferenceEquals()`?
- Why is `IEquatable<T>` useful?
- How do records implement equality?

### Follow-up Questions

- What happens if `Equals()` is overridden but `GetHashCode()` is not?
- Can two objects have the same hash code?
- Can two equal objects have different hash codes?
- Why can mutable dictionary keys be dangerous?
- What comparer would you use for case-insensitive string lookups?

### Code Prediction

What is the output?

```csharp
var a = new Person();
var b = new Person();

Console.WriteLine(
    ReferenceEquals(a, b));
```

---

What is the output?

```csharp
public record Money(decimal Amount);

Console.WriteLine(
    new Money(10)
    ==
    new Money(10));
```

---

What is the output?

```csharp
var set = new HashSet<int>();

set.Add(1);
set.Add(1);

Console.WriteLine(set.Count);
```

---

What is the result?

```csharp
string a = "ABC";
string b = "abc";

Console.WriteLine(
    a.Equals(
        b,
        StringComparison
            .OrdinalIgnoreCase));
```

---

Will this work correctly?

```csharp
public class User
{
    public override bool Equals(
        object? obj)
    {
        return true;
    }
}
```

Why or why not?

## Practical Tasks

### Implement Value Equality

Implement equality for:

```csharp
Money
```

Requirements:

- Equal amounts are equal
- Compatible hash codes
- Implement `IEquatable<Money>`

---

### Code Review

Review:

```csharp
public override bool Equals(
    object? obj)
{
    ...
}
```

without a `GetHashCode` override.

Explain potential issues.

---

### Dictionary Design

Design a suitable key type for:

```csharp
Dictionary<UserId, User>
```

What equality behavior should exist?

---

### Bug Investigation

A `HashSet<Product>` contains duplicates unexpectedly.

Investigate possible equality and hashing problems.

---

### Record Refactoring

Convert:

```csharp
public class Money
{
    public decimal Amount { get; }
}
```

to a record.

Discuss benefits and trade-offs.

## Readiness Criteria

You should be able to:

- Explain reference equality and value equality.
- Explain the relationship between equality and hashing.
- Understand `Equals`, `ReferenceEquals`, and `GetHashCode`.
- Explain why equal objects must have equal hash codes.
- Implement `IEquatable<T>` correctly.
- Design safe dictionary key types.
- Understand record equality behavior.
- Identify common equality-related bugs.
- Reason about hash-based collection behavior.
- Confidently answer common interview follow-up questions.

## References

### Microsoft Learn

- Object.Equals
- Object.GetHashCode
- IEquatable<T>
- Equality Comparisons
- Records
- HashSet<T>
- Dictionary<TKey, TValue>

### Additional Reading

- Framework Design Guidelines
- CLR via C#
- C# in Depth
- C# Language Specification
