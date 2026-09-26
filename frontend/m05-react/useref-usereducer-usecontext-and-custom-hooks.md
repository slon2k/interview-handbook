# `useRef`, `useReducer`, `useContext`, and Custom Hooks

## Definition

React provides distinct hooks for distinct problems: `useRef` holds a mutable value across renders without triggering one when it changes; `useReducer` models state transitions as explicit actions handled by a pure function; `useContext` reads a value provided by an ancestor; a custom hook composes existing hooks into a focused, reusable API. They solve genuinely different problems and shouldn't be treated as interchangeable ways to "hold some state."

```tsx
const inputRef = useRef<HTMLInputElement>(null); // no rerender when .current changes
const [state, dispatch] = useReducer(reducer, initialState); // explicit action-driven transitions
```

## Alternatives & Trade-offs

Several related `useState` calls are simpler for a handful of independent values, but become hard to reason about once transitions are interdependent — `useReducer` centralizes those transitions in one place, at the cost of more upfront structure (an action type, a reducer function) than just calling a setter directly. Passing a value down through props is simplest for a shallow tree, but becomes unwieldy ("prop drilling") through many intermediate layers that don't themselves need the value — `useContext` solves that reach at the cost of making every consumer implicitly depend on a provider existing above it.

## How It Works

### `useRef` — a value that survives renders without causing one

```tsx
function Stopwatch() {
  const intervalRef = useRef<number | null>(null); // holds an ID; changing it does NOT trigger a rerender

  function start() {
    intervalRef.current = window.setInterval(() => console.log("tick"), 1000);
  }
  function stop() {
    if (intervalRef.current !== null) clearInterval(intervalRef.current);
  }
}
```

If `intervalRef.current` were instead `useState`, every tick's ID assignment would (unnecessarily) schedule a rerender — a ref is exactly the right tool because the *value itself* never needs to appear in rendered output.

### `useReducer` with typed, discriminated actions

```tsx
type DialogAction = { type: "open"; itemId: string } | { type: "close" };
type DialogState = { open: boolean; itemId: string | null };

function dialogReducer(state: DialogState, action: DialogAction): DialogState {
  switch (action.type) {
    case "open": return { open: true, itemId: action.itemId };
    case "close": return { open: false, itemId: null };
  }
}

const [state, dispatch] = useReducer(dialogReducer, { open: false, itemId: null });
dispatch({ type: "open", itemId: "42" }); // the only way any caller can trigger a valid transition
```

This applies [Module 3's discriminated-union model](../m03-typescript/narrowing-type-guards-and-discriminated-unions.md) directly to React state transitions — every valid action is enumerated, and the reducer's `switch` narrows the payload correctly for each case, with the compiler ensuring every case is actually handled.

### `useContext` — and why a broad, frequently-changing context re-renders everything

```tsx
const SessionContext = createContext<Session | undefined>(undefined);

function useSession(): Session {
  const session = useContext(SessionContext);
  if (session === undefined) {
    throw new Error("useSession must be used inside SessionProvider"); // fails loudly at the point of misuse
  }
  return session;
}
```

Hiding the raw, possibly-`undefined` context behind `useSession()` means every consumer gets a guaranteed, non-null `Session` and a clear error if the provider is missing — instead of every single call site needing its own `if (session === undefined)` check.

```tsx
// A context combining rarely-changing commands with frequently-changing data forces EVERY
// consumer to re-render on every data change, even ones that only needed the stable commands:
const AppContext = createContext<{ theme: string; mousePosition: { x: number; y: number } } | undefined>(undefined);
// Splitting these into two separate contexts lets a "theme-only" consumer skip re-rendering
// every time mousePosition updates, which a combined context cannot avoid.
```

### Custom hooks — packaging behavior, not sharing state between callers

```tsx
function useFetch<T>(url: string) {
  const [state, setState] = useState<{ status: "loading" | "success" | "error"; data?: T }>({ status: "loading" });

  useEffect(() => {
    let active = true;
    fetch(url).then(res => res.json()).then(data => { if (active) setState({ status: "success", data }); });
    return () => { active = false; };
  }, [url]);

  return state;
}

// TWO components calling useFetch("/api/orders") each get their OWN independent state —
// they do NOT automatically share a single cached result the way a server-state library would
```

## Application

Use refs for values that don't affect rendered output — DOM node access, timer/interval handles, any "instance variable"-like value. Use reducers when transitions are related or complex enough that centralizing them in one function clarifies the logic. Use context for genuinely cross-cutting, infrequently-changing dependencies, splitting frequently-changing data into its own context to avoid over-broad re-renders. Package reusable stateful logic into a custom hook with a small, clearly-typed return contract.

## Common Mistakes

- Using a ref for a value that should appear in rendered output, then being confused why the UI doesn't update when it changes.
- Combining frequently-changing data and rarely-changing commands in one context, causing every consumer to re-render on every data update.
- Assuming two components calling the same custom hook share state, when each call creates its own independent instance unless the hook explicitly wraps a shared external source.
- Writing a reducer with mutations or side effects instead of a pure state transition function.
- Exposing a context value typed as possibly `undefined` to every consumer instead of centralizing the null-check in one dedicated hook.

## Common Interview Questions

### Basic
- What's the difference between state and a ref?
- What problem does `useReducer` solve that several `useState` calls don't?

### Intermediate
- How does putting frequently-changing data in a broad context affect unrelated consumers?
- Do two components calling the same custom hook share the same underlying state?

### Advanced
- How would you split a context whose value mixes stable commands with frequently-changing data, and what does that change about which consumers re-render?
- When is a reducer genuinely clearer than several related `useState` calls, versus just adding complexity?

### Follow-up Questions
- Does changing a ref's `.current` value ever trigger a rerender on its own?
- Can a custom hook itself hold state that's genuinely shared across every component that calls it?

### Code Prediction
```tsx
function Timer() {
  const countRef = useRef(0);
  function handleClick() { countRef.current += 1; console.log(countRef.current); }
  return <button onClick={handleClick}>Increment</button>;
}
```
Does the displayed button text ever change after clicks? Does `console.log` still show the incrementing value? Explain the distinction.

## Practical Tasks

- Build a custom hook that owns request state and returns a small, typed API (`{ data, isLoading, error }`).
- Refactor a form's multiple related boolean/string state variables into a single `useReducer` with explicit actions.
- Split a context mixing frequently-changing and rarely-changing values into two separate contexts, and verify which consumers stop re-rendering unnecessarily.

## Readiness Criteria

Distinguish refs, reducers, context, and custom hooks precisely by the problem each solves, and choose among them based on rendering behavior and state-transition complexity rather than habit.

## References

- [React: `useRef`](https://react.dev/reference/react/useRef)
- [React: `useReducer`](https://react.dev/reference/react/useReducer)
- [React: `useContext`](https://react.dev/reference/react/useContext)
- [React: Reusing Logic with Custom Hooks](https://react.dev/learn/reusing-logic-with-custom-hooks)
