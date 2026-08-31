# Async/Concurrency Problem Analysis

## What This Assesses

Given a piece of async or multithreaded code, can you predict its behavior precisely, identify a race condition, deadlock, or thread-pool issue, and propose the correct fix — the sharpest, most commonly-tested subset of Module 6's material.

## Format and Time Expectations

A short snippet (10-20 lines), with a prompt like "is this thread-safe?" or "what could go wrong here under load?" — precise reasoning about *why* matters more than just naming the right keyword.

## Exercise 1: Is This Thread-Safe?

**Problem:** Is this counter thread-safe under concurrent calls to `Increment`?

```csharp
public class Counter
{
    private int _count;
    public void Increment() => _count++;
    public int Value => _count;
}
```

**What a strong answer demonstrates:** Correctly identifying that `_count++` is not atomic (Module 6) — it's read, add, write as three separate steps — so two threads can both read the same value before either writes back, losing an increment. Fix: `Interlocked.Increment(ref _count)`.

**Common mistakes:** Assuming a single-line increment is automatically atomic because it "looks like one operation" in source code.

## Exercise 2: Sequential or Concurrent?

**Problem:** Are these two calls running sequentially or concurrently?

```csharp
var task1 = FetchDataAsync(url1);
var result1 = await task1;
var task2 = FetchDataAsync(url2);
var result2 = await task2;
```

**What a strong answer demonstrates:** Correctly identifying this as sequential (Module 6) despite using `Task`/`await` — `task1` is awaited (and therefore effectively completed) before `task2` is even started, so there's no overlap. Fix, if concurrency was actually intended: start both calls before awaiting either.

```csharp
var task1 = FetchDataAsync(url1);
var task2 = FetchDataAsync(url2); // started before task1 is awaited
var (result1, result2) = (await task1, await task2);
```

**Common mistakes:** Assuming any code using `async`/`await` is automatically concurrent, missing that the *order* of starting vs. awaiting is what actually determines this.

## Exercise 3: Diagnosing a Deadlock

**Problem:** This synchronous method calling into async code hangs when called from a WPF button-click handler, but works fine from a console app. Why the difference?

```csharp
public string GetData()
{
    return GetDataAsync().Result;
}
```

**What a strong answer demonstrates:** Explaining the synchronization-context deadlock (Module 6) — in WPF, the blocked calling thread (via `.Result`) is the same UI thread `GetDataAsync`'s continuation needs to resume on, causing a genuine deadlock. In a console app, there's no captured synchronization context, so the continuation can resume on any thread-pool thread — no deadlock, but `.Result` is still risky practice in general (masks exceptions as `AggregateException`, blocks a thread unnecessarily).

**Common mistakes:** Concluding "this code is just fine because it works in a console app," without recognizing that it's fragile and environment-dependent rather than actually correct.

## Exercise 4: Thread-Pool Starvation

**Problem:** A web API's throughput degrades severely under moderate load. Every endpoint calls a legacy library with no async support, using it directly (a blocking call) inside otherwise-async controller actions. What's likely happening, and what would you check?

**What a strong answer demonstrates:** Diagnosing thread-pool starvation (Module 6) — many concurrent requests each blocking a thread-pool thread on the synchronous legacy call, exhausting the pool's capacity to serve other requests. Check: `dotnet-counters` (Module 13) showing a growing thread-pool queue length. Fix: wrap the specific blocking call in `Task.Run` if the workload is genuinely CPU-bound and isolating it helps, or (better) find/build an async-capable path if the legacy call is actually I/O-bound underneath.

**Common mistakes:** Suggesting "just add more threads" or "increase the thread pool's minimum thread count" as the primary fix, treating the symptom rather than addressing why threads are being blocked unnecessarily in the first place.

## Readiness Criteria

Correctly identify race conditions, sequential-vs-concurrent execution shape, deadlocks, and thread-pool starvation from reading code alone, explain the precise mechanism behind each (not just the symptom), and propose the fix that addresses the actual root cause.

## References

- [Race conditions, deadlocks, and shared mutable state (Module 6)](../m06-async-concurrency/race-conditions-and-deadlocks.md)
- [Task.WhenAll, Task.WhenAny, and sequential vs. concurrent execution (Module 6)](../m06-async-concurrency/combinators-and-execution-shape.md)
- [Synchronization context and ConfigureAwait (Module 6)](../m06-async-concurrency/synchronization-context-and-configureawait.md)
- [Thread and ThreadPool (Module 6)](../m06-async-concurrency/threads-and-threadpool.md)
