# `useRef`, `useReducer`, `useContext`, and Custom Hooks

## Definition

React provides hooks for values that should persist without rendering, explicit state transitions, shared context, and reusable stateful behavior. `useRef`, `useReducer`, `useContext`, and custom hooks solve different problems and should not be treated as interchangeable state tools.

## How It Works

- `useRef` stores a mutable value across renders without causing a rerender when that value changes.
- A ref can point to a DOM node or hold an instance-like value such as a timer handle.
- `useReducer` models state transitions as actions handled by a pure reducer, which is useful when transitions are related or complex.
- `useContext` reads a value provided by an ancestor, but consumers still rerender when the provided context value changes.
- A custom hook composes hooks and returns a focused API; it shares logic, not state, between component instances.

## Application

Use refs for imperative escape hatches and values that do not affect rendering. Use reducers for explicit transition logic, context for stable cross-cutting dependencies, and custom hooks to package behavior with a clear contract.

### Typed reducers and focused contexts

Use discriminated action unions so a reducer handles only valid payloads and TypeScript checks every transition. This applies [Module 3's discriminated-union model](../m03-typescript/narrowing-type-guards-and-discriminated-unions.md) to React transitions.

```tsx
type DialogAction =
  | { type: "open"; itemId: string }
  | { type: "close" };

type DialogState = { open: boolean; itemId: string | null };
```

If a context combines frequently changing data with stable commands, split it. Consumers that need only commands then avoid rerendering for every data update. Hide an optional raw context behind a custom hook that gives callers a clear provider error:

```tsx
function useSession(): Session {
  const session = useContext(SessionContext);
  if (session === undefined) {
    throw new Error("useSession must be used inside SessionProvider");
  }
  return session;
}
```

A custom hook should return a compact typed API, such as `{ state, reload, cancel }`, rather than leaking implementation details or pretending multiple callers share state automatically.

## Common Mistakes

- Using a ref for data that should appear in rendered output.
- Putting a large, frequently changing object in one context and causing broad rerenders.
- Treating a custom hook as a global store; each caller normally receives its own state.
- Writing reducers with mutations or side effects.
- Using context to avoid designing a better component boundary.
- Putting high-frequency values such as pointer position into broad context and rerendering unrelated consumers.
- Exposing `ContextValue | undefined` to every caller instead of checking once in a provider-specific hook.

## Common Interview Questions

### Foundation

- What is the difference between state and a ref?
- What problem does `useReducer` solve?

### Intermediate

- How does context affect rerenders?
- Do two components calling the same custom hook share state?

### Advanced and Follow-up

- How would you split a context whose value contains both stable commands and frequently changing data?
- When is a reducer clearer than several related state setters?

### Code Prediction

Given a ref whose `current` value changes in an event handler, predict whether the component renders again and whether the next render can observe the new value.

## Practical Tasks

- Build a custom hook that owns request state and returns a small typed API.
- Refactor a complex form state machine from multiple booleans into a reducer with explicit actions.
- Split a context containing stable commands and frequently changing data, then explain which consumers stop rerendering.

## Readiness Criteria

You can distinguish refs, reducers, context, and custom hooks and choose among them based on rendering behavior, state ownership, and transition complexity.

## References

- [React: `useRef`](https://react.dev/reference/react/useRef)
- [React: `useReducer`](https://react.dev/reference/react/useReducer)
- [React: `useContext`](https://react.dev/reference/react/useContext)
- [React: Reusing logic with custom hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
