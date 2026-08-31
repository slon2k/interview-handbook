# Debugging Broken Code

## What This Assesses

Given code that compiles but behaves incorrectly (or throws), can you form a hypothesis, verify it, and fix the root cause rather than a surface symptom? This exercise type specifically probes whether you actually understand the underlying mechanics (closures, async, nullability) rather than having only memorized correct patterns.

## Format and Time Expectations

A short snippet (10-30 lines) with a described symptom ("this always prints the same value," "this occasionally throws under load") — talk through your diagnosis process, not just the fix.

## Exercise 1: The Classic Closure-Capture Bug

**Problem:** This is meant to print 0, 1, 2 but prints 3, 3, 3. Why, and how would you fix it?

```csharp
var actions = new List<Action>();
for (int i = 0; i < 3; i++)
{
    actions.Add(() => Console.WriteLine(i));
}
foreach (var action in actions) action();
```

**What a strong answer demonstrates:** Correctly explaining that all three lambdas capture the *same* variable `i` by reference, not its value at the time of capture — by the time any lambda runs, the loop has already finished and `i` is 3. (Note: in modern C#, `for` loop variables are scoped per-iteration differently than `foreach` — worth knowing this specific example's behavior depends on the loop construct and language version, and being precise about which one you're diagnosing.) The fix: capture a local copy inside the loop body (`int local = i;`) before the lambda closes over it.

**Common mistakes:** Fixing it by trial and error (moving code around until the output looks right) without being able to articulate *why* the original version was wrong.

## Exercise 2: Silent Async Void Exception

**Problem:** This method is supposed to log an error if it fails, but nothing appears in the logs when it fails. Why?

```csharp
private async void ProcessOrder(Order order)
{
    await _repository.SaveAsync(order); // throws for a specific order
}
```

**What a strong answer demonstrates:** Identifying `async void` as the problem (Module 6) — there's no `Task` for any caller to observe, so the exception propagates to the `SynchronizationContext` (or crashes the process) rather than being catchable by normal means, and definitely isn't being logged by any try/catch that might exist around the *caller*. Fix: change to `async Task` if at all possible, or add an internal try/catch if this genuinely must be `async void` (an event handler).

**Common mistakes:** Adding a try/catch around the *caller's* invocation of `ProcessOrder`, which does nothing, since that's exactly the pattern `async void` breaks.

## Exercise 3: Deadlock

**Problem:** This WPF button-click handler hangs the UI. Why?

```csharp
private void Button_Click(object sender, EventArgs e)
{
    var result = GetDataAsync().Result;
}

private async Task<string> GetDataAsync()
{
    return await _httpClient.GetStringAsync(url);
}
```

**What a strong answer demonstrates:** Correctly diagnosing the classic synchronization-context deadlock (Module 6) — `.Result` blocks the UI thread, while `GetDataAsync`'s continuation is trying to resume on that same, now-blocked UI thread. Fix: make `Button_Click` `async void` (acceptable here specifically because it's a genuine event handler) and `await GetDataAsync()` instead of blocking.

**Common mistakes:** Suggesting `ConfigureAwait(false)` as "the fix" without explaining that the *real* fix is not blocking on async code at all, and that `ConfigureAwait(false)` only reduces (doesn't eliminate) this specific risk pattern.

## Exercise 4: NullReferenceException from a Silent Binding Failure

**Problem:** `GET /orders/abc` (an invalid ID) returns a `200 OK` with unexpected data instead of an error, and later code throws `NullReferenceException`. Why, given this controller?

```csharp
[HttpGet("{id}")]
public IActionResult Get(int id) // no [ApiController], no route constraint
{
    var order = _repository.GetById(id);
    return Ok(order.CustomerName); // throws if order is null
}
```

**What a strong answer demonstrates:** Recognizing that without `[ApiController]`'s automatic model-validation `400` (Module 8) and without a route constraint (`{id:int}`), a non-numeric `id` silently binds to `0` rather than producing an error — the method then runs with `id = 0`, likely finding no matching order, and the resulting `null` causes the later crash. Fix: add a route constraint and/or `[ApiController]`, and a null check before accessing `order.CustomerName`.

**Common mistakes:** Only adding a null check without addressing the upstream silent-binding-failure root cause, fixing the symptom but leaving the actual design gap.

## Readiness Criteria

For each bug, state a specific, falsifiable hypothesis before proposing a fix, explain the underlying mechanism precisely (not just "add a try/catch" or "it just works now"), and identify the root cause rather than only the symptom.

## References

- [Delegates, lambdas and closures (Module 2)](../m02-csharp-language/delegates-lambdas-and-closures.md)
- [Exception propagation and async void (Module 6)](../m06-async-concurrency/exception-handling-in-async-code.md)
- [Synchronization context and ConfigureAwait (Module 6)](../m06-async-concurrency/synchronization-context-and-configureawait.md)
- [Model binding (Module 8)](../m08-aspnet-core-fundamentals/model-binding.md)
