# `Thread` and `ThreadPool`

## Definition

A `Thread` is an OS-level unit of execution you can create and manage directly. The `ThreadPool` is a shared, managed pool of worker threads that .NET reuses across many short-lived work items instead of paying the cost of creating a new OS thread each time. `Task` and async/await are built on top of the thread pool by default.

```csharp
var thread = new Thread(() => Console.WriteLine("On a dedicated thread"));
thread.Start();

ThreadPool.QueueUserWorkItem(_ => Console.WriteLine("On a pooled thread"));
```

## Alternatives & Trade-offs

Creating a dedicated `Thread` gives full control (priority, background/foreground, a long-lived identity) but is expensive to create and destroy and doesn't scale to many concurrent short operations. The thread pool amortizes that cost by reusing a bounded set of threads, which is why `Task.Run` and async continuations use it by default — but pool threads shouldn't be blocked for long periods, since that reduces the pool's capacity to serve other work.

## How It Works

### Thread pool starvation

```csharp
// Blocking many pool threads at once can starve the pool of workers for other queued work
Parallel.For(0, 200, i => Thread.Sleep(5000)); // each iteration occupies a pool thread for 5 seconds
```

If enough pool threads are blocked simultaneously, the pool has to grow (slowly, with a delay per new thread) or other queued work items wait, which can look like unrelated parts of the application "hanging" under load.

### Dedicated thread for a genuinely long-lived task

```csharp
var thread = new Thread(RunBackgroundLoop) { IsBackground = true };
thread.Start();

void RunBackgroundLoop()
{
    while (true) { /* long-running, dedicated work */ }
}
```

A dedicated, long-running loop is one of the few cases where a raw `Thread` (rather than a pooled one) is appropriate — it shouldn't compete with short-lived work items for pool capacity.

### `Task.Run` uses the thread pool

```csharp
await Task.Run(() => ComputeExpensiveHash(buffer)); // runs on a thread-pool thread, returned when done
```

## Application

Use the thread pool (via `Task`/`Task.Run`) for short-to-medium CPU-bound work. Use a dedicated `Thread` only for genuinely long-running, continuously-active work that shouldn't share the pool with other short-lived tasks (e.g., a dedicated processing loop, or work requiring specific thread priority or apartment state).

## Common Mistakes

- Blocking many thread-pool threads simultaneously (e.g., via synchronous I/O or `.Result`/`.Wait()`), causing thread-pool starvation that affects unrelated parts of the application.
- Creating dedicated `Thread` objects for short-lived work, paying unnecessary creation/teardown cost that the pool exists specifically to avoid.
- Assuming the thread pool has unlimited or instantly-scaling capacity — it grows gradually, with a deliberate delay to avoid over-provisioning threads in response to short bursts.
- Confusing "asynchronous" with "runs on the thread pool" — many async I/O operations use zero threads while waiting, not a thread-pool thread.

## Common Interview Questions

### Basic
- What is the difference between a `Thread` and a thread-pool thread?
- Why does .NET use a thread pool instead of creating a new thread for every task?

### Intermediate
- What is thread-pool starvation, and what commonly causes it?
- When would you use a dedicated `Thread` instead of `Task.Run`?

### Advanced
- How does blocking synchronous I/O on pool threads affect the throughput of an ASP.NET Core application under load?
- How does the thread pool decide when to grow, and why is that growth deliberately gradual?

### Follow-up Questions
- Does every `await` resume on a thread-pool thread?
- What does `IsBackground` control on a `Thread`?

### Code Prediction
Given a web application handling 500 concurrent requests, each of which calls a synchronous, blocking database driver method (no async support) that takes 2 seconds, what happens to the application's ability to handle additional incoming requests during that time?

## Practical Tasks

- Reproduce thread-pool starvation by blocking many pool threads concurrently and observing delayed execution of other queued work.
- Identify, in a hypothetical codebase, a case where a dedicated `Thread` would be more appropriate than `Task.Run`.
- Explain the throughput difference between synchronous and asynchronous I/O under concurrent load in terms of thread-pool usage.

## Readiness Criteria

Explain the relationship between `Task`, `Task.Run`, and the thread pool, diagnose thread-pool starvation from a described symptom, and know when a dedicated thread is the right tool instead of the pool.

## References

### Microsoft Learn

- [The managed thread pool](https://learn.microsoft.com/dotnet/standard/threading/the-managed-thread-pool)
- [Thread class](https://learn.microsoft.com/dotnet/api/system.threading.thread)
