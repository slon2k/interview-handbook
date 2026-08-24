# Exception Propagation and `async void`

## Definition

Exceptions thrown inside an `async Task` or `async Task<T>` method are captured and stored on the returned `Task`, then rethrown when that task is awaited. Exceptions thrown inside an `async void` method cannot be captured this way — there's no `Task` to observe — so they're instead raised directly on the `SynchronizationContext` that was active when the method started, which usually means crashing the process.

```csharp
public async Task RiskyAsync() { throw new InvalidOperationException(); }
// caller:
try { await RiskyAsync(); }
catch (InvalidOperationException) { /* caught normally */ }

public async void RiskyVoidAsync() { throw new InvalidOperationException(); }
RiskyVoidAsync(); // exception cannot be caught by the caller at all — it surfaces elsewhere, often crashing the app
```

## Alternatives & Trade-offs

`async Task` gives you normal, catchable exception propagation and is almost always the right choice. `async void` exists specifically for event handlers, which must have a `void` signature to satisfy the delegate type they implement — outside of that one case, `async void` should be avoided.

## How It Works

### Why `async void` exceptions can't be caught normally

```csharp
private async void Button_Click(object sender, EventArgs e)
{
    await SomethingAsync();
    throw new InvalidOperationException(); // cannot be try/caught by the event-raising code
}
```

Since there's no `Task` for the caller to hold and await, the exception has nowhere to be stored — it propagates immediately on whatever `SynchronizationContext` was captured, typically terminating the application rather than being observable by ordinary exception handling.

### Correct pattern: `async void` only for event handlers, with an internal try/catch

```csharp
private async void Button_Click(object sender, EventArgs e)
{
    try
    {
        await SomethingAsync();
    }
    catch (Exception ex)
    {
        _logger.LogError(ex, "Button click handler failed");
    }
}
```

### Exception propagation through multiple `await`s

```csharp
public async Task ProcessAsync()
{
    await StepOneAsync();
    await StepTwoAsync(); // if StepOneAsync throws, this line never runs — normal try/catch semantics apply
}
```

### `Task.WhenAll` and multiple exceptions

As covered in `combinators-and-execution-shape.md`, `await Task.WhenAll(...)` only rethrows the first exception; the rest remain attached to their respective tasks unless explicitly inspected.

## Application

Always use `async Task`/`async Task<T>` for your own async methods. The only legitimate use of `async void` is a UI or framework event handler whose signature is fixed by a `void`-returning delegate type — and even then, wrap the body in a try/catch so failures are handled deliberately rather than crashing the app.

## Common Mistakes

- Writing `async void` methods "because there's no return value," when `async Task` (with no meaningful result) is almost always available and safe instead.
- Assuming a try/catch around a fire-and-forget `async void` call will catch its exceptions — it cannot.
- Not wrapping the body of a legitimate `async void` event handler in its own try/catch, leaving it to crash the process on any unhandled exception.
- Forgetting that an *unobserved* faulted `Task` (one whose exception is never awaited or inspected) can, depending on .NET version and configuration, go unnoticed or surface later via the `TaskScheduler.UnobservedTaskException` event.

## Common Interview Questions

### Basic
- Why can't you catch an exception thrown by an `async void` method the normal way?
- When, if ever, is `async void` acceptable?

### Intermediate
- What happens to an exception thrown inside an `async Task` method that's never awaited?
- How should an `async void` event handler handle its own exceptions?

### Advanced
- How does `await Task.WhenAll` behave when multiple tasks fail, and how would you observe every exception rather than just the first?
- What is `TaskScheduler.UnobservedTaskException`, and what scenario does it help diagnose?

### Follow-up Questions
- Does wrapping a call to an `async void` method in try/catch do anything useful?
- Is `async Task` always preferable to `async void`, or are there other legitimate exceptions?

### Code Prediction
```csharp
public async void FireAndForget()
{
    await Task.Delay(10);
    throw new Exception("boom");
}

try
{
    FireAndForget();
    Console.WriteLine("Done");
}
catch (Exception)
{
    Console.WriteLine("Caught");
}
```
What gets printed, and does the exception ever reach the `catch` block?

## Practical Tasks

- Convert an `async void` method that isn't an event handler into `async Task`, and update its callers accordingly.
- Add a defensive try/catch to a UI event handler and log failures instead of letting them crash the app.
- Write code demonstrating that `await Task.WhenAll` only surfaces the first of several exceptions, then fix it to observe all of them.

## Readiness Criteria

Explain precisely why `async void` breaks normal exception handling, correctly restrict its use to event handlers with internal error handling, and reason about exception propagation through chained `await`s and combinators.

## References

### Microsoft Learn

- [Async in depth — exception handling](https://learn.microsoft.com/dotnet/standard/async-in-depth)
- [Task.Exception property](https://learn.microsoft.com/dotnet/api/system.threading.tasks.task.exception)
