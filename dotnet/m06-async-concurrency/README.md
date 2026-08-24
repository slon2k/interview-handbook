# Module 6 - Asynchronous Programming and Concurrency

**Status:** Complete
**Priority:** Critical
**Prerequisites:** [C# Language and Type System](../m02-csharp-language/README.md)

## Scope

This module covers `async`/`await`, `Task`-based programming, thread-pool behavior, and the synchronization primitives needed to write correct concurrent code. It is a frequent differentiator between junior and mid-level candidates — most candidates can use `await`, but far fewer can explain what it actually does, why a given piece of code deadlocks, or how to safely bound concurrent work against a real dependency.

The focus is on precise mental models (what actually happens at an `await`, why races and deadlocks occur) over API trivia.

## Learning Outcomes

By the end of this module, you should be able to:

- Distinguish synchronous/asynchronous execution and concurrency/parallelism, and classify work as I/O-bound or CPU-bound.
- Explain why `async` does not imply a new thread, at a conceptual state-machine level.
- Correctly shape async code as sequential or concurrent using `Task.WhenAll`/`Task.WhenAny`.
- Handle exceptions and cancellation correctly in async code, and explain why `async void` is dangerous outside event handlers.
- Diagnose and fix race conditions and deadlocks, including the classic synchronization-context deadlock.
- Choose the correct synchronization primitive (`lock`, `Interlocked`, `SemaphoreSlim`) and concurrent collection for a given scenario.
- Implement bounded concurrency against a rate- or capacity-limited dependency.

## Topics

### 1. Foundations

- [Synchronous vs. asynchronous, concurrency vs. parallelism](sync-vs-async-and-concurrency-fundamentals.md)
- [`Task` and `Task<T>`](task-and-task-of-t.md)
- [How `async`/`await` work conceptually](async-await-mechanics.md)

### 2. Shaping and Handling Async Work

- [`Task.WhenAll`, `Task.WhenAny`, and sequential vs. concurrent execution](combinators-and-execution-shape.md)
- [Exception propagation and `async void`](exception-handling-in-async-code.md)
- [Cancellation tokens and timeouts](cancellation-and-timeouts.md)
- [`IAsyncEnumerable<T>` and async streams](iasyncenumerable-and-async-streams.md)
- [Async disposal](async-disposal.md)

### 3. Threads and Synchronization

- [`Thread` and `ThreadPool`](threads-and-threadpool.md)
- [Race conditions, deadlocks, and shared mutable state](race-conditions-and-deadlocks.md)
- [`lock`, `Monitor`, `Interlocked`, and `SemaphoreSlim`](locks-and-synchronization-primitives.md)
- [Concurrent collections](concurrent-collections.md)
- [Synchronization context and `ConfigureAwait`](synchronization-context-and-configureawait.md)
- [Bounded concurrency](bounded-concurrency.md)

## Scope Boundaries

- Basic C# generics, delegates, and value semantics belong in [Module 2 - C# Language and Type System](../m02-csharp-language/README.md).
- Collection types themselves (as opposed to their thread-safety) belong in [Module 3 - Collections, LINQ, and Basic Algorithms](../m03-collections-linq/README.md); concurrent collections are documented here since their defining feature is thread-safety.
- ASP.NET Core request cancellation, `HttpClientFactory`, and background services belong in Module 7.
- Detailed EF Core `DbContext` thread-safety and concurrency conflict handling belong in Module 9.

## Suggested Learning Sequence

1. Synchronous vs. asynchronous, concurrency vs. parallelism, and I/O- vs. CPU-bound classification.
2. `Task`/`Task<T>` and the conceptual mechanics of `async`/`await`.
3. Shaping execution with `Task.WhenAll`/`Task.WhenAny`, exception handling, cancellation, and async streams.
4. Threads and the thread pool.
5. Race conditions, deadlocks, and the synchronization primitives that prevent them.
6. Synchronization context, `ConfigureAwait`, and bounded concurrency.

## Practical Deliverables

- Refactor a sequence of independently-awaited async calls into a concurrent `Task.WhenAll`-based version and measure the time difference.
- Reproduce a lost-update race condition and fix it with `Interlocked` or a lock.
- Reproduce the classic synchronization-context deadlock and fix it by awaiting instead of blocking.
- Implement bounded concurrency for a batch operation against a simulated rate-limited API.
- Build a small `IAsyncEnumerable<T>`-based streaming method with correct cancellation support.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and API familiarity.
- Intermediate questions involving common usage and trade-offs.
- Advanced questions involving runtime behavior, deadlocks, and performance under load.
- Follow-up questions that test precise understanding rather than memorized definitions.
- Code-prediction questions — this module produces some of the most common "what does this print / does this deadlock" interview questions.

## References

### Microsoft Learn

- [Asynchronous programming in C#](https://learn.microsoft.com/dotnet/csharp/asynchronous-programming/)
- [Async in depth](https://learn.microsoft.com/dotnet/standard/async-in-depth)
- [Threading](https://learn.microsoft.com/dotnet/standard/threading/)
- [Overview of synchronization primitives](https://learn.microsoft.com/dotnet/standard/threading/overview-of-synchronization-primitives)
- [Thread-safe collections](https://learn.microsoft.com/dotnet/standard/collections/thread-safe/)
