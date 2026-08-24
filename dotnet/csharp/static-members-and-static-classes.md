# Static Members and Static Classes

## Definition

The `static` keyword indicates that a member belongs to a type itself rather than to a specific instance of that type.

Static can be applied to:

- Fields
- Properties
- Methods
- Constructors
- Classes

### Instance Members

Instance members belong to particular objects.

```csharp
public class User
{
    public string Name { get; set; } = string.Empty;
}
```

Each object has its own value.

```csharp
var user1 = new User();
var user2 = new User();

user1.Name = "Alice";
user2.Name = "Bob";
```

Result:

```text
user1.Name = Alice
user2.Name = Bob
```

### Static Members

Static members belong to the type itself.

```csharp
public class User
{
    public static int Count { get; set; }
}
```

The value is shared by all instances.

```csharp
User.Count++;
```

Access occurs through the type rather than an object.

### Static Classes

A static class cannot be instantiated and can contain only static members.

```csharp
public static class MathHelper
{
    public static int Square(int value)
    {
        return value * value;
    }
}
```

Usage:

```csharp
var result = MathHelper.Square(5);
```

Static classes are commonly used for utility functions and extension methods.

## Alternatives & Trade-offs

### Static Member vs Instance Member

#### Static Member

Pros:

- No object creation required
- Easy access
- Suitable for shared state
- Useful for utility functions

Cons:

- Harder to test
- Harder to mock
- Global state can introduce bugs
- Can create hidden dependencies

#### Instance Member

Pros:

- Supports dependency injection
- Easier testing
- Supports polymorphism
- Better encapsulation

Cons:

- Requires object creation

Use static members when behavior truly belongs to the type rather than an instance.

---

### Static Class vs Service Class

Static utility:

```csharp
public static class DateHelper
{
}
```

Dependency-injected service:

```csharp
public sealed class DateService
{
}
```

For simple deterministic calculations:

```csharp
Math
DateTime
Guid
```

a static design is often appropriate.

For business logic that depends on:

- Configuration
- Databases
- External services
- Current time

a service class is often preferable.

---

### Static State vs Dependency Injection

Static state:

```csharp
public static class Settings
{
    public static string ConnectionString { get; set; }
}
```

Dependency injection:

```csharp
public sealed class DatabaseOptions
{
    public string ConnectionString { get; init; } = string.Empty;
}
```

Dependency injection is usually preferred because dependencies become explicit and testable.

## How It Works

### Static Fields

A static field exists once per type.

```csharp
public class User
{
    public static int Count;
}
```

All instances share the same field.

```csharp
var user1 = new User();
var user2 = new User();

User.Count++;
```

There is only a single `Count` value.

---

### Static Properties

Static properties provide controlled access to static data.

```csharp
public class ApplicationSettings
{
    public static string Version { get; } = "1.0";
}
```

Usage:

```csharp
Console.WriteLine(ApplicationSettings.Version);
```

---

### Static Methods

Static methods do not operate on instance state.

```csharp
public static class MathHelper
{
    public static int Add(int left, int right)
    {
        return left + right;
    }
}
```

Usage:

```csharp
var result = MathHelper.Add(10, 20);
```

---

### Accessing Instance Members

Static methods cannot directly access instance members.

Invalid:

```csharp
public class User
{
    public string Name { get; set; } = string.Empty;

    public static void Print()
    {
        Console.WriteLine(Name);
    }
}
```

Compilation error:

```text
An object reference is required
```

A static member has no implicit object instance.

---

### Static Constructor

A static constructor initializes static data.

```csharp
public class Configuration
{
    static Configuration()
    {
        Console.WriteLine("Initialized");
    }
}
```

Characteristics:

- Executes automatically
- Runs once per type
- Cannot accept parameters
- Cannot have access modifiers

---

### Static Classes

A static class:

- Cannot be instantiated
- Cannot inherit from another class
- Cannot contain instance members

Example:

```csharp
public static class StringExtensions
{
}
```

The compiler enforces these rules.

## Application

### Utility Classes

Common examples:

```csharp
Math
Convert
Guid
Environment
Path
```

These APIs expose functionality that does not depend on object state.

---

### Extension Methods

Extension methods must be defined in static classes.

```csharp
public static class StringExtensions
{
    public static bool IsNullOrWhiteSpaceCustom(
        this string value)
    {
        return string.IsNullOrWhiteSpace(value);
    }
}
```

---

### Shared Counters

```csharp
public class User
{
    public static int CreatedUsers;
}
```

Useful for tracking application-wide metrics.

---

### Caching

```csharp
private static readonly Dictionary<int, string> Cache
    = new();
```

Static state may be used for application-wide caches.

Care must be taken regarding thread safety.

---

### Constants and Configuration

```csharp
public static class ApplicationConstants
{
    public const int MaxRetries = 3;
}
```

Provides a centralized location for shared values.

## Common Mistakes

### Using Static for Business Logic

Poor:

```csharp
public static class UserService
{
}
```

Business services often benefit from dependency injection and testability.

---

### Treating Static State as Global Storage

Problematic:

```csharp
public static class CurrentUser
{
    public static User User { get; set; }
}
```

This introduces hidden dependencies and testing difficulties.

---

### Ignoring Thread Safety

Unsafe:

```csharp
public static int Counter;
```

Multiple threads modifying the same static field may produce race conditions.

---

### Overusing Utility Classes

Not everything must be static.

Many utility classes eventually grow dependencies and become difficult to maintain.

---

### Accessing Instance Members from Static Members

Invalid:

```csharp
public static void Print()
{
    Console.WriteLine(Name);
}
```

Static members do not have access to an instance.

---

### Assuming Static Means Faster

Static methods avoid object creation but do not automatically improve application performance.

Design and maintainability are usually more important.

## Common Interview Questions

### Basic

- What is a static member?
- What is a static class?
- What is the difference between static and instance members?
- How do you access a static method?

### Intermediate

- What is a static constructor?
- When does a static constructor execute?
- Why can't static methods access instance members?
- When should a class be static?

### Advanced

- What are the implications of static state in multithreaded applications?
- How do you implement thread-safe static caches and singletons?
- What is the difference between eager initialization and lazy initialization with `Lazy<T>`?
- How should static members be designed to support unit testing?
- What are the implications of static constructors for assembly loading and diagnostics?
- How do you refactor static utilities to support dependency injection?
- What design patterns use static members (Singleton, Factory, Registry)?
- How does static state interact with async/await and task scheduling?
- What are the memory implications of static fields in long-running applications?
- How should you handle static state in multi-domain scenarios (e.g., plugin architectures)?

### Follow-up Questions

- Can a static class implement an interface?
- Can a static class be inherited?
- Are static fields shared between all instances?
- What thread-safety concerns exist with static members?
- What are the drawbacks of static state?
- Why is dependency injection often preferred over static classes?

### Code Prediction

What is the output?

```csharp
public class Counter
{
    public static int Value;
}

Counter.Value++;
Counter.Value++;

Console.WriteLine(Counter.Value);
```

---

What is the output?

```csharp
public class User
{
    public static int Count;
}

var user1 = new User();
var user2 = new User();

User.Count++;

Console.WriteLine(User.Count);
```

---

Will this compile?

```csharp
public class User
{
    public string Name { get; set; } = string.Empty;

    public static void Print()
    {
        Console.WriteLine(Name);
    }
}
```

Why?

---

How many times is the static constructor executed?

```csharp
public class Config
{
    static Config()
    {
        Console.WriteLine("Initialized");
    }
}
```

## Practical Tasks

### Refactoring

Review:

```csharp
public static class UserService
{
    public static void CreateUser()
    {
    }
}
```

Would you keep it static?

Discuss the trade-offs.

---

### Code Review

Review:

```csharp
public static class CurrentUser
{
    public static string Name { get; set; }
}
```

Identify potential design problems.

---

### Design Exercise

Choose between:

- Static class
- Service class

for:

```text
EmailSender
TaxCalculator
DateFormatter
CurrencyConverter
```

Explain your reasoning.

---

### Thread-Safety Exercise

Review:

```csharp
public static class Statistics
{
    public static int RequestsProcessed;
}
```

Would this be safe in a multi-threaded application?

How would you improve it?

---

### Extension Method

Implement an extension method:

```csharp
string.IsValidEmail()
```

and explain why the class must be static.

## Readiness Criteria

You should be able to:

- Explain the difference between static and instance members.
- Explain when static members are appropriate.
- Explain how static fields and properties behave.
- Understand the purpose of static constructors.
- Explain the limitations of static classes.
- Identify design problems caused by global static state.
- Discuss testing and dependency-injection implications.
- Recognize common thread-safety issues involving static members.
- Choose between static and instance-based designs appropriately.
- Confidently answer common interview follow-up questions.

## References

### Microsoft Learn

- [`static` keyword](https://learn.microsoft.com/dotnet/csharp/language-reference/keywords/static)
- [Static classes and static class members](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/static-classes-and-static-class-members)
- [Static constructors](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/static-constructors)
- [Extension methods](https://learn.microsoft.com/dotnet/csharp/programming-guide/classes-and-structs/extension-methods)
