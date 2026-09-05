# The Event Loop, Microtasks, and Macrotasks

## Definition

JavaScript executes synchronous code on a call stack. The event loop coordinates later work by running queued tasks when the stack is empty. Promise reactions use the microtask queue; timers and many browser events use the task queue, commonly called the macrotask queue.

## How It Works

- Synchronous code runs to completion before queued work begins.
- After the current task completes, the runtime drains microtasks before taking the next task.
- `.then`, `.catch`, `.finally`, and continuation after `await` schedule microtasks when their promise settles.
- `setTimeout` schedules a task after its minimum delay; it is not a guarantee of exact execution time.
- Long synchronous work blocks input, painting, timer callbacks, and promise continuations until it returns control to the event loop.
- Excessive recursive microtasks can delay rendering and task-queue work.

## Application

Use this model to explain ordering bugs, delayed UI feedback, and why an async function can resume before a zero-delay timer. For CPU-heavy browser work, later Module 4's Web Worker awareness is more appropriate than assuming promises create parallelism.

## Common Mistakes

- Saying that `setTimeout(fn, 0)` runs immediately.
- Treating promises as threaded or parallel CPU work.
- Forgetting that synchronous loops still block the browser.
- Calling every later callback a “macrotask” without checking the API.

## Common Interview Questions

### Foundation

- What is the event loop?
- What is the difference between a microtask and a timer task?

### Intermediate

- Why does a resolved promise callback run before a zero-delay timer?
- Why can a long synchronous loop freeze an otherwise async UI?

### Advanced and Follow-up

- How could repeated microtask scheduling affect rendering responsiveness?

### Code Prediction

Predict the logging order for synchronous logs, `Promise.resolve().then(...)`, and `setTimeout(..., 0)`.

## Practical Tasks

- Explain an event-loop logging snippet without running it, then confirm it with browser developer tools.
- Diagnose why a loading indicator cannot paint before CPU-heavy synchronous work begins.

## Readiness Criteria

You can predict common task ordering and explain why async I/O does not make synchronous CPU work non-blocking.

## References

- [MDN: Execution model](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Execution_model)
- [MDN: Using promises](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Using_promises)