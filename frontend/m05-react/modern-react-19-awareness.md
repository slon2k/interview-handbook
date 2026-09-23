# Modern React 19 Awareness

## Definition

Modern React includes APIs for keeping urgent interaction responsive while non-urgent rendering catches up, and for describing optimistic or action-based updates. These APIs extend the core rendering, state, and effect model; they do not remove the need to understand it.

## How It Works

`startTransition` and `useTransition` mark a state update as non-urgent. Use them when an immediate interaction such as typing should remain responsive while React prepares a heavier update such as filtering a large result view. They do not make a network request faster and they are not a replacement for debouncing, cancellation, or server-state caching.

React 19 introduces APIs that candidates may encounter in current codebases:

- `useOptimistic` renders a temporary expected result while a mutation is pending, then reconciles it with the confirmed outcome.
- Actions and `useActionState` coordinate form submission state in environments that support them.
- `use` can read a promise or context during rendering in supported patterns.
- Server Components render some component work on the server in a framework or server-rendering architecture; they are not part of a conventional client-only React SPA by default.

## Application

Use a transition when the same interaction has an urgent update and an expensive non-urgent update. For example, update the input state immediately and transition the expensive result rendering. Use optimistic UI only when the product can explain and recover from a rejected mutation. Continue to use ordinary state, effects, and [TanStack Query](tanstack-query-and-server-state.md) when they better match a client-side API feature.

## Common Mistakes

- Claiming that transitions cancel requests or create CPU parallelism.
- Showing optimistic data without a rollback or reconciliation plan.
- Treating `use` or Server Components as required in every React application.
- Replacing a clear event handler with an effect merely because a newer API exists.

## Common Interview Questions

### Foundation

- What problem does `useTransition` solve?
- What is optimistic UI?

### Intermediate

- Why is a transition different from a loading state?
- When should an optimistic update roll back?

### Advanced and Follow-up

- How do Server Components differ from a conventional client-rendered SPA, and why is that mostly a framework architecture decision?

## Practical Tasks

- Identify an expensive filter update and explain which part would be urgent versus transition work.
- Design an optimistic status toggle with pending, confirmed, and rollback states.

## Readiness Criteria

You can recognize modern React APIs in a codebase, explain their trade-offs, and choose a classic state/effect or server-state pattern when it is clearer.

## References

- [React: `useTransition`](https://react.dev/reference/react/useTransition)
- [React: `useOptimistic`](https://react.dev/reference/react/useOptimistic)
- [React: Server Components](https://react.dev/reference/rsc/server-components)
