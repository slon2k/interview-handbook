# Reflection Basics

## Definition

Reflection allows code to inspect assemblies, types, members, and metadata at runtime, and in some cases invoke or construct members dynamically.

```csharp
Type type = typeof(string);
Console.WriteLine(type.FullName);
```

## Alternatives & Trade-offs

Use ordinary static code when types and members are known at compile time. Use interfaces, dependency injection, source generators, or explicit registration when runtime discovery is not required.

Reflection enables extensibility, serializers, test tools, and plugin systems, but it can reduce compile-time safety, add startup cost, complicate trimming, and produce runtime failures.

## How It Works

Assemblies contain metadata describing types and members. `Type`, `MethodInfo`, `PropertyInfo`, and related APIs expose that metadata.

```csharp
MethodInfo? method = typeof(string).GetMethod(nameof(string.ToUpperInvariant), Type.EmptyTypes);
object? result = method?.Invoke("hello", null);
```

Reflection invocation performs runtime checks and may wrap target exceptions in `TargetInvocationException`. Access restrictions and trimming can affect whether members are available.

## Application

- Discover plugins and registered implementations.
- Build serializers, mappers, and test frameworks.
- Inspect attributes and generated metadata.
- Implement diagnostics and tooling.
- Load types from configured assemblies when extensibility is required.

## Common Mistakes

- Calling reflection in hot loops without caching metadata.
- Assuming a member exists or has the expected signature.
- Ignoring accessibility and generic type rules.
- Swallowing `TargetInvocationException` without examining its inner exception.
- Relying on reflection in trimmed or native-AOT applications without preserving metadata.
- Using reflection where a direct call or interface would be clearer and safer.

## Common Interview Questions

### Basic

- What is reflection?
- What is the difference between `Type` and an object instance?
- How do you get a type using `typeof`?
- How do you inspect methods or properties?

### Intermediate

- How do you invoke a method using reflection?
- What is an assembly?
- How do you create an object dynamically?
- What exceptions can reflection invocation produce?

### Advanced

- How does reflection affect trimming and native AOT?
- What are the performance costs of `MethodInfo.Invoke`?
- How would you cache reflection metadata safely?
- How do open and closed generic types appear through reflection?
- How can source generation replace reflection in serializers?
- What security considerations apply to loading arbitrary assemblies?
- How do binding flags change member discovery?
- How should reflection-based plugin loading handle versioning and isolation?
- How do custom attributes interact with reflection caching?
- How would you design a reflection fallback with a statically generated fast path?

### Follow-up Questions

- What is the difference between `typeof(T)` and `obj.GetType()`?
- Why can `GetMethod` return null?
- What exception wraps an exception thrown by an invoked method?
- Can reflection access private members?
- What is the difference between metadata and runtime state?

### Code Prediction

What is printed?

```csharp
object value = "hello";
Console.WriteLine(value.GetType().Name);
Console.WriteLine(typeof(string).Name);
```

## Practical Tasks

### Type Inspection

Write a utility that lists public instance properties and their types for a supplied object.

### Plugin Discovery

Design a plugin loader that discovers implementations of an interface and rejects incompatible assemblies.

### Performance Review

Cache `MethodInfo` or compiled delegates and compare invocation cost with direct calls.

## Readiness Criteria

You should be able to inspect types and members, invoke methods carefully, explain runtime failure modes and performance costs, and identify when source generation or static abstractions are preferable.

## References

### Microsoft Learn

- [Reflection in .NET](https://learn.microsoft.com/dotnet/fundamentals/reflection/)
- [View type information](https://learn.microsoft.com/dotnet/fundamentals/reflection/view-type-information)
- [Dynamically load and use types](https://learn.microsoft.com/dotnet/fundamentals/reflection/dynamically-load-and-use-types)
- [System.Type](https://learn.microsoft.com/dotnet/api/system.type)
