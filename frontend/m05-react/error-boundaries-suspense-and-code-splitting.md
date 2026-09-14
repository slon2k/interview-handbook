# Error Boundaries, Suspense, and Code Splitting

## Definition

Error boundaries isolate rendering failures so one broken subtree does not necessarily take down the entire application. Suspense coordinates fallback UI while a supported operation is not ready, and code splitting loads less JavaScript up front.

## How It Works

- An error boundary catches rendering errors in its descendant tree and renders a fallback or recovery action.
- Traditional error boundaries are implemented with a class component or a library abstraction; ordinary `try/catch` around JSX does not catch render errors.
- `React.lazy` loads a component dynamically and is commonly paired with `Suspense` for a loading fallback.
- Route-based code splitting delays feature code until the route is needed.
- Fallbacks should preserve useful layout and communicate what the user can recover from.

## Application

Place boundaries around meaningful feature or route units and provide recovery or navigation options. Use code splitting for routes or features whose initial cost is worth delaying, while measuring the resulting loading experience.

## Common Mistakes

- Assuming one top-level boundary gives every feature an appropriate recovery experience.
- Catching render errors with a normal `try/catch` in the parent function.
- Showing a blank screen or generic spinner for every Suspense boundary.
- Splitting tiny components and increasing request overhead without meaningful bundle benefit.
- Treating code splitting as a substitute for reducing unnecessary dependencies.

## Common Interview Questions

### Foundation

- What problem does an error boundary solve?
- What is `Suspense` used for?

### Intermediate

- Why does a normal `try/catch` not catch every rendering error?
- Why is route-based code splitting useful?

### Advanced and Follow-up

- How would you design a recovery action after a feature subtree fails?
- What trade-offs should be measured before adding a new split point?

### Code Prediction

Given a lazy route without a surrounding `Suspense` boundary, predict what the user sees while the module is loading and why a fallback is required.

## Practical Tasks

- Add a feature-level error boundary with a retry or navigation fallback.
- Split a route-level feature and explain how the initial bundle and loading experience change.

## Readiness Criteria

You can distinguish rendering errors from event-handler errors, explain Suspense and lazy loading, and choose useful component or route boundaries.

## References

- [React: Component APIs](https://react.dev/reference/react/Component)
- [React: `lazy`](https://react.dev/reference/react/lazy)
- [React: `Suspense`](https://react.dev/reference/react/Suspense)
