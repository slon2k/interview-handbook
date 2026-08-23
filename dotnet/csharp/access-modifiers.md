# Access Modifiers

## Definition

Access modifiers control the visibility and accessibility of types and members.

They define which code can access:

- Classes
- Structs
- Interfaces
- Methods
- Properties
- Fields
- Constructors

Access modifiers are a fundamental encapsulation mechanism and help prevent unintended dependencies between components.

The available access modifiers are:

| Modifier | Accessible From |
|-----------|----------------|
| `public` | Anywhere |
| `private` | Containing type only |
| `protected` | Containing type and derived types |
| `internal` | Same assembly |
| `protected internal` | Same assembly OR derived types |
| `private protected` | Same assembly AND derived types |

### Public

```csharp
public class UserService
{
}
```

Accessible from any code that can reference the assembly.

### Private

```csharp
public class User
{
    private string _name;
}
```

Accessible only within the containing type.

### Protected

```csharp
public class Animal
{
    protected string Name;
}
```

Accessible within derived types.

### Internal

```csharp
internal class CacheManager
{
}
```

Accessible only within the current assembly.

### Protected Internal

```csharp
protected internal void Process()
{
}
```

Accessible from:

- The same assembly
- Derived types in other assemblies

### Private Protected

```csharp
private protected void Process()
{
}
```

Accessible only from derived types within the same assembly.

## Alternatives & Trade-offs

### Public vs Private

#### Public

Pros:

- Easy to consume
- Exposes functionality to clients

Cons:

- Creates API surface area
- Harder to change later
- Increases coupling

#### Private

Pros:

- Strong encapsulation
- Easier refactoring
- Implementation details remain hidden

Cons:

- Not accessible from outside the type

A good rule is:

```text
Start with the most restrictive access possible.
Relax it only when necessary.
```

---

### Protected vs Internal

#### Protected

Best when supporting inheritance.

```csharp
protected virtual void Validate()
{
}
```

#### Internal

Best when sharing functionality within a solution or assembly.

```csharp
internal class QueryBuilder
{
}
```

Many teams prefer `internal` over inheritance-based extension points.

---

### Public API vs Internal Implementation

Good design:

```csharp
public class UserService
{
}
```

```csharp
internal class UserRepository
{
}
```

Consumers see the public API while implementation details remain hidden.

## How It Works

### Member Accessibility

Consider:

```csharp
public class User
{
    private string _name;

    public void SetName(string name)
    {
        _name = name;
    }
}
```

`_name` is accessible only inside `User`.

External code cannot access it.

```csharp
var user = new User();

user._name = "Alice";
```

Compilation error:

```text
Inaccessible due to its protection level
```

---

### Type Accessibility

Top-level types may be:

```csharp
public
internal
```

For example:

```csharp
internal class EmailSender
{
}
```

Only code within the same assembly can use this type.

---

### Protected Members

```csharp
public class Animal
{
    protected string Name = string.Empty;
}

public class Dog : Animal
{
    public void Print()
    {
        Console.WriteLine(Name);
    }
}
```

The derived type may access `Name`.

External consumers cannot.

---

### Constructor Accessibility

Constructors can have access modifiers.

Public constructor:

```csharp
public User()
{
}
```

Private constructor:

```csharp
private User()
{
}
```

Private constructors are commonly used in:

- Singleton implementations
- Factory methods
- Static utility classes

Example:

```csharp
public static class MathHelper
{
}
```

The compiler implicitly prevents instantiation.

## Application

### Encapsulation

Preferred:

```csharp
public class User
{
    private string _email = string.Empty;

    public string Email => _email;
}
```

Internal implementation remains hidden.

---

### Building Stable APIs

Expose only what users need.

```csharp
public interface IUserService
{
}
```

```csharp
internal class UserService : IUserService
{
}
```

Consumers depend on contracts rather than implementations.

---

### Framework and Library Development

Libraries frequently use:

```csharp
public
```

for supported APIs and

```csharp
internal
```

for implementation details.

---

### Inheritance-Based Design

Protected members provide extension points.

```csharp
protected virtual void Validate()
{
}
```

Derived types can extend behavior.

---

### Testing

Internal members can be exposed to test projects using:

```csharp
InternalsVisibleTo
```

This is a common interview discussion topic.

## Common Mistakes

### Making Everything Public

Poor:

```csharp
public class User
{
    public string Name;
    public string Email;
    public int Age;
}
```

Excessive visibility increases coupling.

---

### Exposing Fields Instead of Properties

Poor:

```csharp
public string Name;
```

Preferred:

```csharp
public string Name { get; private set; }
```

---

### Using Protected Without a Real Inheritance Scenario

Many developers expose members as `protected` "just in case".

This often creates unnecessary coupling.

---

### Misunderstanding Internal

Some candidates assume:

```text
internal == current namespace
```

Incorrect.

```text
internal == current assembly
```

---

### Using Public Setters Everywhere

Poor:

```csharp
public string Email { get; set; }
```

Consider:

```csharp
public string Email { get; private set; }
```

when external modification should be restricted.

---

### Ignoring Encapsulation

Access modifiers are not merely a language feature; they help define boundaries and maintain invariants.

## Common Interview Questions

### Basic

- What access modifiers does C# provide?
- What is the difference between `public` and `private`?
- What is the purpose of `protected`?
- What does `internal` mean?

### Intermediate

- What is the difference between `protected` and `private`?
- What is the difference between `internal` and `public`?
- Why might you make a constructor private?
- When should a member be `protected`?

### Follow-up Questions

- What is the difference between `protected internal` and `private protected`?
- What are top-level type accessibility rules?
- What is `InternalsVisibleTo`?
- Why should public APIs be minimized?
- What access modifier would you choose for a repository implementation?

### Code Prediction

Will this compile?

```csharp
public class User
{
    private string _name;
}

var user = new User();

Console.WriteLine(user._name);
```

Why?

---

Will this compile?

```csharp
public class Animal
{
    protected string Name;
}

var animal = new Animal();

Console.WriteLine(animal.Name);
```

Why?

---

Can `Dog` access `Name`?

```csharp
public class Animal
{
    protected string Name;
}

public class Dog : Animal
{
}
```

---

Which modifier should be used?

```csharp
class QueryBuilder
{
}
```

This type is used only inside the current assembly.

## Practical Tasks

### Refactoring

Review:

```csharp
public class User
{
    public string Name;
    public string Email;
}
```

Reduce visibility where appropriate.

---

### Design Exercise

Design a service with:

- Public API
- Private implementation details

Explain your modifier choices.

---

### Library Design

Determine whether each type should be:

- public
- internal

```text
UserService
EmailValidator
SqlQueryGenerator
ApiClient
```

Explain your reasoning.

---

### Inheritance Exercise

Implement:

```csharp
Animal
Dog
Cat
```

using `protected` members appropriately.

---

### Code Review

Review:

```csharp
public class Order
{
    public decimal Total { get; set; }
}
```

Should the setter be public?

Why or why not?

## Readiness Criteria

You should be able to:

- Explain all C# access modifiers.
- Describe the accessibility rules for each modifier.
- Understand the difference between class-level and member-level accessibility.
- Explain when to use `internal` versus `public`.
- Explain when `protected` is appropriate.
- Understand `protected internal` and `private protected`.
- Design APIs that expose only necessary functionality.
- Apply encapsulation principles effectively.
- Recognize common visibility and coupling problems.
- Confidently answer common interview follow-up questions.

## References

### Microsoft Learn

- Access Modifiers
- Access Levels
- Encapsulation
- Accessibility Levels
- Access Control

### Additional Reading

- Framework Design Guidelines
- C# Language Specification
- CLR via C#
- C# in Depth
