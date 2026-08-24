# Exception Handling

## Definition

Exceptions represent failures or unusual conditions that interrupt normal control flow. C# uses `try`, `catch`, `finally`, `throw`, and exception filters to handle them.

```csharp
try
{
    Process(input);
}
catch (FormatException exception)
{
    logger.LogWarning(exception, "Invalid input");
}
finally
{
    ReleaseResource();
}
```

## Alternatives & Trade-offs

Use exceptions for exceptional failures, not ordinary expected branching. Return a result, nullable value, or `Try` pattern when failure is common and part of normal control flow.

Exceptions preserve rich diagnostic information but are more expensive than ordinary branches and can obscure ownership if caught too broadly.

## How It Works

When an exception is thrown, the runtime searches the call stack for a compatible handler. Stack unwinding runs applicable `finally` blocks. A bare `throw;` preserves the original stack trace; `throw exception;` resets the throw location.

```csharp
catch (Exception)
{
    throw;
}
```

Exception filters can decide whether a handler applies without changing the exception state.

```csharp
catch (HttpRequestException ex) when (ex.StatusCode == HttpStatusCode.NotFound)
{
    return null;
}
```

## Application

- Translate infrastructure failures at application boundaries.
- Add context and preserve the original exception as an inner exception.
- Use `finally` or `using` for cleanup.
- Define domain-specific exceptions only when callers need to distinguish a meaningful condition.
- Log exceptions at an ownership boundary, avoiding duplicate logs.

## Common Mistakes

- Catching `Exception` and continuing as if nothing happened.
- Losing the stack trace with `throw exception;`.
- Using exceptions for routine validation or parsing.
- Catching and logging the same exception at every layer.
- Throwing `null` or using exceptions without meaningful context.
- Catching cancellation as a generic failure.
- Returning sensitive data in exception messages.

## Common Interview Questions

### Basic

- What is an exception?
- What is the purpose of `try`, `catch`, and `finally`?
- What is the difference between `throw;` and `throw ex;`?
- What does `using` have to do with exceptions?

### Intermediate

- How are multiple catch blocks selected?
- What is an inner exception?
- When should you create a custom exception?
- What is an exception filter?

### Advanced

- How does stack unwinding execute nested `finally` blocks?
- What are the performance costs of throwing exceptions?
- How should exception boundaries be designed across layers?
- How does cancellation differ from failure in asynchronous code?
- How do exception filters affect logging and state?
- How should exceptions be handled in parallel or aggregate operations?
- What information should be preserved when translating exceptions?
- How do first-chance exceptions differ from unhandled exceptions?
- How should retry logic distinguish transient from permanent failures?
- What are the compatibility implications of changing exception types in a public API?

### Follow-up Questions

- Does `finally` execute when a catch block returns?
- Why should cleanup code avoid throwing?
- What is `InnerException` used for?
- Should you catch `OperationCanceledException`?
- What is the difference between handled and unhandled exceptions?

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

### Boundary Translation

Translate a low-level file exception into a domain-specific failure while preserving the original exception.

### Stack Trace Review

Demonstrate the difference between `throw;` and `throw exception;` and inspect their stack traces.

### Retry Policy

Design a retry policy that handles transient errors without retrying validation or cancellation failures.

## Readiness Criteria

You should be able to explain propagation and stack traces, choose between exceptions and result patterns, preserve diagnostic context, handle cleanup, and design sensible error boundaries.

## References

### Microsoft Learn

- [Exception handling](https://learn.microsoft.com/dotnet/csharp/fundamentals/exceptions/)
- [Exception-handling statements](https://learn.microsoft.com/dotnet/csharp/language-reference/statements/exception-handling-statements)
- [Best practices for exceptions](https://learn.microsoft.com/dotnet/standard/exceptions/best-practices-for-exceptions)
- [Creating and throwing exceptions](https://learn.microsoft.com/dotnet/standard/exceptions/how-to-create-and-throw-exceptions)
