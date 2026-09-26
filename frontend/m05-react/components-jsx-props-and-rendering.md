# Components, JSX, Props, and Rendering

## Definition

A React component is a function that receives inputs, usually called props, and returns a description of UI. JSX is syntax that lets component code describe element structure while remaining ordinary JavaScript and TypeScript — it compiles down to plain function calls, not a separate template language with its own runtime.

```tsx
function Greeting({ name }: { name: string }) {
  return <p>Hello, {name}</p>; // compiles to something like createElement("p", null, "Hello, ", name)
}
```

## Alternatives & Trade-offs

Writing UI as explicit `createElement()` calls (what JSX actually compiles to) works identically but reads far less like the HTML structure it produces, making a deeply nested UI hard to visually parse. JSX trades a small amount of build-tooling dependency (a transpile step) for markup that closely resembles the rendered output, which is why virtually no real React codebase writes `createElement` calls by hand despite them being equivalent under the hood.

## How It Works

### Typed props and events

```tsx
import type { MouseEvent, ReactNode } from "react";

type SaveButtonProps = {
  children: ReactNode;
  disabled?: boolean;
  onSave: (event: MouseEvent<HTMLButtonElement>) => void;
};

function SaveButton({ children, disabled = false, onSave }: SaveButtonProps) {
  return <button type="button" disabled={disabled} onClick={onSave}>{children}</button>;
}
```

Type required and optional props deliberately, and type a handler by the DOM element that actually emits it. See [TypeScript components, props, events, and generics](typescript-components-props-events-and-generics.md) for render callbacks, generic components, reducers, context, and ref interoperability.

### JSX expressions are evaluated during rendering — including a surprising `0`

```tsx
function Cart({ count }: { count: number }) {
  return <div>{count && <Badge count={count} />}</div>;
  // if count is 0: {count && ...} evaluates to 0 (not false), and React RENDERS the literal text "0"
}
```

```tsx
// Fixed: force an actual boolean, so the falsy branch renders nothing instead of the number 0
function Cart({ count }: { count: number }) {
  return <div>{count > 0 && <Badge count={count} />}</div>;
}
```

`false`, `null`, and `undefined` all render as nothing — but `0` and `""` are valid, renderable values in JSX, so `{count && <X />}` prints a stray `0` on screen the moment `count` is legitimately zero, which is exactly the truthiness trap from Module 2 resurfacing inside JSX specifically.

### Stable keys for lists

```tsx
{items.map(item => <ListItem key={item.id} {...item} />)} // stable, item-specific key — survives reordering correctly
```

See [Reconciliation, rerenders, and keys](reconciliation-rerenders-and-keys.md) for exactly what goes wrong with index-based keys and how identity is determined during reconciliation.

## Application

Keep components focused on describing one coherent piece of UI and its inputs. Use typed props to make required data and callback contracts visible, and keep display-only transformations close to the component boundary. JSX should still emit semantic HTML — a component does not make a `div` an appropriate replacement for a native button, label, or landmark (see [Module 1: Semantic HTML](../m01-web-platform-foundations/semantic-html-and-document-structure.md)).

## Common Mistakes

- Calling a handler directly while building JSX (`onClick={onSave()}`) instead of passing the function reference (`onClick={onSave}`).
- Using array indexes as keys for a reorderable or insertable list.
- Mutating props or state while constructing the JSX for the current render.
- Assuming a falsy value like `0` disappears from a conditional expression the way `false`/`null` do.
- Using a generic `div` with a click handler for a semantic control, losing native keyboard behavior for no benefit.

## Common Interview Questions

### Basic
- What is a React component, at its core?
- What does JSX actually compile to?

### Intermediate
- Why must list keys be stable and unique among siblings?
- Why does `{count && <Badge />}` render a literal `0` when `count` is zero?

### Advanced
- How would you type a component that accepts either `children` or a render callback, but not both?
- When should a large component be split into smaller ones, and what signals suggest that split point?

### Follow-up Questions
- Is JSX required to use React, or is it purely a convenience over `createElement`?
- Does a component need to return a single root element, or can it return multiple siblings?

### Code Prediction
```tsx
function Cart({ count }: { count: number }) {
  return <div>{count && <span>{count} items</span>}</div>;
}
```
What does this render when `count` is `0`? What's the minimal fix?

## Practical Tasks

- Convert a loosely typed component into one with explicit, typed props and callback types.
- Fix a conditional-rendering bug where a legitimate `0` displays as stray text.
- Replace a clickable `div` with a typed native `<button>` while preserving its visual styling.

## Readiness Criteria

Describe the relationship between JSX, props, state, and rendering precisely, and spot the common falsy-value and unstable-key mistakes in a component before they cause a visible bug.

## References

- [React: Your First Component](https://react.dev/learn/your-first-component)
- [React: Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component)
- [React: Rendering Lists](https://react.dev/learn/rendering-lists)
