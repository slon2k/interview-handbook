# Promises and `async`/`await`

## Definition

A promise represents the eventual result or failure of asynchronous work. `async` functions always return promises; `await` pauses only that async function until a promise settles, then resumes it with the fulfilled value or throws its rejection.

## How It Works

- A promise is pending, fulfilled, or rejected; settlement happens once.
- Calling an async function starts executing it immediately until its first `await`; it does not wait until the caller awaits it.
- `await` does not block the JavaScript thread. It schedules continuation after the awaited promise settles.
- Use `Promise.all` for independent work that should fail together; use `Promise.allSettled` when every outcome is needed.
- A `try`/`catch` around `await` handles rejection. A promise created but neither awaited nor returned can produce an unhandled rejection.
- Sequential `await`s are correct when later work needs earlier output; otherwise start operations first, then await their combined result.

## Application

Make execution shape explicit in data-loading and submission code. Preserve errors, cancellation, and loading state for the UI layer rather than converting every failure to an empty value.

## Common Mistakes

- Assuming `await` creates a new thread.
- Accidentally serialising independent requests with consecutive awaits.
- Forgetting to await or return a promise from a wrapper function.
- Using `Promise.all` when partial results must remain useful after one failure.

## Common Interview Questions

### Foundation

- What does `async` guarantee about a function's return value?
- What happens when an awaited promise rejects?

### Intermediate

- How would you run two independent requests concurrently?
- When should you use `Promise.allSettled` instead of `Promise.all`?

### Advanced and Follow-up

- Why can an async error escape a surrounding synchronous `try`/`catch`?

### Code Prediction

Predict whether two requests begin sequentially or concurrently in `await first(); await second();` versus creating both promises before `await Promise.all`.

## Practical Tasks

- Refactor independent async operations so they run concurrently and handle their combined failure deliberately.
- Repair a submission handler that reports success before its request has settled.

## Readiness Criteria

You can choose sequential or concurrent promise composition, propagate failures deliberately, and explain what `await` does and does not block.

## References

- [MDN: Promise](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Promise)
- [MDN: async function](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Statements/async_function)