# Typed Data Fetching and Async UI States

## Definition

A React data-fetching feature connects a typed API boundary to UI states such as idle, loading, success, empty, error, and retrying. The component should represent these states explicitly instead of treating data as the only meaningful outcome.

## How It Works

- A typed API client returns validated data or a structured failure model from Module 4's boundary.
- The component renders a state-specific branch rather than accessing data before it exists.
- Requests should be cancelled or ignored when inputs change so old results cannot overwrite current state.
- Loading indicators should distinguish initial loading from background refresh where the user already has data.
- Retry behavior should preserve enough context to repeat the request without duplicating side effects or hiding authorization failures.

### Request lifecycle at the React boundary

When effect inputs change, create an `AbortController` for that request and abort it in cleanup. Cancellation prevents unnecessary work, but the client should still ensure it only applies results that belong to the current request. The browser mechanics are covered in [Module 4: Fetch and cancellation](../m04-browser-platform-and-aspnet-core-api-integration/fetch-requests-cancellation-and-stale-responses.md).

```tsx
useEffect(() => {
  const controller = new AbortController();
  let current = true;

  void loadProducts(filters, controller.signal)
    .then(result => {
      if (current) {
        setState({ status: "success", data: result });
      }
    })
    .catch(error => {
      if (current && error.name !== "AbortError") {
        setState({ status: "error", error });
      }
    });

  return () => {
    current = false;
    controller.abort();
  };
}, [filters]);
```

The active-result guard complements cancellation: cleanup makes a request irrelevant before an API or abstraction finishes settling. Treat cancellation as neither success nor an error UI. Keep initial loading distinct from background refresh, retain existing data only when the product decision calls for it, and map status-specific API failures through the contract and auth rules from [Module 4](../m04-browser-platform-and-aspnet-core-api-integration/api-dtos-pagination-and-validation-errors.md).

## Application

Model async UI states as a discriminated union or an equivalent explicit state machine. Keep transport concerns in an API client or custom hook and keep the component responsible for rendering the state honestly.

## Common Mistakes

- Using `data?: T`, `loading`, and `error?: string` in combinations that permit contradictory states.
- Treating an empty successful result as a failure or loading state.
- Updating the current view from a stale request after route or filter changes.
- Retrying `401` or `403` responses as though they were transient network failures.
- Putting all request logic directly in a large component.
- Showing an initial-load spinner for a background refresh and hiding usable existing data unnecessarily.
- Rendering an aborted request as a network failure.

## Common Interview Questions

### Foundation

- Which states should a data-driven component represent?
- Why is an empty result different from an error?

### Intermediate

- How do you prevent a stale request from overwriting newer data?
- Where should response validation and error mapping happen?
- How should initial loading differ from background refresh and retry in the UI?

### Advanced and Follow-up

- How would you distinguish initial loading, refreshing, and retrying in the UI?
- When does raw effect-based fetching become limiting enough to justify a server-state library?

### Code Prediction

Given a list request that starts for `query = "a"` and then for `query = "ab"`, explain why the first response must not replace the second result even if it resolves later.

## Practical Tasks

- Build a typed list component with loading, empty, error, success, and retry branches.
- Add cancellation or request identity checks to a search feature and test the race scenario with real delayed responses.
- Change a filter rapidly and verify that an aborted request never replaces the latest list or renders an error state.

## Readiness Criteria

You can connect typed API results to honest React UI states, handle races and cancellation, and distinguish client rendering decisions from API-boundary validation.

## References

- [React: Synchronizing with effects](https://react.dev/learn/synchronizing-with-effects)
- [React: Conditional rendering](https://react.dev/learn/conditional-rendering)
- [TypeScript Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
