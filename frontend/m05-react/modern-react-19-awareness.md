# Modern React 19 Awareness

## Definition

React 19 adds APIs for keeping urgent interaction responsive while a related, non-urgent update catches up (`useTransition`), and for showing an expected result before a mutation is actually confirmed (`useOptimistic`). This is working-awareness content — knowing what these APIs are for and their limits, not necessarily reaching for them by default over the ordinary state/effect model this module already covers.

```tsx
const [isPending, startTransition] = useTransition();
const [filterQuery, setFilterQuery] = useState("");

function handleChange(newQuery: string) {
  setQuery(newQuery); // URGENT: keep the input responsive immediately
  startTransition(() => setFilterQuery(newQuery)); // NON-URGENT: schedule the derived view update
}

const filteredResults = expensiveFilter(allItems, filterQuery);
```

## Alternatives & Trade-offs

Without a transition, a single state update that triggers an expensive render (filtering a large list on every keystroke) can make typing itself feel laggy, since React treats every update with equal urgency by default. Marking the state update that drives the expensive render as a transition lets React keep the input update urgent and make the derived render interruptible. The filter function still runs synchronously during rendering; a transition does not make CPU work faster or move it to another thread.

## How It Works

### `useTransition` — splitting one interaction into an urgent part and a non-urgent part

```tsx
function SearchBox() {
  const [query, setQuery] = useState("");
  const [filterQuery, setFilterQuery] = useState("");
  const [isPending, startTransition] = useTransition();

  function handleChange(e: ChangeEvent<HTMLInputElement>) {
    const value = e.target.value;
    setQuery(value); // updates immediately — the input never feels laggy

    startTransition(() => setFilterQuery(value)); // React can interrupt the render driven by this update
  }

  const results = expensiveFilter(allItems, filterQuery);

  return (
    <>
      <input value={query} onChange={handleChange} />
      {isPending ? <Spinner /> : <ResultsList items={results} />}
    </>
  );
}
```

Without the transition, the state that drives the expensive render would be updated with the same urgency as `setQuery`. With the transition, React can commit the urgent input update first and interrupt/restart the render for the new results when needed. The `expensiveFilter` call itself still runs synchronously during that render.

### `useTransition` does not cancel requests or make anything computationally faster

```tsx
startTransition(() => {
  fetchExpensiveReport(); // this is NOT sped up, parallelized, or cancelled by being inside a transition —
                            // transitions only affect how REACT SCHEDULES rendering work, nothing about
                            // network calls or raw computation speed
});
```

A common misconception worth correcting directly: `useTransition` is a rendering-priority tool, not a networking or concurrency tool — it has nothing to do with debouncing a search input or cancelling an in-flight `fetch` (Module 4's territory), both of which still need their own explicit handling.

### `useOptimistic` — showing an expected result before the server confirms it

```tsx
function TodoItem({ todo, onTodoUpdated }: {
  todo: Todo;
  onTodoUpdated: (todo: Todo) => void;
}) {
  const [optimisticTodo, setOptimisticTodo] = useOptimistic(
    todo,
    (state, newDone: boolean) => ({ ...state, done: newDone })
  );

  function handleToggle() {
    const nextDone = !optimisticTodo.done;

    startTransition(async () => {
      setOptimisticTodo(nextDone); // shows the NEW state immediately, before the request resolves
      const savedTodo = await toggleTodoRequest(todo.id, nextDone);
      startTransition(() => onTodoUpdated(savedTodo)); // commit the confirmed server state
    });
  }

  return <Checkbox checked={optimisticTodo.done} onChange={handleToggle} />;
}
```

`useOptimistic` renders the expected outcome instantly for a snappier feel, then reconciles with the confirmed value supplied through `onTodoUpdated`. If the request fails and the canonical `todo` remains unchanged, the optimistic value automatically reverts. The setter must run inside an Action, and the canonical state or server-state cache must be updated after success. This only makes sense for actions where a rare failure and its visible "snap back" are an acceptable user experience — not for anything where showing, then un-showing, a result would be confusing or costly.

## Application

Use `useTransition` specifically when one user interaction has both an urgent part (keep the input responsive) and a non-urgent, potentially expensive part (recompute a large filtered/derived view) — not as a general performance switch. Use `useOptimistic` only for actions where an optimistic result the product can clearly explain and gracefully roll back is worth the snappier feel; keep using ordinary state and [TanStack Query](tanstack-query-and-server-state.md) for everything else.

## Common Mistakes

- Assuming `useTransition` cancels in-flight requests or speeds up computation, rather than understanding it purely as a rendering-priority scheduling tool.
- Adding `useOptimistic` to a mutation with no sensible, explainable rollback behavior for the (rare but real) case where the request fails.
- Reaching for these newer APIs by default instead of recognizing that ordinary `useState`/`useEffect`/TanStack Query already solve most UI needs perfectly well.
- Treating Server Components or `use()` as required knowledge for every React application, when they're specific to frameworks/architectures that opt into server rendering, not a plain client-rendered SPA by default.

## Common Interview Questions

### Basic
- What problem does `useTransition` solve?
- What is optimistic UI, and what does `useOptimistic` provide?

### Intermediate
- Why doesn't wrapping a `fetch` call in `startTransition` make the request itself faster or cancel it?
- When would an optimistic update need to roll back, and what triggers that?

### Advanced
- Walk through how `useTransition` changes the *order* React schedules two state updates from the same event, without changing anything about the underlying computation cost.
- How do Server Components differ from a conventional client-rendered SPA, and why is adopting them a framework/architecture decision rather than a React language feature?

### Follow-up Questions
- Does `isPending` from `useTransition` reflect network loading time, or purely React's own rendering scheduling?
- Is `useOptimistic` appropriate for every mutation, or only specific ones?

### Code Prediction
```tsx
function handleChange(value: string) {
  setQuery(value);
  startTransition(() => setFilterQuery(value));
}
```
While typing quickly, when does `expensiveFilter` run, and what specifically is `startTransition` doing to keep its render interruptible?

## Practical Tasks

- Build a search input where typing stays responsive while an expensive filter/sort operation is deferred using `useTransition`.
- Implement an optimistic toggle (like a like/favorite button) using `useOptimistic`, including its rollback behavior on a simulated failure.
- Explain, for a specific mutation in a hypothetical app, whether optimistic UI would be appropriate or risky given its failure mode.

## Readiness Criteria

Explain what `useTransition` and `useOptimistic` actually do (and don't do) precisely, and judge when they're worth reaching for versus when ordinary state and server-state tooling are already sufficient.

## References

- [React: `useTransition`](https://react.dev/reference/react/useTransition)
- [React: `useOptimistic`](https://react.dev/reference/react/useOptimistic)
- [React: Server Components](https://react.dev/reference/rsc/server-components)
