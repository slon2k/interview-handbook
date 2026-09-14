# Typed Data Fetching and Async UI States

## Definition

A React data-fetching feature connects a typed API boundary to UI states such as idle, loading, success, empty, error, and retrying. The component should represent these states explicitly instead of treating data as the only meaningful outcome.

## How It Works

- A typed API client returns validated data or a structured failure model from Module 4's boundary.
- The component renders a state-specific branch rather than accessing data before it exists.
- Requests should be cancelled or ignored when inputs change so old results cannot overwrite current state.
- Loading indicators should distinguish initial loading from background refresh where the user already has data.
- Retry behavior should preserve enough context to repeat the request without duplicating side effects or hiding authorization failures.

## Application

Model async UI states as a discriminated union or an equivalent explicit state machine. Keep transport concerns in an API client or custom hook and keep the component responsible for rendering the state honestly.

## Common Mistakes

- Using `data?: T`, `loading`, and `error?: string` in combinations that permit contradictory states.
- Treating an empty successful result as a failure or loading state.
- Updating the current view from a stale request after route or filter changes.
- Retrying `401` or `403` responses as though they were transient network failures.
- Putting all request logic directly in a large component.

## Common Interview Questions

### Foundation

- Which states should a data-driven component represent?
- Why is an empty result different from an error?

### Intermediate

- How do you prevent a stale request from overwriting newer data?
- Where should response validation and error mapping happen?

### Advanced and Follow-up

- How would you distinguish initial loading, refreshing, and retrying in the UI?
- When does raw effect-based fetching become limiting enough to justify a server-state library?

### Code Prediction

Given a list request that starts for `query = "a"` and then for `query = "ab"`, explain why the first response must not replace the second result even if it resolves later.

## Practical Tasks

- Build a typed list component with loading, empty, error, success, and retry branches.
- Add cancellation or request identity checks to a search feature and test the race scenario with real delayed responses.

## Readiness Criteria

You can connect typed API results to honest React UI states, handle races and cancellation, and distinguish client rendering decisions from API-boundary validation.

## References

- [React: Synchronizing with effects](https://react.dev/learn/synchronizing-with-effects)
- [React: Conditional rendering](https://react.dev/learn/conditional-rendering)
- [TypeScript Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
