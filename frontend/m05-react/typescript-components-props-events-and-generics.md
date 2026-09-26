# TypeScript Components, Props, Events, and Generics

## Definition

TypeScript makes a React component's inputs and outputs explicit: props describe what a caller must provide, event handlers describe which DOM event a callback receives, and generics let a reusable component preserve the exact type of the data it renders. A plain function with an explicit props type is sufficient for almost every component — `React.FC` is optional, not required.

```tsx
type ButtonProps = { label: string; onClick: () => void };
function Button({ label, onClick }: ButtonProps) { return <button onClick={onClick}>{label}</button>; }
```

## Alternatives & Trade-offs

`React.FC<Props>` implicitly adds a `children` prop and a specific return type, which was historically the "default" way examples were written — but it makes every component accept `children` whether or not that's actually intended, and adds indirection with no real benefit over an ordinary typed function. Writing a plain function with an explicit, precise props type is more work to type out per-component but says exactly what the component accepts, with no implicit extras.

## How It Works

### Props, children, and DOM events

```tsx
import type { ChangeEvent, ReactNode } from "react";

type SearchBoxProps = {
  label: ReactNode;
  value: string;
  onChange: (event: ChangeEvent<HTMLInputElement>) => void;
  children?: ReactNode;
};

export function SearchBox({ label, value, onChange, children }: SearchBoxProps) {
  return (
    <label>
      {label}
      <input value={value} onChange={onChange} />
      {children}
    </label>
  );
}
```

Use `ReactNode` when a prop accepts renderable content — text, elements, fragments, arrays of those. Type handlers by the specific element that emits them (`ChangeEvent<HTMLInputElement>`, not a generic click-event type used for everything), rather than leaving them implicitly `any`.

### Mutually exclusive props and render callbacks

```tsx
type TextLabelProps = { label: string; renderLabel?: never };
type RenderLabelProps = { label?: never; renderLabel: () => ReactNode };
type LabelProps = TextLabelProps | RenderLabelProps;

function Label({ label, renderLabel }: LabelProps) {
  return <span>{renderLabel ? renderLabel() : label}</span>;
}
```

This mutually-exclusive union (Module 3's discriminated-union model, applied to component props) prevents a caller from supplying *both* a text label and a render callback at once — a combination that would be ambiguous and that a simple `{ label?: string; renderLabel?: () => ReactNode }` shape would have silently allowed.

### Generic components — preserving the caller's actual item type

```tsx
type ListProps<Item extends { id: string }> = {
  items: readonly Item[];
  renderItem: (item: Item) => ReactNode;
};

function List<Item extends { id: string }>({ items, renderItem }: ListProps<Item>) {
  return <ul>{items.map(item => <li key={item.id}>{renderItem(item)}</li>)}</ul>;
}
```

The constraint (`extends { id: string }`) only requires what the component actually needs — an `id` for the key — while still letting `renderItem` receive the caller's *exact* item type (a `Product`, a `User`, whatever was actually passed in), not a widened, generic shape.

### Wrapping a native element without repeating its props

```tsx
import type { ComponentPropsWithoutRef } from "react";

type PrimaryButtonProps = ComponentPropsWithoutRef<"button"> & {
  tone?: "primary" | "danger";
};
```

`ComponentPropsWithoutRef<"button">` gives every standard HTML button attribute (`disabled`, `onClick`, `type`, and so on) for free, so a wrapper component doesn't need to manually redeclare each one it wants to pass through.

## Application

Keep the TypeScript contract at the component boundary explicit and precise. Use the framework-independent state models from [Module 3](../m03-typescript/typing-ui-state-forms-and-async-results.md) for props and hook results, and reach for React-specific types only for JSX, DOM events, children, and ref interoperability.

## Common Mistakes

- Defaulting to `React.FC` out of habit, implicitly granting every component an unintended `children` prop.
- Giving an event handler an implicit `any`, or using the wrong DOM event type (a click-event type for a change handler).
- Writing a generic component with no constraint at all, then assuming every item has an `id` available for its key.
- Publishing a context value typed as possibly `undefined` to every consumer instead of checking it once in a dedicated hook.

## Common Interview Questions

### Basic
- Why is a plain function with typed props usually sufficient for a React component?
- When would a prop's type be `ReactNode` instead of `string` or a specific element type?

### Intermediate
- How would you type an input change handler versus a form submit handler?
- How would you prevent a component from accepting both a text label and a render callback at the same time?

### Advanced
- How would you type a reusable list component while preserving the caller's specific item type through to the render callback?
- Why should a context value that might be `undefined` typically be accessed through a dedicated hook rather than directly?

### Follow-up Questions
- Does `React.FC` provide any type safety that a plain typed function doesn't?
- Can a generic component's constraint be narrower than the full shape of the data it's given?

### Code Prediction
```tsx
function List<Item extends { id: string }>({ items }: { items: Item[] }) {
  return <ul>{items.map(item => <li key={item.id} />)}</ul>;
}
<List items={[{ name: "Widget" }]} />
```
Does this compile? Why or why not, given the constraint on `Item`?

## Practical Tasks

- Type a controlled input component with a `ReactNode` label and a properly-typed change handler.
- Refactor a component with two incompatible optional props into a mutually exclusive union.
- Build a generic list component that accepts a typed render callback and enforces a stable key requirement via its constraint.

## Readiness Criteria

Type React component boundaries, DOM events, and generic components precisely without resorting to `any` or unsafe assertions, and explain why `React.FC` is avoidable rather than required.

## References

- [React TypeScript Cheatsheet: Basic Prop Types](https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example/)
- [React: Passing Props to a Component](https://react.dev/learn/passing-props-to-a-component)
- [TypeScript Handbook: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
