# Fields, Properties and Constructors

## Definition

Fields, properties, and constructors are fundamental building blocks of object design in C#.

They define:

- What data an object stores
- How that data can be accessed
- How an object is initialized

### Fields

A field is a variable declared directly within a type.

```csharp
public class User
{
    private string _name;
}
```

Fields represent the actual storage of data.

### Properties

A property provides controlled access to data.

```csharp
public class User
{
    public string Name { get; set; }
}
```

Properties are the preferred way to expose state from a type.

### Constructors

A constructor initializes a new instance of a type.

```csharp
public class User
{
    public User()
    {
    }
}
```

Constructors ensure an object starts in a valid state.

## Alternatives & Trade-offs

### Field vs Property

#### Field

```csharp
public string Name;
```

Pros:

- Simple
- Minimal syntax
- Useful for internal implementation details

Cons:

- No validation
- No encapsulation
- Difficult to evolve without breaking consumers

#### Property

```csharp
public string Name { get; set; }
```

Pros:

- Encapsulation
- Validation support
- Backward-compatible evolution
- Framework-friendly

Cons:

- Slightly more verbose

Properties are generally preferred for public APIs.

---

### Auto-Property vs Full Property

Auto-property:

```csharp
public string Name { get; set; }
```

Simple and commonly used.

Full property:

```csharp
private string _name = string.Empty;

public string Name
{
    get => _name;
    set
    {
        if (string.IsNullOrWhiteSpace(value))
        {
            throw new ArgumentException();
        }

        _name = value;
    }
}
```

Use when validation or custom logic is required.

---

### Constructor vs Object Initializer

Constructor:

```csharp
var user = new User("Alice");
```

Object initializer:

```csharp
var user = new User
{
    Name = "Alice"
};
```

Constructors are preferable when an object has required data.

Object initializers are useful for optional configuration.

## How It Works

### Fields

A field stores data within an object.

```csharp
public class User
{
    private string _name = string.Empty;
}
```

Every instance receives its own field values.

```csharp
var user1 = new User();
var user2 = new User();
```

The field values belong to the object instance.

---

### Properties

Properties expose accessors.

```csharp
public string Name
{
    get;
    set;
}
```

The compiler automatically generates a hidden backing field.

Conceptually:

```csharp
private string _name;

public string Name
{
    get => _name;
    set => _name = value;
}
```

---

### Read-Only Properties

```csharp
public string Name { get; }
```

The value can only be assigned during construction.

---

### Init-Only Properties

```csharp
public string Name { get; init; }
```

The property may be assigned during object creation.

```csharp
var user = new User
{
    Name = "Alice"
};
```

After initialization the value becomes read-only.

---

### Constructors

A constructor executes whenever an object is created.

```csharp
public class User
{
    public User()
    {
        Console.WriteLine("Created");
    }
}
```

---

### Parameterized Constructors

```csharp
public class User
{
    public string Name { get; }

    public User(string name)
    {
        Name = name;
    }
}
```

Used when data is required.

---

### Constructor Chaining

Constructors can delegate to one another.

```csharp
public class User
{
    public User()
        : this("Unknown")
    {
    }

    public User(string name)
    {
        Name = name;
    }

    public string Name { get; }
}
```

This reduces duplication.

---

### Primary Constructors

Modern C# supports primary constructors.

```csharp
public class User(string name)
{
    public string Name { get; } = name;
}
```

They reduce boilerplate for simple types.

## Application

### Encapsulation

```csharp
private decimal _balance;
```

Internal fields remain hidden.

---

### Validation

```csharp
public string Email
{
    get => _email;
    set
    {
        if (string.IsNullOrWhiteSpace(value))
        {
            throw new ArgumentException();
        }

        _email = value;
    }
}
```

Properties provide a natural place for validation.

---

### Immutable Objects

```csharp
public class User
{
    public string Name { get; }

    public User(string name)
    {
        Name = name;
    }
}
```

Useful for thread safety and predictable behavior.

---

### Enforcing Invariants

```csharp
public Order(decimal total)
{
    if (total < 0)
    {
        throw new ArgumentOutOfRangeException();
    }

    Total = total;
}
```

Constructors ensure invalid objects cannot be created.

---

### Dependency Injection

Constructors commonly receive dependencies.

```csharp
public class UserService
{
    private readonly IUserRepository _repository;

    public UserService(IUserRepository repository)
    {
        _repository = repository;
    }
}
```

This is known as constructor injection.

## Common Mistakes

### Exposing Public Fields

Poor:

```csharp
public string Name;
```

Preferred:

```csharp
public string Name { get; set; }
```

Public fields bypass encapsulation.

---

### Performing Heavy Work in Constructors

Problematic:

```csharp
public UserService()
{
    LoadLargeDataSet();
}
```

Constructors should generally be lightweight.

---

### Creating Invalid Objects

Poor:

```csharp
public class User
{
    public string Email { get; set; }
}
```

This permits:

```csharp
new User
{
    Email = ""
};
```

Provide validation when necessary.

---

### Excessive Property Logic

Properties should not hide expensive work.

Surprising:

```csharp
public decimal Price
{
    get
    {
        LoadFromDatabase();
        return _price;
    }
}
```

Consumers expect properties to be inexpensive.

---

### Constructor Duplication

Poor:

```csharp
public User()
{
    Name = "Unknown";
    Age = 0;
}

public User(string name)
{
    Name = name;
    Age = 0;
}
```

Prefer constructor chaining.

---

### Overusing Setters

Not every property should be mutable.

Many domain models become easier to reason about when state changes are controlled.

## Common Interview Questions

### Basic

- What is the difference between a field and a property?
- Why are properties preferred over public fields?
- What is a constructor?
- When is a constructor executed?

### Intermediate

- What is an auto-property?
- What is a backing field?
- What is a read-only property?
- What is an init-only property?
- What is constructor chaining?

### Follow-up Questions

- Can a class have multiple constructors?
- Can constructors be private?
- Why might a private constructor be useful?
- Should validation live in properties or constructors?
- What are the advantages of immutable objects?
- How does constructor injection work?

### Code Prediction

What is the output?

```csharp
public class User
{
    public string Name { get; set; } = "Unknown";
}

var user = new User();

Console.WriteLine(user.Name);
```

---

What is the output?

```csharp
public class User
{
    public string Name { get; }

    public User(string name)
    {
        Name = name;
    }
}

var user = new User("Alice");

Console.WriteLine(user.Name);
```

---

Will this compile?

```csharp
public class User
{
    public string Name { get; }
}

var user = new User();

user.Name = "Alice";
```

Why?

---

What happens?

```csharp
public class User
{
    public User()
    {
        throw new Exception();
    }
}

var user = new User();
```

## Practical Tasks

### Refactoring

Convert:

```csharp
public class User
{
    public string Name;
    public string Email;
}
```

to use properties.

Explain the benefits.

---

### Validation

Implement a property that prevents empty values.

```csharp
public string Email
{
    ...
}
```

---

### Constructor Design

Design a constructor that guarantees:

```text
Order total must be >= 0
Customer Id must not be empty
```

---

### Code Review

Review:

```csharp
public class Product
{
    public decimal Price { get; set; }
}
```

Should negative values be possible?

How would you improve the design?

---

### Dependency Injection

Implement constructor injection for:

```csharp
ILogger
IEmailSender
```

inside a service class.

## Readiness Criteria

You should be able to:

- Explain the difference between fields and properties.
- Know when a field should be private.
- Explain why properties are preferred for public APIs.
- Describe auto-properties, backing fields, and accessors.
- Explain read-only and init-only properties.
- Explain the purpose of constructors.
- Use constructor chaining appropriately.
- Design objects that enforce their own invariants.
- Recognize common encapsulation and initialization problems.
- Confidently answer common interview follow-up questions.

## References

### Microsoft Learn

- Fields
- Properties
- Automatically Implemented Properties
- init Accessors
- Constructors
- Primary Constructors

### Additional Reading

- Framework Design Guidelines
- C# in Depth
- C# Language Specification
- CLR via C#
