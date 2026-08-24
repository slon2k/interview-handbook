# Resource Management and Disposal

## Definition

Managed resources are controlled by the GC. Unmanaged resources, such as OS handles, native memory, sockets, and file handles, require deterministic release through an owning wrapper.

`IDisposable` defines synchronous cleanup. `IAsyncDisposable` defines asynchronous cleanup.

## Alternatives & Trade-offs

Prefer managed abstractions and established wrappers such as `SafeHandle`. Use `using` or `await using` when ownership is local. Do not dispose an object owned by a longer-lived component.

## How It Works

A using declaration lowers to disposal logic similar to `try/finally`. Resources acquired in the same scope are disposed in reverse order.

```csharp
using StreamReader reader = File.OpenText(path);
string text = reader.ReadToEnd();
```

Asynchronous disposal uses `await using`:

```csharp
await using IAsyncDisposable resource = await OpenAsync();
```

Finalizers are a fallback for missed unmanaged cleanup and are nondeterministic. Safe handles reduce the need to write finalizers directly.

## Application

- Document who creates and disposes each resource.
- Dispose streams, commands, connections, and native wrappers at the ownership boundary.
- Use `SafeHandle` for native handles where possible.
- Use async disposal when cleanup performs asynchronous I/O.
- Keep disposal idempotent and safe during partial construction.

## Common Mistakes

- Assuming the GC releases external resources promptly.
- Returning a lazy operation over a disposed stream or context.
- Disposing a dependency that the caller does not own.
- Blocking on `DisposeAsync`.
- Assuming finalizers run promptly.
- Exposing pooled or native memory after returning it.

## Common Interview Questions

### Basic

- What is an unmanaged resource?
- What does `IDisposable` represent?
- What does `using` do?
- What is `IAsyncDisposable`?

### Intermediate

- In what order are nested resources disposed?
- What is the difference between `using` and `await using`?
- Why is `SafeHandle` preferred over a handwritten finalizer?
- How should ownership be documented?

### Advanced

- How do disposal and deferred execution interact?
- How can disposal be safe after partial construction or repeated calls?
- How does finalization affect GC latency?
- How should ownership transfer work across API boundaries?
- How do asynchronous cleanup and cancellation interact?

### Follow-up Questions

- Does `using` catch exceptions?
- Does disposal make an object unusable immediately?
- Who should dispose a dependency supplied through constructor injection?

### Code Prediction

Which resource is disposed first?

```csharp
using Stream first = OpenFirst();
using Stream second = OpenSecond();
```

## Practical Tasks

- Refactor nested `try`/`finally` blocks into using declarations.
- Implement a disposable wrapper with clear ownership.
- Review a lazy query that outlives its resource and fix its lifetime.
- Replace raw handle cleanup with a `SafeHandle`-based design.

## Readiness Criteria

You should be able to distinguish managed and unmanaged resources, define ownership, use synchronous and asynchronous disposal correctly, and explain why finalizers are only a fallback.

## References

### Microsoft Learn

- [Implement a Dispose method](https://learn.microsoft.com/dotnet/standard/garbage-collection/implementing-dispose)
- [Implement a DisposeAsync method](https://learn.microsoft.com/dotnet/standard/garbage-collection/implementing-disposeasync)
- [Using objects that implement IDisposable](https://learn.microsoft.com/dotnet/standard/garbage-collection/using-objects)
- [SafeHandle](https://learn.microsoft.com/dotnet/api/system.runtime.interopservices.safehandle)
