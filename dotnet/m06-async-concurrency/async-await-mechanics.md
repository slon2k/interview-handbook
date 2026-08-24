# How `async`/`await` Work Conceptually

## Definition

`async`/`await` is syntax that lets the compiler rewrite a method into a state machine: at each `await`, if the awaited operation isn't already complete, the method returns control to its caller immediately, and the rest of the method runs later as a continuation when the operation completes. Critically, `async` does not mean "runs on a new thread" — it means "this method may suspend and resume without blocking the calling thread."

```csharp
public async Task<string> GetUserNameAsync(int id)
{
    var user = await _repository.GetUserAsync(id); // suspension point
    return user.Name;
}
```

## Alternatives & Trade-offs

Writing the equivalent logic by hand with raw `Task.ContinueWith` chains is possible but far less readable and much easier to get wrong (exception handling, `SynchronizationContext` capture, cancellation all become manual). `async`/`await` trades a small amount of "magic" (the compiler-generated state machine) for code that reads almost like synchronous code while behaving asynchronously.

## How It Works

### Why `async` doesn't necessarily create a thread

```csharp
public async Task<string> DownloadAsync(string url)
{
    return await _httpClient.GetStringAsync(url); // no new thread created here
}
```

The underlying I/O operation (a network call) is handled by the OS asynchronously; no thread is dedicated to "waiting." The calling thread is released back to the thread pool (or, in a UI app, back to the message loop) the moment `await` hits an incomplete task, and a callback resumes the method later — often on a thread-pool thread, not necessarily the original one, unless a `SynchronizationContext` says otherwise.

### The state machine, conceptually

```csharp
// What you write:
public async Task<int> AddAsync(int a, int b)
{
    await Task.Delay(10);
    return a + b;
}

// Roughly what the compiler generates (simplified):
// A struct implementing IAsyncStateMachine with a MoveNext() method,
// tracking which "step" the method is on and resuming from there
// when the awaited Task completes, via a continuation callback.
```

You don't need to write this by hand, but knowing it exists explains why local variables "survive" across an `await` (they become fields of the generated state machine) and why exceptions thrown inside an async method are captured and rethrown at the `await` point rather than immediately.

### Synchronous completion is still possible

```csharp
public async Task<int> GetValueAsync()
{
    if (_cache.TryGetValue("key", out var value))
        return value; // no actual suspension happens; the returned Task is already completed
    return await LoadAsync();
}
```

## Application

Use `async`/`await` for any method that performs I/O or otherwise needs to avoid blocking a thread while waiting. Understanding the state machine model helps explain non-obvious behavior: why exceptions surface at `await`, why an `async void` method can't be awaited (see `exception-handling-in-async-code.md`), and why the thread running the continuation may differ from the thread that started the method.

## Common Mistakes

- Assuming every `async` method spins up a new thread — most I/O-bound async code uses zero dedicated threads while waiting.
- Assuming the code after an `await` always resumes on the same thread it started on — true only in contexts with a capturing `SynchronizationContext` (like a UI thread) or when `ConfigureAwait(false)` isn't used.
- Forgetting that variables local to an async method persist across `await` points because they live in the compiler-generated state machine, which can matter for closures created inside the method.
- Treating `async` as a keyword that changes what a method *does*, rather than as a keyword that changes how it's *compiled* to support suspension.

## Common Interview Questions

### Basic
- Does `async` create a new thread?
- What does the `await` keyword actually do at a suspension point?

### Intermediate
- What happens to the calling thread when an `await` hits an incomplete `Task`?
- Can an async method complete synchronously? Give an example.

### Advanced
- Conceptually, how does the compiler transform an `async` method into a state machine?
- Why do local variables in an async method "survive" across `await` calls?
- What determines which thread resumes execution after an `await`?

### Follow-up Questions
- Is it possible for an `async` method to never actually suspend?
- What's the difference between a method that returns `Task` without `async` and one that's marked `async Task`?

### Code Prediction
```csharp
Console.WriteLine($"Before: {Thread.CurrentThread.ManagedThreadId}");
await Task.Delay(100);
Console.WriteLine($"After: {Thread.CurrentThread.ManagedThreadId}");
```
In a console app, are the two thread IDs guaranteed to be the same? What about in a UI application with a synchronization context?

## Practical Tasks

- Write an async method that sometimes completes synchronously and sometimes asynchronously, and log the thread ID before and after each `await` to observe the difference.
- Explain, without running it, why a `for` loop variable captured before an `await` behaves differently from one captured in a closure passed to `Task.Run`.
- Trace through the conceptual state-machine steps for a two-`await` async method on paper.

## Readiness Criteria

Explain precisely why `async` doesn't imply a new thread, describe the state-machine model at a conceptual level, and predict which thread resumes execution after an `await` in different hosting contexts.

## References

### Microsoft Learn

- [Async in depth](https://learn.microsoft.com/dotnet/standard/async-in-depth)
- [Asynchronous programming with async and await](https://learn.microsoft.com/dotnet/csharp/asynchronous-programming/)
