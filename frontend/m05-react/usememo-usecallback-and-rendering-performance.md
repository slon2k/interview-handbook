# `useMemo`, `useCallback`, and Rendering Performance

## Definition

`useMemo` caches the *result* of a calculation between renders, recomputing only when its dependencies change. `useCallback` caches a *function reference* the same way. Both exist purely to preserve referential equality across renders — neither makes a calculation itself faster, and neither is required for correctness.

```tsx
const sortedItems = useMemo(() => expensiveSort(items), [items]); // recomputes only when `items` changes
const handleClick = useCallback(() => doSomething(id), [id]);       // same FUNCTION reference across renders, as long as `id` doesn't change
```

## Alternatives & Trade-offs

Recomputing a value or recreating a function on every render is simpler and is genuinely fine for the vast majority of components — the cost of most calculations and closures is negligible compared to the cost of reasoning about a dependency array. `useMemo`/`useCallback` trade that simplicity for avoiding real, measured work, at the cost of a dependency list that must stay honest — an incomplete dependency array silently freezes a stale cached value or function, exactly like the `useEffect` dependency problem.

## How It Works

### `useMemo` avoids repeating an expensive calculation

```tsx
function ProductList({ products, sortOrder }: { products: Product[]; sortOrder: string }) {
  const sortedProducts = useMemo(
    () => [...products].sort((a, b) => compareBy(a, b, sortOrder)), // expensive for a large list
    [products, sortOrder] // only recompute when EITHER of these actually changes
  );
  return <ul>{sortedProducts.map(p => <li key={p.id}>{p.name}</li>)}</ul>;
}
```

Without `useMemo`, `sortedProducts` would be recalculated on *every* render of `ProductList`, even one triggered by something entirely unrelated (a parent rerendering for its own reasons) — `useMemo` skips that recalculation unless `products` or `sortOrder` genuinely changed.

### `useCallback` matters specifically when paired with `React.memo`

```tsx
const MemoButton = React.memo(function MemoButton({ onClick }: { onClick: () => void }) {
  console.log("MemoButton rendered");
  return <button onClick={onClick}>Save</button>;
});

function Parent() {
  const [count, setCount] = useState(0);

  const handleSave = useCallback(() => { console.log("saving"); }, []); // SAME function reference every render

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <MemoButton onClick={handleSave} /> {/* stable reference -> React.memo actually skips this render */}
    </>
  );
}
```

Without `useCallback`, `handleSave` would be a brand-new function on every render of `Parent`, which — exactly like the inline-object-prop trap in the reconciliation topic — would defeat `MemoButton`'s `React.memo` even though the function's *behavior* never actually changes.

### A memoized value can go stale just like an effect can

```tsx
function SearchResults({ items, query }: { items: Item[]; query: string }) {
  const filtered = useMemo(
    () => items.filter(i => i.name.includes(query)),
    [items] // WRONG: query is used inside the calculation but missing from the dependency array
  );
  // filtered keeps using whatever `query` was true the FIRST time this ran — a stale value bug,
  // identical in shape to the missing-dependency bug covered in the useEffect topic
}
```

### Memoization has a real, ongoing cost — it isn't free

```tsx
// For a cheap calculation, useMemo's own bookkeeping (storing the dependency array, comparing it
// every render) can cost MORE than just recalculating the value directly would have:
const doubled = useMemo(() => count * 2, [count]); // almost certainly not worth it — `count * 2` is trivially cheap
```

## Application

Reach for `useMemo`/`useCallback` specifically to preserve referential equality for a `React.memo`-wrapped child, or to avoid repeating a measurably expensive calculation — not as a default habit applied to every value and function. Keep dependency arrays honest, exactly as with `useEffect`. Profile before adding either, and remove them if profiling shows no measurable benefit.

## Common Mistakes

- Wrapping every calculation in `useMemo` and every function in `useCallback` regardless of actual cost, adding bookkeeping overhead that can exceed the cost of simply recalculating.
- Using `useCallback` with no `React.memo`-wrapped consumer (or other referential-equality-sensitive dependency) actually relying on the stable reference, gaining nothing.
- Omitting a dependency the memoized calculation or callback actually uses, producing a stale value bug identical in shape to a missing `useEffect` dependency.
- Assuming `useMemo`/`useCallback` alone fix a performance problem that's actually caused by unnecessary state scope, an unstable key, or a genuinely expensive child tree.

## Common Interview Questions

### Basic
- What does `useMemo` do, and what does `useCallback` do?
- Are `useMemo` and `useCallback` required for a component to behave correctly?

### Intermediate
- Why does `useCallback` only matter when paired with something referential-equality-sensitive, like `React.memo`?
- How can a memoized value become stale, and how does that resemble a `useEffect` dependency bug?

### Advanced
- Walk through why wrapping a trivial calculation in `useMemo` can make a component slower rather than faster.
- How would you decide, using the profiler, whether a specific `useMemo`/`useCallback` addition is actually justified?

### Follow-up Questions
- Does `useMemo` guarantee the cached value is never recalculated, or just that React will try to reuse it when possible?
- Can a `useCallback`'d function still close over a stale value if its dependency array is incomplete?

### Code Prediction
```tsx
function Search({ items, query }: { items: Item[]; query: string }) {
  const filtered = useMemo(() => items.filter(i => i.name.includes(query)), [items]);
  return <ul>{filtered.map(i => <li key={i.id}>{i.name}</li>)}</ul>;
}
```
If `query` changes but `items` doesn't, does `filtered` reflect the new `query`? Why or why not, and what's the one-line fix?

## Practical Tasks

- Reproduce the stale-memoized-value bug from a missing dependency, then fix it.
- Use `useCallback` to stabilize a callback prop passed to a `React.memo`-wrapped child, and verify (via a console log) that the child stops rerendering unnecessarily.
- Use the React DevTools Profiler to measure whether a specific `useMemo` addition provides any real benefit for a given calculation.

## Readiness Criteria

Explain what `useMemo`/`useCallback` actually preserve (referential equality, not raw speed), correctly diagnose stale-memoization bugs, and justify their use with profiling evidence rather than applying them by default.

## References

- [React: `useMemo`](https://react.dev/reference/react/useMemo)
- [React: `useCallback`](https://react.dev/reference/react/useCallback)
- [React: React Developer Tools](https://react.dev/learn/react-developer-tools)
