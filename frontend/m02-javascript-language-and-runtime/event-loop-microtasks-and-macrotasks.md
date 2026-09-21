# The Event Loop, Microtasks, and Macrotasks

## Definition

JavaScript runs synchronous code on a single call stack, one thing at a time. The event loop is what lets asynchronous work happen anyway: once the stack is empty, it processes queued callbacks — promise reactions go into the **microtask queue**, while timers and most browser events go into the **task queue** (commonly called the macrotask queue) — and microtasks are always fully drained before the next macrotask runs.

```javascript
console.log("1: sync");

setTimeout(() => console.log("2: setTimeout"), 0);

Promise.resolve().then(() => console.log("3: promise"));

console.log("4: sync");

// Output: 1, 4, 3, 2 — NOT 1, 2, 3, 4, and NOT 1, 4, 2, 3
```

## Alternatives & Trade-offs

There's no real alternative to this model within JavaScript — it's the platform's single-threaded execution design. The trade-off this topic actually illuminates is between microtasks and macrotasks as scheduling choices: microtasks (promises) run sooner, right after the current synchronous code finishes and before any rendering or timer callback, while macrotasks (timers, most events) wait their turn behind any pending microtasks — choosing the wrong one for a given need produces subtly wrong ordering.

## How It Works

### Why the output above is `1, 4, 3, 2`, step by step

```javascript
console.log("1: sync");                              // runs immediately — synchronous code always runs first
setTimeout(() => console.log("2: setTimeout"), 0);       // SCHEDULES a macrotask; does not run yet, even with 0ms delay
Promise.resolve().then(() => console.log("3: promise"));  // SCHEDULES a microtask; does not run yet either
console.log("4: sync");                                     // ALSO runs immediately — still synchronous code

// Call stack is now empty. Before taking the NEXT macrotask, the event loop drains ALL pending microtasks first:
// "3: promise" logs.
// ONLY NOW does the event loop take the next macrotask:
// "2: setTimeout" logs.
```

`setTimeout(fn, 0)` does not mean "run immediately" — it means "queue this as a macrotask, to run after at least 0ms AND after the current synchronous code and any pending microtasks have finished." Microtasks always drain completely before the next macrotask is taken, which is why the promise callback logs before the timer callback even though the timer was scheduled first in the source.

### `await` schedules a microtask for the code after it

```javascript
console.log("A");

async function example() {
  console.log("B");
  await null;               // pauses here, and everything AFTER this line becomes a microtask
  console.log("D");
}
example();

console.log("C");

// Output: A, B, C, D
```

The code before the first `await` inside an `async` function runs synchronously, immediately, when the function is called — it's only the code *after* an `await` that gets deferred as a microtask, resuming once the current synchronous code (here, `console.log("C")`) has finished.

### Long synchronous work blocks everything — rendering, timers, and promise callbacks alike

```javascript
function blockForOneSecond() {
  const start = Date.now();
  while (Date.now() - start < 1000) { } // a tight synchronous loop — nothing else can run during this
}

setTimeout(() => console.log("this waits for the block to finish, no matter how it was scheduled"), 0);
blockForOneSecond(); // the browser can't paint, respond to clicks, or run ANY queued callback until this returns
```

This is the mechanism behind a frozen or unresponsive page during heavy synchronous computation — it isn't that async code is somehow broken, it's that the single call stack is fully occupied and the event loop has no opportunity to process anything else until it's free again.

### Recursive microtask scheduling can starve macrotasks and rendering

```javascript
function scheduleForever() {
  Promise.resolve().then(scheduleForever); // each microtask schedules ANOTHER microtask, endlessly
}
scheduleForever();
// this can prevent the browser from ever getting to paint or process a queued setTimeout,
// since the microtask queue is drained COMPLETELY before the next macrotask is even considered
```

## Application

Use this model to predict logging order in interview snippets and to explain real symptoms: a loading spinner that doesn't paint until heavy synchronous work finishes, or a resolved-promise callback that runs before a same-tick `setTimeout(fn, 0)`. Browsers get an opportunity to render after work completes, but the exact rendering moment is browser-controlled; `requestAnimationFrame` coordinates visual work with that rendering cycle in [Module 4](../m04-browser-platform-and-aspnet-core-api-integration/README.md). For genuinely CPU-heavy work that would otherwise block the main thread, a Web Worker (covered later in this track) runs it off the main thread entirely, rather than relying on promises or timers to somehow make it non-blocking.

## Common Mistakes

- Believing `setTimeout(fn, 0)` runs immediately, rather than as a macrotask queued behind the current synchronous code and any pending microtasks.
- Assuming promises provide actual CPU parallelism, rather than just a scheduling mechanism still running on the same single thread.
- Writing a long synchronous loop and being surprised the UI freezes, not realizing async code offers no protection against blocking synchronous work.
- Forgetting that microtasks are drained *completely* before the next macrotask, which can cause recursive microtask scheduling to starve rendering and timers indefinitely.

## Common Interview Questions

### Basic
- What is the event loop, at a high level?
- What's the difference between the microtask queue and the macrotask queue?

### Intermediate
- Why does a `Promise.resolve().then(...)` callback run before a `setTimeout(fn, 0)` callback, even though the timer was scheduled first?
- Why can a long synchronous loop freeze an otherwise-asynchronous page?

### Advanced
- Walk through, step by step, the exact output order of a snippet mixing synchronous logs, a `setTimeout`, and a resolved promise.
- How could excessive recursive microtask scheduling delay rendering or timer callbacks indefinitely?

### Follow-up Questions
- Does code after an `await` run synchronously, or is it deferred as a microtask?
- Are all browser events (clicks, scroll) macrotasks, or are some handled differently?

### Code Prediction
```javascript
console.log("start");
setTimeout(() => console.log("timeout"), 0);
Promise.resolve()
  .then(() => console.log("promise 1"))
  .then(() => console.log("promise 2"));
console.log("end");
```
Predict the full logging order, including why both promise callbacks log before the timeout callback despite each being a separate `.then()`.

## Practical Tasks

- Predict the output of a snippet mixing synchronous code, `setTimeout`, and promises, then verify it in an actual browser console.
- Reproduce a frozen-UI scenario with a synchronous blocking loop, and explain why wrapping it in a promise doesn't fix it.
- Explain, for a code review, why two chained `.then()` callbacks both log before a same-tick `setTimeout` callback.

## Readiness Criteria

Predict common event-loop ordering scenarios precisely, explain why synchronous work blocks rendering and callbacks regardless of async code elsewhere, and distinguish microtasks from macrotasks by name and by scheduling behavior.

## References

- [MDN: The event loop](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Execution_model#the_event_loop)
- [MDN: Microtask guide](https://developer.mozilla.org/docs/Web/API/HTML_DOM_API/Microtask_guide)
