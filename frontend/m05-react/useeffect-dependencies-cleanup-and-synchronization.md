# `useEffect`, Dependencies, Cleanup, and Synchronization

## Definition

`useEffect` synchronizes a component with an external system — a subscription, a timer, a network request, a browser API — after React has committed the render to the screen. It is not a general-purpose "run this after rendering" callback; anything triggered directly by a user action (a click, a submit) belongs in an event handler, not an effect.

```tsx
useEffect(() => {
  const id = setInterval(() => setTick(t => t + 1), 1000);
  return () => clearInterval(id); // cleanup: runs before the next effect, and when the component unmounts
}, []); // empty dependency array: this effect's setup/cleanup pair runs once
```

## Alternatives & Trade-offs

An event handler runs exactly once, in direct response to a specific user action, with no dependency array to reason about. An effect runs after every render where its dependencies changed, specifically to keep the component synchronized with something outside React's own state — the trade-off is that effects introduce a whole category of correctness concerns (missing dependencies, cleanup, re-run timing) that a plain event handler never has to deal with, which is exactly why an effect should only be reached for when something external genuinely needs to be kept in sync.

## How It Works

### The dependency array must list every reactive value the effect actually reads

```tsx
function SearchResults({ query }: { query: string }) {
  const [results, setResults] = useState<Result[]>([]);

  useEffect(() => {
    fetchResults(query).then(setResults); // reads `query`
  }, []); // WRONG: query is missing — this effect captures the FIRST render's query forever

  // RIGHT:
  useEffect(() => {
    fetchResults(query).then(setResults);
  }, [query]); // now reruns whenever query actually changes
}
```

An effect's dependency array isn't a performance tuning knob — it's a correctness contract describing every reactive value the effect body reads. Omitting `query` doesn't make the effect run "more efficiently"; it makes the effect permanently use whichever `query` value existed on the render that first created it, a classic stale-closure bug.

### Cleanup — runs before the next effect, and on unmount

```tsx
useEffect(() => {
  function handleResize() { console.log(window.innerWidth); }
  window.addEventListener("resize", handleResize);
  return () => window.removeEventListener("resize", handleResize); // ALWAYS pair a subscribe with an unsubscribe
}, []);
```

Without the returned cleanup function, every time this effect re-ran (or the component unmounted and a new instance mounted elsewhere) would add *another* listener without ever removing the old one — a memory leak and duplicate-handling bug that compounds the more often the effect runs.

### Effects run for every render where a listed dependency changed — including double-invocation in development

```tsx
useEffect(() => {
  console.log("effect ran");
  return () => console.log("cleanup ran");
}, []);
// In development, React Strict Mode deliberately runs setup -> cleanup -> setup once extra,
// specifically to surface effects whose cleanup is missing or incorrect. This does NOT happen
// in production — an effect that "seems to run twice" in development but works in production
// is usually fine; one that leaves duplicated side effects behind is a real cleanup bug.
```

### The stale-closure trap in an event handler stored across renders

```tsx
function Timer() {
  const [count, setCount] = useState(0);

  useEffect(() => {
    const id = setInterval(() => {
      console.log(count); // this closure was created ONCE, when the effect first ran — count is frozen at that value
    }, 1000);
    return () => clearInterval(id);
  }, []); // empty deps: the interval callback never sees a fresh `count`
}
```

```tsx
// Fixed: use a functional update, so the callback never needs to read the stale `count` closure at all
useEffect(() => {
  const id = setInterval(() => {
    setCount(c => c + 1); // reads the LATEST queued state, not a frozen closure value
  }, 1000);
  return () => clearInterval(id);
}, []);
```

### Deriving values that don't need an effect at all

```tsx
// WRONG: uses an effect to compute something that could just be computed during render
const [fullName, setFullName] = useState("");
useEffect(() => { setFullName(`${firstName} ${lastName}`); }, [firstName, lastName]);

// RIGHT: no effect needed — just calculate it directly during render
const fullName = `${firstName} ${lastName}`;
```

## Application

Use `useEffect` specifically to synchronize with something outside React — a subscription, a timer, a fetch, direct DOM/browser API access. List every reactive value the effect body reads as a dependency, honestly. Always pair a subscription/timer/listener with matching cleanup. Compute derived values directly during render instead of storing and syncing them through an effect.

## Common Mistakes

- Omitting a dependency the effect actually uses, creating a stale closure that keeps referencing an old value forever.
- Forgetting to return a cleanup function for a subscription, timer, or listener, causing duplicates to accumulate every time the effect reruns.
- Using an effect to calculate a value that could simply be computed during render, adding an unnecessary extra render cycle.
- Putting logic that should run directly in response to a user action (a click, a form submit) inside an effect instead of an event handler.
- Panicking at React Strict Mode's deliberate double-invocation in development instead of treating it as a signal to check cleanup correctness.

## Common Interview Questions

### Basic
- What is `useEffect` for, and when does it run relative to rendering?
- When does an effect's cleanup function run?

### Intermediate
- Why does omitting a dependency an effect reads create a stale-closure bug rather than just "skipping some work"?
- How would you fix a `setInterval` callback inside an effect that always logs the same, outdated state value?

### Advanced
- Walk through why React Strict Mode intentionally runs an effect's setup, cleanup, and setup again in development, and what that's meant to surface.
- How do you decide whether a piece of logic belongs in an event handler or an effect?

### Follow-up Questions
- Does an effect with an empty dependency array ever run more than once in production?
- Can an effect's cleanup function reference variables from the same render that scheduled the effect?

### Code Prediction
```tsx
function Counter() {
  const [count, setCount] = useState(0);
  useEffect(() => {
    const id = setInterval(() => console.log(count), 1000);
    return () => clearInterval(id);
  }, []);
  return <button onClick={() => setCount(c => c + 1)}>{count}</button>;
}
```
After clicking the button several times, what value does the interval's `console.log` keep printing, and why doesn't it reflect the current `count`?

## Practical Tasks

- Reproduce the stale-closure interval bug and fix it using a functional state update.
- Refactor an effect that derives a value (like a full name from first/last name) into a plain render-time calculation.
- Add correct cleanup to an effect that subscribes to a browser event, verifying no duplicate listeners accumulate across re-renders.

## Readiness Criteria

Explain effect timing and honest dependency arrays precisely, diagnose and fix stale-closure bugs in effects, and distinguish synchronization (effects) from direct event response (handlers).

## References

- [React: Synchronizing with Effects](https://react.dev/learn/synchronizing-with-effects)
- [React: You Might Not Need an Effect](https://react.dev/learn/you-might-not-need-an-effect)
- [React: `useEffect`](https://react.dev/reference/react/useEffect)
