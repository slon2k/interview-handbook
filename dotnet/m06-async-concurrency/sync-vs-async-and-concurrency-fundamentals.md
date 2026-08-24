# Synchronous vs. Asynchronous, Concurrency vs. Parallelism

## Definition

**Synchronous** code executes one step at a time, blocking the calling thread until each step completes. **Asynchronous** code starts an operation and returns control to the caller before it finishes, resuming later when it's done. **Concurrency** means multiple tasks make progress over overlapping time periods (not necessarily simultaneously); **parallelism** means multiple tasks literally execute at the same instant on different CPU cores. Async/await is primarily a concurrency tool, not a parallelism tool.

```csharp
// Synchronous: this thread is blocked for the whole download
string html = client.DownloadString(url);

// Asynchronous: this thread is free while the download is in flight
string html = await client.GetStringAsync(url);
```

## Alternatives & Trade-offs

Synchronous code is simpler to reason about but wastes a thread for the duration of any I/O wait — expensive in a server handling many concurrent requests, where threads are a limited pool resource. Asynchronous code frees the thread during I/O waits at the cost of more complex control flow (exception propagation, cancellation, ordering) and a coding style that reads slightly less linearly than synchronous code.

## How It Works

### I/O-bound vs. CPU-bound work

```csharp
// I/O-bound: waiting on the network. The thread is idle, not busy — async helps a lot here.
await httpClient.GetStringAsync(url);

// CPU-bound: doing actual work (parsing, computing). The thread is busy the whole time —
// async alone doesn't help; you'd need Task.Run to move it to another thread, which is
// about parallelism/offloading, not about avoiding a wait.
var hash = ComputeExpensiveHash(largeBuffer);
```

Async shines for I/O-bound work because the thread has nothing useful to do while waiting anyway. For CPU-bound work, async doesn't make the computation faster — it only decides which thread does it.

### Concurrency without parallelism

```csharp
var task1 = client.GetStringAsync(url1);
var task2 = client.GetStringAsync(url2);
await Task.WhenAll(task1, task2);
```

On a single CPU core, this is still "concurrent" — both requests are in flight at once from the application's perspective — even though only one CPU is doing any work at any given instant (the rest is I/O wait time managed by the OS).

## Application

Use `async`/`await` for anything that waits on I/O: HTTP calls, database queries, file access, message queues. Use actual parallelism (`Parallel.For`, `Task.Run` combined with multiple cores, PLINQ) for CPU-bound work that can be split across cores. Don't reach for `async` expecting it to speed up a CPU-bound loop — it won't, by itself.

## Common Mistakes

- Assuming `async` makes code run in parallel or makes CPU-bound work faster.
- Wrapping CPU-bound work in `Task.Run` inside an ASP.NET Core request handler purely to "make it async," which just moves the busy work to a different thread pool thread without actually reducing total work done.
- Confusing "asynchronous" with "runs on a background thread" — many async I/O operations complete without occupying any thread at all while waiting (see async-await-mechanics.md).
- Treating concurrency and parallelism as synonyms in an interview answer — a common way to lose credibility on this topic quickly.

## Common Interview Questions

### Basic
- What's the difference between synchronous and asynchronous execution?
- What's the difference between concurrency and parallelism?

### Intermediate
- Why does async help I/O-bound work but not CPU-bound work?
- Can code be concurrent without being parallel? Give an example.

### Advanced
- Why doesn't wrapping CPU-bound work in `Task.Run` inside a web request handler improve server throughput?
- How would you decide whether a given workload benefits more from async I/O or from parallel processing?

### Follow-up Questions
- Is a single-core machine capable of concurrency?
- Does `await` block the calling thread?

### Code Prediction
```csharp
async Task<int> ComputeAsync()
{
    return ExpensiveCpuWork(); // no actual I/O, no Task.Run
}
```
Does marking this method `async` make `ExpensiveCpuWork()` run on a different thread or run any faster? Why or why not?

## Practical Tasks

- Classify a list of operations (HTTP call, database query, JSON parsing, file read, image resizing) as I/O-bound or CPU-bound, and justify whether `async` alone helps each one.
- Measure server thread usage under synchronous vs. asynchronous I/O calls under load.
- Write a small example demonstrating concurrency without parallelism (two async I/O operations awaited together on a single logical thread).

## Readiness Criteria

Explain concurrency vs. parallelism precisely, correctly classify work as I/O-bound or CPU-bound, and explain why that classification determines whether async or explicit parallelism is the right tool.

## References

### Microsoft Learn

- [Asynchronous programming in C#](https://learn.microsoft.com/dotnet/csharp/asynchronous-programming/)
- [Async in depth](https://learn.microsoft.com/dotnet/standard/async-in-depth)
