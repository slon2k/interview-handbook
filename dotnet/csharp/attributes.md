# Attributes

## Definition

Attributes attach declarative metadata to program elements such as types, methods, properties, and parameters.

```csharp
[Obsolete("Use the new API")]
public void LegacyMethod() { }
```

A custom attribute derives from `System.Attribute`.

```csharp
[AttributeUsage(AttributeTargets.Class, Inherited = true)]
public sealed class AuditedAttribute : Attribute { }
```

## Alternatives & Trade-offs

Attributes are a good fit for metadata consumed by compilers, frameworks, serializers, or tooling. Explicit configuration is easier to discover and validate for complex behavior. Interfaces are better when the type must provide executable behavior.

Attributes are declarative and reusable, but their meaning depends on a consumer. Overusing them can hide control flow and make configuration difficult to trace.

## How It Works

Attribute instances are represented in metadata and are usually constructed when reflection requests them. Attribute arguments must be compile-time constants, types, enums, or arrays of supported values.

```csharp
var attribute = typeof(MyType)
    .GetCustomAttributes(typeof(AuditedAttribute), inherit: true)
    .SingleOrDefault();
```

Applying an attribute does not execute its behavior by itself. A compiler, analyzer, source generator, or runtime framework must inspect it.

## Application

- Mark obsolete APIs or supported platforms.
- Configure serialization, validation, routing, and authorization.
- Expose metadata to analyzers and source generators.
- Mark tests, data rows, and framework conventions.

## Common Mistakes

- Assuming attributes automatically change runtime behavior.
- Putting complex logic inside attribute constructors.
- Using reflection repeatedly in hot paths without caching.
- Forgetting attribute inheritance rules.
- Passing mutable objects as if attribute arguments could be arbitrary runtime values.
- Using attributes where explicit configuration would be clearer.

## Common Interview Questions

### Basic

- What is an attribute?
- How do you create a custom attribute?
- What does `AttributeUsage` control?
- When are attribute values evaluated?

### Intermediate

- How do you retrieve attributes with reflection?
- What is the difference between named and positional arguments?
- Are attributes inherited by derived classes?
- Can an attribute be applied multiple times?

### Advanced

- How are attribute arguments encoded in assembly metadata?
- What are the trimming and AOT risks of reflection-based attribute discovery?
- How should attribute lookup be cached safely?
- How do source generators differ from runtime attribute consumers?
- What does `AllowMultiple` change in a custom attribute?
- How do inherited attributes behave on overridden members?
- How would you validate attribute configuration at build time?
- What are the versioning risks of changing attribute constructor signatures?
- How can attributes affect framework startup performance?
- When should metadata be modeled as a type or configuration object instead?

### Follow-up Questions

- Can an attribute constructor accept a `DateTime` value?
- What is the `Attribute` suffix convention?
- How do you retrieve inherited attributes?
- Can attributes be applied to generic type parameters?
- What is the difference between an attribute and a pragma?

### Code Prediction

What does this print?

```csharp
[Obsolete]
static void OldApi() { }
```

## Practical Tasks

### Custom Metadata

Create an attribute that marks properties as sensitive and write a reflection-based report that finds them.

### Usage Rules

Configure `AttributeUsage` to permit one attribute on methods and inherited use on derived classes.

### Design Review

Replace an attribute carrying many settings with a typed configuration object where that improves clarity.

## Readiness Criteria

You should be able to define custom attributes, control their usage, retrieve them correctly, and explain the boundary between metadata and the code that consumes it.

## References

### Microsoft Learn

- [Attributes](https://learn.microsoft.com/dotnet/csharp/advanced-topics/reflection-and-attributes/)
- [Create custom attributes](https://learn.microsoft.com/dotnet/standard/attributes/writing-custom-attributes)
- [AttributeUsageAttribute](https://learn.microsoft.com/dotnet/api/system.attributeusageattribute)
- [System.Attribute](https://learn.microsoft.com/dotnet/api/system.attribute)
