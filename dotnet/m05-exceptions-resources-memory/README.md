# Module 5 - Exceptions, Resources, and Memory Management

**Status:** Complete  
**Priority:** High  
**Prerequisites:** [C# Language and Type System](../m02-csharp-language/README.md)

## Scope

This module covers failure handling, deterministic resource cleanup, managed memory behavior, and practical allocation awareness in .NET applications.

The focus is on choosing appropriate error and resource-management strategies, understanding runtime behavior, and diagnosing common leaks or allocation problems. Deep GC tuning and runtime implementation details are optional advanced material.

## Learning Outcomes

By the end of this module, you should be able to:

- Design exception boundaries and preserve useful diagnostic context.
- Distinguish exceptional failures from expected validation or result states.
- Dispose managed and unmanaged resources deterministically.
- Explain `IDisposable`, `IAsyncDisposable`, `using`, and finalizers.
- Describe GC roots, generations, the Large Object Heap, and common managed leaks.
- Recognize allocation and boxing costs in ordinary application code.
- Explain when `Span<T>` and `Memory<T>` are appropriate at an awareness level.

## Topics

- [Exception handling and design](exception-handling-and-design.md)
- [Resource management and disposal](resource-management-and-disposal.md)
- [Garbage collection and object lifetime](gc-and-object-lifetime.md)
- [Boxing and allocation pressure](boxing-and-allocation-pressure.md)
- [Span, memory, pooling, and measurement](span-memory-and-pooling.md)
- [Memory diagnostics and performance review](memory-diagnostics-and-performance.md)

## Scope Boundaries

- Basic exception syntax is covered in [C# exception handling](../m02-csharp-language/exception-handling.md).
- Async exception propagation, cancellation, synchronization, and bounded concurrency belong in [Module 6 - Asynchronous Programming and Concurrency](../m06-async-concurrency/README.md).
- ASP.NET Core exception middleware, `ProblemDetails`, and HTTP error contracts belong in Module 7.
- Database transaction behavior belongs in Modules 8 and 9.
- Detailed performance investigation and observability belong in Module 12.

## Suggested Learning Sequence

1. Exception handling, propagation, boundaries, and API design.
2. Resource ownership, `IDisposable`, `using`, async disposal, and finalizers.
3. GC roots, generations, object lifetime, and managed leaks.
4. Boxing, temporary allocations, and allocation pressure.
5. Spans, memory, pooling, and lifetime restrictions.
6. Memory diagnostics, benchmarking, and performance review.

## Practical Deliverables

- Design an exception policy for a layered application.
- Implement a disposable wrapper with clear ownership.
- Review code for missing disposal and event-handler retention.
- Demonstrate a managed memory leak caused by a long-lived publisher.
- Compare allocation behavior before and after removing unnecessary boxing.
- Use a profiler or benchmark to investigate allocation pressure.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and runtime behavior.
- Intermediate questions involving API design and resource ownership.
- Advanced questions involving GC behavior, leaks, allocation pressure, and diagnostics.
- Follow-up questions that test failure-handling judgment.
- Code-prediction or code-review questions involving propagation, disposal, and lifetime.

## References

### Microsoft Learn

- [Exception handling](https://learn.microsoft.com/dotnet/csharp/fundamentals/exceptions/)
- [Best practices for exceptions](https://learn.microsoft.com/dotnet/standard/exceptions/best-practices-for-exceptions)
- [Implement a Dispose method](https://learn.microsoft.com/dotnet/standard/garbage-collection/implementing-dispose)
- [Using objects that implement IDisposable](https://learn.microsoft.com/dotnet/standard/garbage-collection/using-objects)
- [Fundamentals of garbage collection](https://learn.microsoft.com/dotnet/standard/garbage-collection/fundamentals)
- [Large Object Heap](https://learn.microsoft.com/dotnet/standard/garbage-collection/large-object-heap)
- [Memory and span usage guidelines](https://learn.microsoft.com/dotnet/standard/memory-and-spans/)
