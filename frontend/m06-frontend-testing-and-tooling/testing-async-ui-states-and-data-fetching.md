# Testing Asynchronous UI States and Data Fetching

## Definition

An asynchronous UI test verifies the states a person sees while a request is pending, succeeds with data, succeeds with no data, fails, or retries. It waits for a meaningful state transition rather than sleeping for an arbitrary duration.

## Alternatives & Trade-offs

Arbitrary timers can make a test appear stable locally while racing under CI load. RTL's `findBy` queries and `waitFor` retry assertions until an observable expectation holds, but they should not be used to conceal an assertion that is too broad or a request that never resolves.

## How It Works

Control the API outcome at the request boundary, assert the initial state, then await the state a user should see:

```tsx
render(<ProductList />);

expect(screen.getByRole("status", { name: "Loading products" })).toBeInTheDocument();
expect(await screen.findByRole("heading", { name: "Keyboard" })).toBeInTheDocument();
expect(screen.queryByRole("status", { name: "Loading products" })).not.toBeInTheDocument();
```

Use MSW for request scenarios in the next topic group. The component's loading and error model is established in [Module 5](../m05-react/typed-data-fetching-and-async-ui-states.md); this topic verifies that model from outside the component.

## Application

Test initial loading separately from background refresh when the product exposes both. Include empty responses, validation errors, retry controls, and authorization outcomes where users need a distinct response. Keep cancellation and stale-response behavior at an integration boundary where delayed requests can be controlled credibly.

## Common Mistakes

- Calling `waitFor` around assertions that could be synchronous.
- Waiting for a fixed timeout instead of a visible state.
- Testing only a success response and leaving error or empty branches unprotected.
- Treating an aborted request as an error state.

## Common Interview Questions

### Foundation

- What is the difference between `getBy`, `findBy`, and `queryBy`?
- Which states should a data-loading component expose?

### Intermediate

- Why are fixed delays flaky?
- How would you prove an empty result is distinct from a failure?

### Advanced and Follow-up

- How would you test that an outdated response cannot replace a newer search result?

### Code Prediction

Why can `expect(screen.getByText("Result"))` fail immediately after render even when the request will succeed?

## Practical Tasks

- Write tests for loading, success, empty, and error states of a list feature.
- Add a delayed request scenario and verify the loading status disappears only after the visible result arrives.

## Readiness Criteria

You can test asynchronous UI transitions without arbitrary waits and can identify which request outcomes need distinct visible behavior.

## References

- [Testing Library async methods](https://testing-library.com/docs/dom-testing-library/api-async/)
- [React: Conditional rendering](https://react.dev/learn/conditional-rendering)
