# Reconciliation, Rerenders, and Keys

## Definition

A **rerender** means React calls a component function again to compute what its UI should look like next; it does not automatically mean the real DOM changes. **Reconciliation** is the process of comparing the new element tree against the previous one to decide which DOM nodes can be reused and which must be created or removed — element type and `key` are what determine identity between the two trees.

```tsx
function Parent() {
  const [count, setCount] = useState(0);
  return <Child label="static" />; // Parent RERENDERS on every click, but Child's PROPS never change
}
```

## Alternatives & Trade-offs

Letting every component rerender freely is simplest to reason about and is fine for the vast majority of UI, since a rerender that produces the same output costs relatively little. Memoization (`React.memo`, `useMemo`, `useCallback`) can prevent unnecessary work for measurably expensive renders, but adds real maintenance cost — dependency arrays to keep honest, and referential-equality subtleties that are easy to get wrong — which is exactly why it should be added in response to a measured problem, not by default.

## How It Works

### A rerender is not automatically a DOM update

```tsx
function Parent() {
  const [count, setCount] = useState(0);
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <ExpensiveStaticContent /> {/* Parent rerenders every click; React still diffs THIS subtree's
                                     output against last time, and if it's identical, no DOM change happens here */}
    </div>
  );
}
```

`Parent` calling its function again on every click doesn't mean every DOM node inside it is torn down and rebuilt — React compares the newly computed element tree against the previous one and only touches the parts that actually differ.

### `React.memo` skips a child's render entirely when its props are shallowly equal

```tsx
const ExpensiveChild = React.memo(function ExpensiveChild({ label }: { label: string }) {
  console.log("ExpensiveChild rendered");
  return <div>{label}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <ExpensiveChild label="static" /> {/* label NEVER changes -> React.memo skips re-running this component */}
    </>
  );
}
```

Without `React.memo`, `ExpensiveChild` would re-run its own function every time `Parent` rerenders, even though its actual output would be identical — `React.memo` compares the new props against the previous ones and skips the child's render entirely when they're shallowly equal.

### The trap: an inline object or function prop is a NEW reference every render

```tsx
const MemoChild = React.memo(function MemoChild({ options }: { options: { size: string } }) {
  console.log("MemoChild rendered");
  return <div>{options.size}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <MemoChild options={{ size: "large" }} /> {/* a BRAND NEW object literal, created fresh every render */}
    </>
  );
}
// MemoChild re-renders on EVERY click, despite React.memo, because {{ size: "large" }} is never
// === to the previous render's object, even though its CONTENTS are identical every time
```

`React.memo`'s shallow comparison checks reference equality, not deep content equality — an inline object or arrow function literal is a fresh reference on every single render, defeating the memoization even though the logical content never actually changes.

### The fix — stabilize the reference, or move the value out of the render entirely

```tsx
const stableOptions = { size: "large" }; // defined OUTSIDE the component: same reference forever

function Parent() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>{count}</button>
      <MemoChild options={stableOptions} /> {/* now genuinely the SAME reference every render -> memo works */}
    </>
  );
}
```

### Keys and identity among siblings

```tsx
{items.map((item, index) => <ListItem key={index} {...item} />)}
// WRONG for reorderable/insertable lists: index-based keys tie identity to POSITION, not the actual item —
// inserting a new item at the top shifts every existing item's key, causing React to think each item CHANGED
// identity rather than just moved, potentially losing per-item local state (like an open/closed toggle)

{items.map(item => <ListItem key={item.id} {...item} />)}
// RIGHT: a stable, item-specific key preserves identity correctly regardless of list reordering or insertion
```

## Application

Don't add memoization reflexively — profile first to confirm a specific render is actually expensive before reaching for `React.memo`/`useMemo`/`useCallback`. When memoizing, make sure prop values passed to a memoized child are genuinely stable references, not fresh object/array/function literals created inline every render. Always key list items by a stable, item-specific identifier, never by array index, for any list that can reorder or have items inserted/removed.

## Common Mistakes

- Treating every component rerender as a performance problem or a guaranteed DOM update, without checking whether reconciliation actually produced any real DOM change.
- Wrapping a component in `React.memo` while still passing it an inline object, array, or arrow function prop, defeating the memoization with a fresh reference every render.
- Using the array index as a list key for a list that can reorder, insert, or delete items, causing item identity to follow position instead of the actual data.
- Adding `useMemo`/`useCallback` without first profiling to confirm the calculation or child render is actually expensive enough to matter.

## Common Interview Questions

### Basic
- What causes a React component to rerender?
- What role do keys play in reconciliation?

### Intermediate
- Why can a `React.memo`-wrapped child still rerender on every parent update despite unchanged-looking props?
- Why are array indexes risky as keys for a reorderable list?

### Advanced
- Walk through, step by step, why an inline object literal prop defeats `React.memo`'s shallow comparison even when its contents never change.
- How would you investigate whether a suspected unnecessary rerender is actually causing a measurable performance problem before optimizing it?

### Follow-up Questions
- Does `React.memo` prevent a component from rerendering when its own internal state changes?
- Is comparing props by reference the same as comparing them by content?

### Code Prediction
```tsx
const Child = React.memo(function Child({ config }: { config: { theme: string } }) {
  console.log("rendered");
  return <div>{config.theme}</div>;
});
function Parent() {
  const [n, setN] = useState(0);
  return <><button onClick={() => setN(x => x + 1)}>{n}</button><Child config={{ theme: "dark" }} /></>;
}
```
Does `"rendered"` log every time the button is clicked, despite `Child` being wrapped in `React.memo`? Why?

## Practical Tasks

- Reproduce the inline-object-prop `React.memo` failure, then fix it by hoisting the object to a stable reference.
- Diagnose a list bug caused by index-based keys where reordering an item causes another item's local state (like an expanded/collapsed toggle) to move to the wrong row.
- Use the React DevTools Profiler to identify whether a suspected expensive render is real before adding memoization.

## Readiness Criteria

Distinguish rerendering from actual DOM changes, explain reconciliation and key-based identity precisely, and correctly diagnose when `React.memo` is defeated by an unstable reference rather than reaching for more memoization blindly.

## References

- [React: Render and Commit](https://react.dev/learn/render-and-commit)
- [React: Preserving and Resetting State](https://react.dev/learn/preserving-and-resetting-state)
- [React: `memo`](https://react.dev/reference/react/memo)
