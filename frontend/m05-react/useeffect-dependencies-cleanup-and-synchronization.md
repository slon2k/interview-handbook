# `useEffect`, Dependencies, Cleanup, and Synchronization

## Definition

`useEffect` synchronizes a component with an external system such as a browser API, subscription, timer, or network request. It is not a general-purpose callback for every action that happens after rendering.

## How It Works

- React renders the component first, then runs an effect after the commit when its dependencies require it.
- The dependency array describes the reactive values used by the effect; it is not a performance wish list.
- Cleanup runs before the effect re-runs and when the component is removed, allowing subscriptions, timers, and requests to be released.
- Values captured by an effect come from the render that created it, so missing dependencies can create stale behavior.
- User events such as submitting a form or clicking a button usually belong in event handlers rather than effects.

## Application

Use effects for synchronization with systems outside React. Keep the effect small, make dependencies honest, and use cleanup for resources that must stop or become irrelevant.

## Common Mistakes

- Using an effect to calculate derived values that could be computed during render.
- Omitting dependencies to silence repeated work and creating stale closures.
- Creating an effect loop by setting state that is also an effect dependency without a terminating condition.
- Forgetting to clean up listeners, timers, subscriptions, or in-flight requests.
- Treating Strict Mode development behavior as proof that production logic is broken instead of making effects idempotent.

## Common Interview Questions

### Foundation

- What is an effect for?
- When does cleanup run?

### Intermediate

- Why does an effect need all reactive dependencies?
- How would you cancel or ignore an outdated request in an effect?

### Advanced and Follow-up

- How do you decide whether logic belongs in an event handler or an effect?
- Why can an effect that derives state create an unnecessary render cycle?

### Code Prediction

Given an effect that reads `query` but has an empty dependency array, predict which query value it uses after the user changes the search field.

## Practical Tasks

- Refactor an effect that derives filtered data into a render-time calculation.
- Add cleanup to an effect that subscribes to a browser event or starts a request.

## Readiness Criteria

You can explain effect timing, honest dependencies, cleanup, stale closures, idempotence, and the distinction between synchronization and event handling.

## References

- [React: Synchronizing with effects](https://react.dev/learn/synchronizing-with-effects)
- [React: You might not need an effect](https://react.dev/learn/you-might-not-need-an-effect)
- [React: `useEffect`](https://react.dev/reference/react/useEffect)
