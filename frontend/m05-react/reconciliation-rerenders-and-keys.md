# Reconciliation, Rerenders, and Keys

## Definition

React rerenders components to calculate the next UI description and reconciles that result with the previous one. Reconciliation uses element type, position, and keys to decide what can be reused and what must be replaced.

## How It Works

- A state update or parent render can cause a component function to run again.
- A rerender does not automatically mean every DOM node is recreated; React compares the new element tree with the previous one.
- Element type and key determine identity among siblings.
- Referential equality matters when memoized components or dependency arrays compare objects and functions.
- `React.memo` can skip a child render when its props compare equal, but it does not prevent the child from rendering when its own state or context changes.

## Application

First make state ownership and rendering behavior correct. Use stable keys and clear prop boundaries before considering memoization. Profile a real bottleneck before adding `React.memo`, `useMemo`, or `useCallback`.

## Common Mistakes

- Treating every rerender as a DOM update or a performance bug.
- Using indexes or random values as keys.
- Creating new object props and callbacks while expecting shallow memoization to treat them as unchanged.
- Adding memoization without measuring or understanding the dependency relationship.
- Assuming `React.memo` prevents a component from responding to context or local state changes.

## Common Interview Questions

### Foundation

- What causes a React component to rerender?
- What role do keys play in reconciliation?

### Intermediate

- Why can a parent rerender even when a child has not changed logically?
- How does referential equality affect `React.memo`?

### Advanced and Follow-up

- When can memoization make a component slower or harder to maintain?
- How would you investigate whether a rerender is actually causing a performance problem?

### Code Prediction

Given a memoized child that receives an inline object prop, predict whether it renders after the parent updates and explain the identity comparison involved.

## Practical Tasks

- Diagnose a list bug caused by unstable keys and explain the observed state movement.
- Profile a component tree, identify an unnecessary expensive render, and justify the smallest optimization.

## Readiness Criteria

You can distinguish rerendering from DOM replacement, explain reconciliation and keys, reason about referential equality, and choose performance tools based on evidence.

## References

- [React: Render and commit](https://react.dev/learn/render-and-commit)
- [React: Preserving and resetting state](https://react.dev/learn/preserving-and-resetting-state)
- [React: `memo`](https://react.dev/reference/react/memo)
