# Boxing and Allocation Pressure

## Definition

Boxing copies a value type into an object on the managed heap. Allocation pressure describes the rate and lifetime pattern of objects that the GC must manage.

```csharp
int number = 42;
object boxed = number;
int value = (int)boxed;
```

## Alternatives & Trade-offs

Typed generic APIs often avoid boxing. Do not remove every allocation automatically; a small allocation can be cheaper and clearer than pooling or unsafe optimization.

## How It Works

Boxing creates an object containing the value. Interface conversion can also box a struct when the interface is not handled through a constrained generic path. Other allocation sources include closures, temporary strings, LINQ materialization, collection growth, and delegate instances.

```csharp
object[] items = [1, 2, 3]; // each int is boxed
```

Unboxing validates the boxed value's type and copies it back to a value type; it does not undo the original allocation.

## Application

- Prefer typed collections over `object[]`.
- Avoid repeated boxing in high-volume loops.
- Pre-size collections when the approximate size is known.
- Measure allocation rate and GC pressure before optimizing.
- Use pooling only with clear ownership, reset, and return rules.

## Common Mistakes

- Assuming all value types are stack-only.
- Missing implicit boxing through `object` or interfaces.
- Pooling objects without clearing sensitive data.
- Optimizing allocations without a benchmark or profile.

## Common Interview Questions

### Basic

- What is boxing?
- What is unboxing?
- Why can boxing affect performance?

### Intermediate

- Where can implicit boxing occur?
- How do generics reduce boxing?
- What is allocation pressure?

### Advanced

- How do constrained calls reduce boxing?
- How can closures and delegates allocate?
- How would you prove an allocation is worth optimizing?
- What are the risks of pooling mutable objects?
- How does allocation lifetime affect GC cost?

### Follow-up Questions

- Does unboxing allocate?
- Can an interface conversion box a struct?
- Is every allocation a performance problem?

### Code Prediction

Where does allocation occur?

```csharp
int value = 10;
object boxed = value;
```

## Practical Tasks

- Find boxing in an `object[]` workflow.
- Compare a generic implementation with an object-based implementation.
- Measure allocations before and after pre-sizing a collection.

## Readiness Criteria

Recognize boxing and common allocation sources, explain their lifetime impact, and use measurements to prioritize optimization.

## References

### Microsoft Learn

- [Boxing and unboxing](https://learn.microsoft.com/dotnet/csharp/programming-guide/types/boxing-and-unboxing)
- [Performance best practices](https://learn.microsoft.com/dotnet/framework/performance/)
