# Exception Handling and Design

## Definition

Exceptions represent failures that interrupt normal control flow. .NET exceptions derive from `System.Exception`; common types include `ArgumentException`, `InvalidOperationException`, `IOException`, and `OperationCanceledException`.

Use exceptions for exceptional failures. Use validation results, nullable values, or Try-pattern APIs when failure is expected and part of normal control flow.

## Alternatives & Trade-offs

Catching locally is useful only when the code can recover, translate, or add meaningful context. Let failures propagate when a higher layer owns the recovery policy. A result type can make expected failure states explicit and avoid exception overhead.

## How It Works

`try` protects risky code, `catch` handles compatible failures, and `finally` performs cleanup. Exception filters use `when` to select a handler without changing the exception.

```csharp
try
{
    Save(document);
}
catch (IOException exception) when (exception.Message.Contains("locked"))
{
    RetryLater();
}
finally
{
    ReleaseTemporaryState();
}
```

Use `throw;` to preserve the original stack trace. `throw exception;` starts the stack trace at the rethrow site. Wrap an exception only when adding useful context, and preserve it as the inner exception.

## Application

- Use `ArgumentException` for invalid caller arguments.
- Use `InvalidOperationException` when the object's current state does not support an operation.
- Use custom exceptions when callers need a distinct recovery policy.
- Translate infrastructure failures at application boundaries.
- Log once at the boundary that owns the failure policy.
- Never expose stack traces or sensitive implementation details to external callers.

### Layered Boundary Example

A storage layer can throw a provider-specific exception. The application layer can translate it into a domain failure, while the transport layer converts that failure into a safe response. Each layer should catch only what it understands.

## Common Mistakes

- Catching `Exception` and continuing with invalid state.
- Using exceptions for ordinary validation or branching.
- Losing the stack trace with `throw exception;`.
- Logging and rethrowing the same failure at every layer.
- Throwing from `finally` and hiding the original failure.
- Treating cancellation as an ordinary server error.
- Returning sensitive details in exception messages.

## Common Interview Questions

### Basic

- What is the base exception type?
- What do `try`, `catch`, and `finally` do?
- What is the difference between `throw;` and `throw exception;`?

### Intermediate

- When should an exception be custom?
- Where should exceptions be logged?
- When should a result type replace an exception?
- How do exception filters work?

### Advanced

- How does stack unwinding execute nested `finally` blocks?
- How should exception boundaries preserve causality and correlation?
- How should retry logic distinguish transient and permanent failures?
- How does changing exception types affect API compatibility?
- How should cancellation differ from failure in asynchronous code?

### Follow-up Questions

- Does `finally` run when a catch block returns?
- Why is `OperationCanceledException` often not an application error?
- What should an external error response contain?

### Code Prediction

What is printed?

```csharp
try
{
    throw new InvalidOperationException();
}
catch
{
    Console.WriteLine("caught");
}
finally
{
    Console.WriteLine("finally");
}
```

## Practical Tasks

- Design an exception policy for a three-layer service.
- Compare `throw;` and `throw exception;` by inspecting stack traces.
- Refactor routine validation exceptions into a Try-pattern result.
- Translate a low-level failure while preserving its inner exception.

## Readiness Criteria

You should be able to choose exception versus result semantics, catch at recovery boundaries, preserve stack traces and context, and design safe public error behavior.

## References

### Microsoft Learn

- [Exception handling](https://learn.microsoft.com/dotnet/csharp/fundamentals/exceptions/)
- [Exception-handling statements](https://learn.microsoft.com/dotnet/csharp/language-reference/statements/exception-handling-statements)
- [Best practices for exceptions](https://learn.microsoft.com/dotnet/standard/exceptions/best-practices-for-exceptions)
- [Create and throw exceptions](https://learn.microsoft.com/dotnet/standard/exceptions/how-to-create-and-throw-exceptions)
