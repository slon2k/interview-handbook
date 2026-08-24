# Factory Pattern

## Definition

A Factory encapsulates object creation when the concrete type, construction rules, or dependencies should not be chosen by the caller.

## Alternatives & Trade-offs

Direct construction is clearest for simple objects. A factory earns its cost when creation varies, requires policy, or should be isolated from consumers.

## How It Works

A factory method or class selects and constructs an implementation. Dependency injection containers are composition tools, not automatically a reason to create another factory.

### Factory Method versus Abstract Factory

Factory Method creates one product through a method that can be overridden or selected by a concrete creator. Abstract Factory creates a related family of products through several creation operations.

```csharp
public interface IParser { }

public abstract class ParserFactory
{
	public abstract IParser CreateParser();
}

public interface IUiFactory
{
	IButton CreateButton();
	IMenu CreateMenu();
}
```

Use Factory Method when one product varies. Use Abstract Factory when several products must be created consistently as a family.

## Application

Use factories for parsers, providers, protocols, and objects requiring validation or configuration.

## Common Mistakes

- Creating a factory that only wraps `new`.
- Hiding required creation failures.
- Returning a broad type that omits needed behavior.

## Common Interview Questions

### Basic
- What is a factory?
- Why hide object creation?

### Intermediate
- When is a factory better than direct construction?
- What is the difference between Factory Method and Abstract Factory?

### Advanced
- How do factories manage versioned or plugin implementations?
- How can factories become service locators?
- How should creation failure be modeled?

### Follow-up Questions
- Can a factory be static?
- Does dependency injection replace all factories?

### Code Prediction
Which implementation is created when a factory receives a protocol value?

## Practical Tasks

- Create a factory for two payment providers.
- Refactor a factory that has accumulated unrelated responsibilities.

## Readiness Criteria

Explain creation encapsulation, factory trade-offs, failure behavior, and the boundary between factories and composition roots.

## References

### Microsoft Learn

- [Factory design pattern](https://learn.microsoft.com/dotnet/standard/design-patterns/factory-method)
