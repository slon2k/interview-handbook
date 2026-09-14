# Hooks, `useState`, and Functional Updates

## Definition

Hooks are functions that let function components use React features such as state and lifecycle synchronization. `useState` stores component state, and functional updates express changes that depend on the previous state.

## How It Works

- Hooks must be called in the same order on every render, which is why they cannot be called conditionally or inside loops.
- `useState` returns the current value and a setter that schedules a later render.
- State setters do not immediately change the value captured by the current render.
- Functional updates such as `setCount(value => value + 1)` use the latest queued state and are important for multiple updates in one event.
- State should be replaced with a new value rather than mutated in place, especially for objects and arrays.

## Application

Use state for values that affect rendered output or need to persist between renders. Use functional updates whenever the next value depends on the previous value or when updates may be queued together.

## Common Mistakes

- Calling hooks conditionally and changing their order between renders.
- Expecting a state setter to update the current render's local variable.
- Mutating an object or array and passing the same reference back to the setter.
- Using `setValue(value + 1)` repeatedly when each update depends on the previous value.
- Storing derived data that can be calculated from existing state during render.

## Common Interview Questions

### Foundation

- Why are hooks required to be called in a consistent order?
- Why does a state setter not immediately change the current render?

### Intermediate

- When should you use a functional state update?
- How should an object in state be updated immutably?

### Advanced and Follow-up

- Why can a stale closure cause an event handler to use an old state value?
- When would `useReducer` communicate state transitions better than several `useState` calls?

### Code Prediction

Given three calls to `setCount(count + 1)` followed by three functional updates, predict the final values and explain which render each expression captures.

## Practical Tasks

- Refactor state updates that depend on previous state to use functional updates.
- Replace an in-place array mutation with an immutable update that preserves React's change detection.

## Readiness Criteria

You can explain hook ordering, state snapshots, functional updates, immutable state changes, and when state should be derived rather than stored.

## References

- [React: Rules of Hooks](https://react.dev/reference/rules/rules-of-hooks)
- [React: `useState`](https://react.dev/reference/react/useState)
- [React: Queueing a series of state updates](https://react.dev/learn/queueing-a-series-of-state-updates)
