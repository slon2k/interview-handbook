# Classes, Structs, Records and Record Structs

## Definition

C# provides four primary ways to define custom data types:

- `class`
- `struct`
- `record`
- `record struct`

The choice affects:

- Value vs reference semantics
- Equality behavior
- Mutability
- Copying behavior
- Intended usage

Understanding the differences is important for API design, domain modelling, performance considerations, and interview discussions.

### Class

A class is a reference type.

```csharp
public class Customer
{
    public string Name { get; set; } = string.Empty;
}
```

Classes are the most commonly used type in C# applications.

### Struct

A struct is a value type.

```csharp
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
}
```

Structs are intended for small value-like data.

### Record

A record is a reference type with built-in value-based equality.

```csharp
public record Customer(string Name);
```

Records were introduced to simplify immutable data models and value objects.

### Record Struct

A record struct is a value type with built-in value-based equality.

```csharp
public record struct Point(int X, int Y);
```

It combines value semantics with record features.

## Alternatives & Trade-offs

### Class vs Struct

| Aspect | Class | Struct |
|----------|----------|----------|
| Category | Reference type | Value type |
| Assignment | Copies reference | Copies value |
| Default equality | Reference equality | Value equality |
| Supports inheritance | Yes | No |
| Nullable by default | Yes | No |
| Typical usage | Domain models, services | Small value objects |

Use a class when:

- Identity matters
- Shared state is required
- The object is large
- Polymorphism is needed

Use a struct when:

- The type is small
- Value semantics are desired
- The type is immutable

---

### Class vs Record

Both are reference types.

Class:

```csharp
public class Customer
{
    public string Name { get; set; }
}
```

Record:

```csharp
public record Customer(string Name);
```

Records automatically provide:

- Value-based equality
- Hash code generation
- Useful `ToString()`
- `with` expressions

Use records when the data itself matters more than object identity.

---

### Struct vs Record Struct

Both are value types.

Struct:

```csharp
public struct Money
{
    public decimal Amount { get; set; }
}
```

Record struct:

```csharp
public record struct Money(decimal Amount);
```

Record structs automatically provide:

- Value-based equality
- Hash code generation
- Improved diagnostics

Use a record struct when value semantics and equality are important.

---

### Record vs Record Struct

| Aspect | Record | Record Struct |
|----------|----------|----------|
| Category | Reference type | Value type |
| Assignment | Copies reference | Copies value |
| Equality | Value-based | Value-based |
| Allocation | Typically reference object | Value type |

Choose based on semantics rather than syntax.

## How It Works

### Class Assignment

Assignment copies the reference.

```csharp
var c1 = new Customer
{
    Name = "Alice"
};

var c2 = c1;

c2.Name = "Bob";
```

Result:

```text
c1.Name = "Bob"
c2.Name = "Bob"
```

Both variables reference the same object.

---

### Struct Assignment

Assignment copies the value.

```csharp
Point p1 = new(1, 1);
Point p2 = p1;

p2.X = 100;
```

Result:

```text
p1.X = 1
p2.X = 100
```

Each variable contains its own copy.

---

### Default Equality

Classes use reference equality by default.

```csharp
var p1 = new Person();
var p2 = new Person();

Console.WriteLine(p1 == p2);
```

Output:

```text
False
```

Even if all property values are identical.

---

### Record Equality

Records use value-based equality.

```csharp
var p1 = new Person("Alice");
var p2 = new Person("Alice");

Console.WriteLine(p1 == p2);
```

Output:

```text
True
```

The generated equality members compare values rather than references.

---

### With Expressions

Records support nondestructive mutation.

```csharp
var original = new Person("Alice");

var modified = original with
{
    Name = "Bob"
};
```

A new instance is created while preserving the original.

---

### Inheritance

Classes support inheritance.

```csharp
public class Animal
{
}

public class Dog : Animal
{
}
```

Records support inheritance.

```csharp
public record Animal(string Name);

public record Dog(string Name)
    : Animal(Name);
```

Structs and record structs do not support inheritance.

## Application

### Class

Good candidates:

```csharp
Customer
Order
Invoice
Product
User
```

Characteristics:

- Identity matters
- State changes over time
- Shared references are expected

---

### Struct

Good candidates:

```csharp
Point
Coordinate
Temperature
Money
Distance
```

Characteristics:

- Small data
- Value semantics
- Immutable design

---

### Record

Good candidates:

```csharp
CustomerDto
CreateOrderRequest
CreateOrderResponse
DomainEvent
MessageContract
```

Characteristics:

- Data transfer
- Immutable data
- Value equality

---

### Record Struct

Good candidates:

```csharp
Money
Percentage
Identifier
Coordinate
```

Characteristics:

- Small immutable value objects
- Frequent equality comparisons

## Common Mistakes

### Assuming Records Are Immutable

Records are not automatically immutable.

This is mutable:

```csharp
public record User
{
    public string Name { get; set; } = string.Empty;
}
```

Immutability depends on design.

---

### Using Large Structs

Large structs can create expensive copies.

Poor candidate:

```csharp
struct LargeReportData
{
    // dozens of fields
}
```

A class is often more appropriate.

---

### Creating Mutable Structs

Problematic:

```csharp
public struct Point
{
    public int X { get; set; }
    public int Y { get; set; }
}
```

Mutable structs frequently cause subtle bugs.

---

### Choosing Structs for Performance Without Measurement

Many candidates assume:

```text
struct = faster
```

This is not always true.

Performance depends on:

- Size
- Allocation patterns
- Copy frequency
- Usage patterns

Measure before optimizing.

---

### Assuming Classes Compare Values

Many developers expect:

```csharp
customer1 == customer2
```

to compare properties.

By default, classes compare references.

---

### Using Records for Entities Without Understanding Identity

Entities often have identity.

```csharp
Customer
Order
User
```

Records are usually a better fit for value objects and DTOs than database entities.

## Common Interview Questions

### Basic

- What is the difference between a class and a struct?
- What is a record?
- What is a record struct?
- Which of these are value types?
- Which of these are reference types?

### Intermediate

- Why were records introduced?
- When should you use a struct?
- What equality behavior does a record provide?
- Why are mutable structs discouraged?
- When would you choose a record over a class?

### Follow-up Questions

- Are records immutable?
- Can records contain methods?
- Can records participate in inheritance?
- Can structs have constructors?
- Why can large structs become problematic?
- What is the purpose of the `with` expression?

### Code Prediction

What is the output?

```csharp
var p1 = new Person("Alice");
var p2 = new Person("Alice");

Console.WriteLine(p1 == p2);
```

Assume `Person` is a record.

---

What is the output?

```csharp
Point p1 = new(1, 1);
Point p2 = p1;

p2.X = 5;

Console.WriteLine(p1.X);
```

Assume `Point` is a struct.

---

What is the output?

```csharp
var c1 = new Customer
{
    Name = "Alice"
};

var c2 = c1;

c2.Name = "Bob";

Console.WriteLine(c1.Name);
```

---

Should this type be:

- class
- struct
- record
- record struct

```csharp
Money
```

Explain your reasoning.

## Practical Tasks

### Design Discussion

Choose the most appropriate type for:

- Customer
- Temperature
- Money
- OrderDto
- GeographicCoordinate

Explain the trade-offs.

---

### Refactoring

Convert:

```csharp
public class CreateOrderRequest
{
    public string ProductId { get; init; }
    public int Quantity { get; init; }
}
```

to a record.

Discuss the benefits.

---

### Code Review

Review:

```csharp
public struct Customer
{
    public string Name { get; set; }
    public string Address { get; set; }
    public string Phone { get; set; }
}
```

Would you keep it as a struct?

Why or why not?

---

### Bug Investigation

A mutable struct behaves unexpectedly after being passed between methods.

Explain possible causes.

---

### API Design

Design a `Money` type and justify:

- Class vs struct
- Record vs non-record
- Mutable vs immutable

## Readiness Criteria

You should be able to:

- Explain the differences between classes, structs, records, and record structs.
- Identify which are value types and which are reference types.
- Explain assignment behavior for each type.
- Explain equality behavior for each type.
- Understand the purpose of records and record structs.
- Decide which construct is appropriate for a given scenario.
- Recognize common design mistakes.
- Discuss trade-offs instead of relying on simple rules.
- Confidently answer common interview follow-up questions.

## References

### Microsoft Learn

- Classes
- Structs
- Records
- Record structs
- Equality in C#

### Additional Reading

- Framework Design Guidelines
- C# Language Specification
- C# in Depth
- CLR via C#
