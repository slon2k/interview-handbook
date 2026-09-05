# Closures

## Definition

A closure is a function together with access to the lexical environment in which it was created. It retains bindings, not frozen snapshots of their values, for as long as the function remains reachable.

## How It Works

- A nested function can read and update bindings declared in an outer function after that outer call has returned.
- Each call to an outer factory function creates a distinct environment.
- Closures are the basis of callbacks, event handlers, module-private state, and many React hook behaviours.
- A closure sees the binding's current value. Capturing a mutable binding can therefore cause a later callback to observe newer state than the author expected.
- Long-lived closures can retain large objects unnecessarily when they remain referenced by listeners, timers, or caches.

## Application

Use closures to keep related state private or parameterize callbacks. When a callback should use a particular value, make its lifecycle and dependencies explicit; this is the underlying issue behind many stale-closure bugs in React.

## Common Mistakes

- Assuming closures copy a value at function creation.
- Capturing a loop variable with `var`.
- Keeping a listener alive after the component or feature that owns it has gone away.

## Common Interview Questions

### Foundation

- What is a closure?
- Why can an inner function still use an outer variable after the outer function returns?

### Intermediate

- How can a closure create private state?
- What causes the loop-variable closure bug?

### Advanced and Follow-up

- How can a stale closure appear in an asynchronous UI callback?

### Code Prediction

Predict the independent counts returned by two calls to a `createCounter` function that returns an incrementing closure.

## Practical Tasks

- Implement a small factory with private state and a limited public API.
- Diagnose a delayed callback that reads an outdated or unintended value.

## Readiness Criteria

You can explain lexical capture, identify closure lifetime issues, and use closures without confusing bindings with snapshots.

## References

- [MDN: Closures](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Closures)