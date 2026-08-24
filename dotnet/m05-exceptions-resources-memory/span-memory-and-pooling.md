# Span, Memory, Pooling, and Measurement

## Definition

`Span<T>` and `ReadOnlySpan<T>` provide stack-only views over contiguous memory. `Memory<T>` and `ReadOnlyMemory<T>` can be stored and passed across asynchronous boundaries. Pools reuse buffers to reduce allocation.

## Alternatives & Trade-offs

Use arrays, strings, and ordinary collections by default. Use spans, memory, or pools for measured allocation-sensitive paths because their lifetime and ownership rules are stricter.

## How It Works

A span is a `ref struct`, so it cannot be stored in a heap object, captured by a lambda, or used across `await` or `yield`. `Memory<T>` can cross those boundaries.

```csharp
ReadOnlySpan<char> prefix = text.AsSpan(0, 3);
```

`ArrayPool<T>.Shared.Rent` returns a buffer that may be larger than requested. The owner must return it in a `finally` block and must not expose it afterward.

```csharp
byte[] buffer = ArrayPool<byte>.Shared.Rent(4096);
try
{
    Process(buffer);
}
finally
{
    ArrayPool<byte>.Shared.Return(buffer, clearArray: true);
}
```

## Application

- Use spans for synchronous parsing and slicing without copying.
- Use memory types for asynchronous pipelines.
- Use pooling for large or frequent temporary buffers.
- Clear returned buffers when they may contain sensitive data.
- Compare allocation and latency before and after optimization.

## Common Mistakes

- Retaining a span beyond its source lifetime.
- Storing a span in a class field.
- Forgetting to return a rented buffer.
- Exposing a rented buffer after returning it.
- Assuming pooling is always faster.
- Ignoring data leakage through reused buffers.

## Common Interview Questions

### Basic

- What is `Span<T>`?
- Why use `Memory<T>`?
- What is `ArrayPool<T>`?

### Intermediate

- Why can a span not cross `await`?
- Who owns a rented buffer?
- When can pooling reduce GC pressure?

### Advanced

- How do spans reduce copying while preserving safety?
- How can pooling increase retention or leak sensitive data?
- How would you benchmark pooling against normal allocation?
- How do `Memory<T>` lifetime rules differ from `Span<T>`?
- When should a specialized memory optimization be rejected for clarity?

### Follow-up Questions

- Can a span be captured by a closure?
- Can `Memory<T>` be stored in a class?
- What happens if a pooled buffer is used after return?

### Code Prediction

Which type can be stored before an asynchronous operation: `Span<byte>` or `Memory<byte>`?

## Practical Tasks

- Rewrite a substring parser using `ReadOnlySpan<char>`.
- Implement rent/return cleanup with `ArrayPool<T>`.
- Compare allocation counts before and after pooling.
- Review a buffer API for ownership and lifetime leaks.

## Readiness Criteria

Explain span and memory lifetime restrictions, pooling ownership, sensitive-data handling, and why measurement must justify these optimizations.

## References

### Microsoft Learn

- [Memory and spans](https://learn.microsoft.com/dotnet/standard/memory-and-spans/)
- [System.Span<T>](https://learn.microsoft.com/dotnet/api/system.span-1)
- [System.Memory<T>](https://learn.microsoft.com/dotnet/api/system.memory-1)
- [ArrayPool<T>](https://learn.microsoft.com/dotnet/api/system.buffers.arraypool-1)
