# Component Identity, Composition, and State

## Definition

React preserves a component's state based on its **type and position** in the rendered tree, not based on the data it currently displays. Composition (passing UI through `children` or other element props) lets a parent share layout without needing to know a child's internal details; lifting state moves shared data to the nearest common ancestor that needs to coordinate it.

```tsx
{isEditing ? <Editor /> : <Viewer />} // DIFFERENT component types at the same position — no state is shared between them
```

## Alternatives & Trade-offs

Keeping state local to the component that needs it is simplest and avoids unnecessary coupling, but requires "lifting" it to a shared ancestor the moment two sibling components need to coordinate on the same value. Lifting state too eagerly (to the app root, by default) makes every value globally accessible at the cost of unnecessary re-renders and unclear ownership; composition (passing `children` or a callback prop) often solves what looks like a "need global state" problem without any lifting at all.

## How It Works

### Same position, same type — state survives; different type — state resets

```tsx
function App({ isEditing }: { isEditing: boolean }) {
  return isEditing ? <Editor /> : <Viewer />;
  // Editor and Viewer are DIFFERENT component types. Switching between them at this
  // position always unmounts one and mounts the other — neither's internal state survives the switch.
}
```

```tsx
function App({ recordId }: { recordId: string }) {
  return <Editor recordId={recordId} />;
  // SAME type, SAME position, across renders — Editor's internal state (e.g., a draft
  // text field) is PRESERVED even as recordId changes, unless something tells React otherwise
}
```

If `recordId` changes from `"1"` to `"2"` in the second example, `Editor`'s own local state (like an unsaved draft) stays intact by default — which is often exactly the bug: the form still shows record 1's half-typed edits, now attached to record 2.

### `key` forces React to treat an update as a fresh identity

```tsx
<Editor key={recordId} recordId={recordId} />
// Changing the key prop tells React: "this is now a DIFFERENT logical instance," even
// though the component type and position are identical — React unmounts the old instance
// (discarding its state) and mounts a brand new one for the new key.
```

This is the deliberate fix for the stale-draft bug above: giving `Editor` a `key` tied to `recordId` means switching records always starts the editor's internal state fresh, rather than reusing a previous record's leftover draft state.

### Composition — passing UI through, instead of the parent knowing every detail

```tsx
function Card({ children }: { children: ReactNode }) {
  return <div className="card">{children}</div>; // Card has NO idea what's actually inside it
}

<Card>
  <UserProfile user={currentUser} /> {/* Card never needed to know about UserProfile at all */}
</Card>
```

`Card` stays completely decoupled from what it wraps — a new kind of content can be placed inside it without ever modifying `Card` itself, which is the composition alternative to a parent needing an ever-growing list of specific props (`showUserProfile`, `showSettingsPanel`, ...) for every possible thing it might contain.

### Lifting state to the nearest shared owner — not necessarily the app root

```tsx
function FilterableList() {
  const [filter, setFilter] = useState(""); // lifted here: the LOWEST common ancestor that both need
  return (
    <>
      <SearchBox filter={filter} onFilterChange={setFilter} />
      <ResultsList filter={filter} />
    </>
  );
}
```

`filter` is lifted only as far as `FilterableList`, the nearest component both `SearchBox` and `ResultsList` share — not all the way to a global store, which would make it accessible (and re-render-triggering) far more broadly than actually needed.

### Derived values — computed during render, not stored and synchronized separately

```tsx
// WRONG: storing a value that's ENTIRELY derivable from other state, risking the two drifting out of sync
const [items, setItems] = useState<Item[]>([]);
const [itemCount, setItemCount] = useState(0); // has to be manually kept in sync with `items` forever

// RIGHT: just compute it during render — there's no separate state to keep synchronized at all
const [items, setItems] = useState<Item[]>([]);
const itemCount = items.length;
```

## Application

Rely on component type and position for state preservation by default, and use `key` deliberately whenever switching between logically distinct records should reset a component's internal state. Use composition to keep a wrapper decoupled from its content instead of growing its prop list. Lift state only as far as the nearest shared owner actually requires. Compute derived values during render rather than storing and manually synchronizing them.

## Common Mistakes

- Expecting a component's state to reset when its *props* change, without realizing state only resets when type, position, or `key` changes.
- Reusing the same component instance (same type, same position, no differentiating `key`) across logically distinct records, leaking one record's local state into the next.
- Growing a wrapper component's prop list indefinitely (`showX`, `showY`, `showZ`) instead of using `children` for composition.
- Lifting state all the way to the application root by default instead of to the nearest actual shared owner.
- Storing a value that's fully derivable from existing state or props, then needing to keep two sources of truth manually synchronized.

## Common Interview Questions

### Basic
- What determines whether React preserves or resets a component's state across renders?
- What does "lifting state" mean?

### Intermediate
- Why does switching between two conditionally-rendered components at the same position always reset their state?
- When is composition (`children`) preferable to a large, ever-growing prop list?

### Advanced
- Walk through why adding a `key` prop tied to a record ID fixes a stale-draft bug in an editable form component.
- How would you decide the correct "nearest shared owner" for a piece of state used by two sibling components?

### Follow-up Questions
- Does changing a component's props alone ever reset its internal state?
- Can two sibling components at the same tree position ever share the exact same state instance without lifting it?

### Code Prediction
```tsx
function App({ recordId }: { recordId: string }) {
  return <Editor recordId={recordId} />;
}
```
If `Editor` maintains its own local draft state and `recordId` changes from `"1"` to `"2"`, does the draft reset? What single change to this JSX would make it reset correctly?

## Practical Tasks

- Reproduce the stale-draft bug from reusing an editor component across different records, then fix it using `key`.
- Refactor a component accepting many boolean "show X" props into one using `children`-based composition instead.
- Identify a piece of state stored separately when it could be derived directly during render, and remove the redundant state.

## Readiness Criteria

Explain React's state-preservation rules by type/position/key precisely, use `key` deliberately to force a fresh identity, and choose composition and appropriate state-lifting over unnecessary prop growth or over-lifted state.

## References

- [React: Preserving and Resetting State](https://react.dev/learn/preserving-and-resetting-state)
- [React: Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component)
- [React: Sharing State Between Components](https://react.dev/learn/sharing-state-between-components)
