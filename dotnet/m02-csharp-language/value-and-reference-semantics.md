# Value and Reference Semantics

## Definition

In C#, every type has either **value semantics** or **reference semantics**.

### Value Types

Value types store their data directly.

Examples:

```csharp
int
long
double
decimal
bool
char
enum
struct
record struct
```

Assigning a value type copies the value.

```csharp
int a = 10;
int b = a;

b = 20;
```

Result:

```text
a = 10
b = 20
```

### Reference Types

Reference types store a reference to an object.

Examples:

```csharp
class
record
string
array
delegate
```

Assigning a reference type copies the reference, not the object itself.

```csharp
var person1 = new Person();
var person2 = person1;
```

Both variables point to the same object.

Changes made through one reference are visible through the other.

```csharp
person2.Name = "John";
```

```text
person1.Name == "John"
```

This topic is one of the most frequently discussed areas in .NET interviews because it influences assignment, method calls, mutation, memory allocation, performance, and API design.

## Alternatives & Trade-offs

### Value Types

Pros:

- Often require fewer allocations
- Can improve locality of reference
- Naturally support value semantics
- Well suited for small immutable data

Cons:

- Entire value is copied when assigned
- Large structs can become expensive to copy
- Mutable structs can be difficult to reason about

Typical examples:

```csharp
DateTime
Guid
decimal
```

### Reference Types

Pros:

- Efficient sharing of large objects
- Natural fit for object-oriented design
- Avoid copying large amounts of data

Cons:

- Require an additional level of indirection
- Shared mutable state can introduce bugs
- Typically require managed heap allocation

Typical examples:

```csharp
Order
Customer
List<T>
```

### Class vs Struct

Prefer a struct when:

- The type is small
- The type represents a single value
- The type is immutable
- Value semantics are desired

Prefer a class when:

- The object is large
- The object has identity
- Shared state is required
- Lifetime management matters

## How It Works

### Value-Type Assignment

Assignment copies the value.

```csharp
Point p1 = new Point(10, 20);
Point p2 = p1;

p2.X = 100;
```

Result:

```text
p1.X == 10
p2.X == 100
```

Each variable contains an independent copy.

---

### Reference-Type Assignment

Assignment copies the reference.

```csharp
var p1 = new Person
{
    Name = "Alice"
};

var p2 = p1;

p2.Name = "Bob";
```

Result:

```text
p1.Name == "Bob"
p2.Name == "Bob"
```

Both variables reference the same object.

---

### Stack vs Heap

Value types are typically allocated on the stack (when local variables).

Reference types are allocated on the heap.

The stack is:

- Faster to allocate/deallocate
- Automatically reclaimed when scope exits
- Limited in size

The heap is:

- Managed by garbage collection
- Larger
- Shared across scopes

---

### Method Parameters

Value types are passed by value (copied).

```csharp
void ModifyValue(int x)
{
    x = 100;
}

int original = 10;
ModifyValue(original);
```

Result:

```text
original == 10
```

The parameter receives a copy.

Reference types are passed by reference (the reference is copied, but both point to the same object).

```csharp
void ModifyObject(Person p)
{
    p.Name = "Bob";
}

var person = new Person { Name = "Alice" };
ModifyObject(person);
```

Result:

```text
person.Name == "Bob"
```

Changes to the object are visible.

---

### Boxing and Unboxing

Boxing converts a value type to a reference type.

```csharp
int value = 10;
object boxed = value;
```

Unboxing converts back.

```csharp
int unboxed = (int)boxed;
```

Boxing has performance implications.

## Application

### Immutable Value Objects

Good candidates for structs:

```csharp
DateTime
TimeSpan
Guid
Money
Coordinate
```

---

### Entity-Like Objects

Good candidates for classes:

```csharp
Customer
Order
User
Product
```

---

### Avoiding Unnecessary Copies

Using `ref` parameters.

```csharp
void Process(ref LargeStruct data)
{
}
```

---

### Efficient Collections

Small immutable value objects can be stored efficiently in collections without boxing overhead.

## Common Mistakes

### Assuming Larger Objects Should Be Classes

Sometimes performance considerations suggest a struct, but complexity suggests a class.

---

### Creating Large Mutable Structs

Large structs that are mutable create expensive copies.

---

### Assuming Value Types Are Always Faster

Value types avoid heap allocation but copying large values can be expensive.

Measurement is important.

---

### Forgetting Stack Limitations

Stack-allocated data has scope limitations. Large data should typically be on the heap.

---

### Misunderstanding Method Parameter Semantics

Modifying a value type parameter does not affect the original.

Many candidates forget this.

## Common Interview Questions

### Basic

- What is the difference between a value type and a reference type?
- Where are value types typically allocated?
- Where are reference types typically allocated?
- What does it mean when we say assignment copies a value?

### Intermediate

- What is the difference between `struct` and `class`?
- How does assignment differ for value types vs reference types?
- Why are mutable structs problematic?
- What happens when a struct is passed as a method parameter?

### Advanced

- How does the CLR manage memory layout for value types vs reference types?
- What are the performance implications of struct size and copying frequency?
- How do you prevent unintended copies of large structs using `ref` parameters?
- What is the relationship between value semantics and immutability?
- How does boxing affect value type performance and behavior?
- What are the implications of `ref struct` and stack-only types?
- How should you design structs to minimize copying overhead?
- What are the implications of value types in generic constraints?
- How do inheritance hierarchies interact with value and reference semantics?
- How does escape analysis and struct optimization relate to performance?

### Follow-up Questions

- What is boxing?
- What is unboxing?
- Why does boxing have performance implications?
- Can you override the behavior of a struct when passed to a method?
- What is a `ref` parameter and when would you use it?

### Code Prediction

What is the output?

```csharp
int a = 5;
int b = a;

b = 10;

Console.WriteLine(a);
```

---

What is the output?

```csharp
var p1 = new Person { Name = "Alice" };
var p2 = p1;

p2.Name = "Bob";

Console.WriteLine(p1.Name);
```

---

What is the output?

```csharp
object value = 10;

int unboxed = (int)value;
unboxed = 20;

Console.WriteLine((int)value);
```

## Practical Tasks

### Type Choice

Choose struct or class for:

- `Temperature`
- `Customer`
- `Coordinate`
- `Order`
- `Color`

Explain your reasoning.

---

### Passing Parameters

Write methods that:

- Accept and modify a value type parameter
- Accept and modify a reference type parameter

Explain the differences in behavior.

---

### Large Data Handling

Design a method that processes large structs efficiently using `ref`.

---

### Boxing Investigation

Identify where boxing occurs in:

```csharp
object[] items = new object[] { 1, 2, 3 };
```

Explain the implications.

## Readiness Criteria

You should be able to:

- Explain the difference between value and reference types.
- Predict assignment behavior for each type.
- Know when to use structs vs classes.
- Understand stack vs heap allocation concepts.
- Recognize boxing and unboxing scenarios.
- Explain method parameter passing semantics.

## References

### Microsoft Learn

- [Value types](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/value-types)
- [Reference types](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/reference-types)
- [Structure types](https://learn.microsoft.com/dotnet/csharp/language-reference/builtin-types/struct)
- [Classes](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/classes)
