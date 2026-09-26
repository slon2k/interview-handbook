# Typed Data Fetching and Async UI States

## Definition

A React data-fetching feature connects a typed API boundary (Module 4) to explicit UI states — idle, loading, success, empty, error, retrying — rather than treating "the data" as the only meaningful outcome. Modeling these states as a discriminated union makes impossible combinations (loading *and* showing stale error data at once) unrepresentable, instead of relying on several independent booleans that can drift out of sync.

```tsx
type RequestState<T> =
  | { status: "idle" }
  | { status: "loading" }
  | { status: "success"; data: T }
  | { status: "empty" }
  | { status: "error"; error: Error };
```

## Alternatives & Trade-offs

Modeling async state as `{ data?: T; loading: boolean; error?: string }` is quicker to write initially, but permits contradictory combinations no real request can produce — `loading: true` alongside a populated `error`, for instance — which a component then has to defensively guard against anyway. A discriminated union costs a bit more upfront type definition, but makes every state explicit and mutually exclusive, so the compiler (not a runtime bug) catches an attempt to read `data` from a branch where it doesn't exist.

## How It Works

### The request lifecycle at the React boundary — cancellation plus a "still current" guard

```tsx
useEffect(() => {
  const controller = new AbortController();
  let isCurrent = true;

  setState({ status: "loading" });
  loadProducts(filters, controller.signal)
    .then(result => {
      if (isCurrent) {
        setState(result.length === 0 ? { status: "empty" } : { status: "success", data: result });
      }
    })
    .catch(error => {
      if (isCurrent && error.name !== "AbortError") {
        setState({ status: "error", error });
      }
    });

  return () => {
    isCurrent = false;
    controller.abort();
  };
}, [filters]);
```

Cancellation (`controller.abort()`) prevents unnecessary work from continuing; the `isCurrent` guard is a second, complementary safety net — even if an abstraction somewhere between here and the network doesn't fully respect cancellation, a response that arrives after this effect's cleanup ran is simply ignored rather than applied to a now-outdated view.

### Rendering each state explicitly, instead of inferring it from booleans

```tsx
function ProductList({ state }: { state: RequestState<Product[]> }) {
  switch (state.status) {
    case "idle": return null;
    case "loading": return <Spinner />;
    case "empty": return <EmptyState message="No products found" />;
    case "error": return <ErrorMessage error={state.error} />;
    case "success": return <ul>{state.data.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
  }
}
```

Every branch is exhaustive and mutually exclusive — there's no way to accidentally render a spinner *and* an error message at once, because the type itself doesn't allow a state that's simultaneously both.

### Distinguishing initial load from background refresh

```tsx
type RequestState<T> =
  | { status: "loading" }                              // NO data yet at all — show a full spinner
  | { status: "refreshing"; data: T }                    // has EXISTING data, quietly fetching an update
  | { status: "success"; data: T }
  | { status: "error"; error: Error; staleData?: T };      // failed, but might still have OLD data worth showing
```

A background refresh shouldn't hide data the user already has and can usefully see — collapsing "no data yet" and "has data, just refreshing" into one `loading` state loses that distinction and produces a jarring, unnecessary full-page spinner on every refresh.

### Authorization failures aren't retryable network errors

```tsx
.catch(error => {
  if (error.status === 401 || error.status === 403) {
    setState({ status: "error", error, retryable: false }); // don't offer a "retry" button for this
  } else {
    setState({ status: "error", error, retryable: true });
  }
});
```

## Application

Model async UI state as a discriminated union with mutually exclusive branches, rendering each explicitly rather than inferring the current state from a combination of independent booleans. Coordinate route/filter changes with request cancellation and a "still current" check. Distinguish true initial loading from background refresh in the UI, and don't offer a retry for genuinely non-retryable failures like authorization errors.

## Common Mistakes

- Modeling async state with independent `data?`, `loading`, and `error?` fields, permitting contradictory combinations the UI then has to defensively check for.
- Treating an empty, successful result the same as an error or a still-loading state.
- Letting a stale response from a previous filter/route overwrite the current view because no cancellation or "still current" check was in place.
- Showing a full-page loading spinner for a background refresh, hiding data the user already had and could still usefully see.
- Offering a retry button for a `401`/`403` failure as though it were a transient network problem.

## Common Interview Questions

### Basic
- What states should a data-fetching component typically represent, beyond just "loading" and "data"?
- Why is an empty successful result different from an error?

### Intermediate
- How does a discriminated union prevent a component from rendering contradictory states at once?
- How would you prevent a stale request's response from overwriting a newer one?

### Advanced
- How would you distinguish initial loading, background refreshing, and retrying in the same component's UI?
- When does raw `useEffect`-based fetching like this become limiting enough to justify a server-state library (previous topics)?

### Follow-up Questions
- Does an aborted request (via `AbortController`) count as an error state in a well-designed model?
- Should a `401`/`403` failure offer the same retry affordance as a `503`?

### Code Prediction
A search box fires a request for `query = "a"`, then almost immediately for `query = "ab"`. If the first request resolves *after* the second one, what must the component do to avoid showing results for `"a"` as the final displayed state?

## Practical Tasks

- Build a typed list component with idle, loading, empty, error, and success branches modeled as a discriminated union.
- Add cancellation and a "still current" guard to a search feature, then verify the race scenario with artificially delayed responses.
- Distinguish initial loading from background refresh in a component's rendered output.

## Readiness Criteria

Model async UI state as an exhaustive discriminated union, coordinate cancellation correctly to avoid stale-response bugs, and distinguish initial loading from refresh and retryable from non-retryable failures.

## References

- [React: Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
- [React: Conditional Rendering](https://react.dev/learn/conditional-rendering)
- [TypeScript Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
