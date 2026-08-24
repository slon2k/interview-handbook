# Decorator Pattern

## Definition

The Decorator pattern adds behavior to an object by wrapping it in another object that implements the same interface, delegating to the wrapped instance and adding work before or after.

```csharp
public interface IReportGenerator { string Generate(); }

public sealed class LoggingReportDecorator : IReportGenerator
{
    private readonly IReportGenerator _inner;
    public LoggingReportDecorator(IReportGenerator inner) => _inner = inner;

    public string Generate()
    {
        Console.WriteLine("Generating report...");
        var result = _inner.Generate();
        Console.WriteLine("Report generated.");
        return result;
    }
}
```

## Alternatives & Trade-offs

Decorators compose cross-cutting behavior (logging, caching, retries, authorization) without subclassing every combination, and each decorator stays small and independently testable. For a single, always-applied behavior with no need for combination or ordering variation, a straightforward method call or middleware may be clearer than a decorator chain. ASP.NET Core middleware is essentially the Decorator pattern applied to the request pipeline.

## How It Works

```csharp
public sealed class CachingReportDecorator : IReportGenerator
{
    private readonly IReportGenerator _inner;
    private string? _cached;

    public CachingReportDecorator(IReportGenerator inner) => _inner = inner;

    public string Generate() => _cached ??= _inner.Generate();
}

// Composing decorators — order matters
IReportGenerator generator = new LoggingReportDecorator(
    new CachingReportDecorator(
        new SalesReportGenerator()));

generator.Generate();
```

Here, logging wraps caching, which wraps the real generator. Calling `Generate()` a second time logs "Generating report..." and "Report generated." even though the cached decorator returns the stored value without re-running `SalesReportGenerator`, because the logging decorator has no idea caching happened underneath it — a concrete illustration of why decorator ordering changes observable behavior.

## Application

Common decorator uses: logging, caching, authorization checks, retry logic, and metrics — all things you might want to add or remove from a component without changing the component itself.

## Common Mistakes

- Changing the wrapped contract unexpectedly (a decorator that changes a method's return semantics breaks substitutability).
- Leaving decorator ordering undocumented, when swapping the order (e.g., caching outside vs. inside a logging decorator) changes what actually gets logged or cached.
- Introducing hidden global/static state inside a decorator, making it behave differently depending on call order elsewhere in the app.
- Using decorators for a single fixed behavior that never varies or combines with anything else, where a direct method call would be simpler.

## Common Interview Questions

### Basic
- What does Decorator solve, and how does it differ from inheritance?
- Is ASP.NET Core middleware an example of the Decorator pattern?

### Intermediate
- Why does decorator ordering matter?
- How does Decorator support the Open/Closed Principle?

### Advanced
- How should decorators handle exceptions and cancellation tokens passed through the chain?
- How can nested decorators affect observability (e.g., a caching decorator hiding calls from a logging decorator)?
- When is middleware a better fit than a hand-rolled decorator chain?

### Follow-up Questions
- Can decorators be stacked arbitrarily deep?
- How would you test that decorators run in the correct order?

### Code Prediction
Using the `LoggingReportDecorator(CachingReportDecorator(...))` example above, what gets printed on the *second* call to `Generate()`? Would the output differ if the decorators were nested in the opposite order?

## Practical Tasks

- Add logging and caching decorators to a report-generation service and observe how their ordering changes behavior.
- Write a unit test asserting that a caching decorator does not call the wrapped generator on the second invocation.
- Document and test the intended decorator ordering for a small pipeline (e.g., retry outside of logging, or vice versa).

## Readiness Criteria

Implement composable decorators that preserve the wrapped contract, explain how ordering changes observable behavior with a concrete example, and distinguish decorators from middleware and plain inheritance.

## References

### Microsoft Learn

- [Decorator design pattern](https://learn.microsoft.com/dotnet/standard/design-patterns/decorator)
- [ASP.NET Core middleware](https://learn.microsoft.com/aspnet/core/fundamentals/middleware/)
