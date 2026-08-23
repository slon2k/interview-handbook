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
