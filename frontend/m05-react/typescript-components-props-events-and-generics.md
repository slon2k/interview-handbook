# TypeScript Components, Props, Events, and Generics

## Definition

TypeScript makes a React component's inputs and outputs explicit: props describe what a caller must provide, event handlers describe which DOM event a callback receives, and generics let a reusable component preserve the type of the data it renders. Prefer an ordinary function with an explicit props type; `React.FC` is optional, not required.

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

Use `ReactNode` when a prop accepts renderable content such as text, elements, fragments, or arrays of those values. Use a more specific element type only when the component truly needs to inspect or clone that element. Type handlers by the element that emits them, rather than giving their event an implicit `any`.

### Mutually exclusive props and render callbacks

```tsx
type TextLabelProps = { label: string; renderLabel?: never };
type RenderLabelProps = { label?: never; renderLabel: () => ReactNode };
type LabelProps = TextLabelProps | RenderLabelProps;

function Label({ label, renderLabel }: LabelProps) {
  return <span>{renderLabel ? renderLabel() : label}</span>;
}
```

A discriminated or mutually exclusive union prevents callers from supplying incompatible alternatives. For a render callback, type both the arguments it receives and the renderable value it returns.

### Generic components and wrapper props

```tsx
type ListProps<Item extends { id: string }> = {
  items: readonly Item[];
  renderItem: (item: Item) => ReactNode;
};

function List<Item extends { id: string }>({ items, renderItem }: ListProps<Item>) {
  return <ul>{items.map(item => <li key={item.id}>{renderItem(item)}</li>)}</ul>;
}
```

Use a constraint only for the properties the component needs. A wrapper around a native element can inherit its standard props without repeating them:

```tsx
import type { ComponentPropsWithoutRef } from "react";

type PrimaryButtonProps = ComponentPropsWithoutRef<"button"> & {
  tone?: "primary" | "danger";
};
```

`forwardRef` is useful when a component deliberately exposes a DOM node to its caller, such as a reusable input. Treat it as an interoperability pattern, not a default component shape.

### Reducers, context, and custom hooks

```tsx
type CounterAction =
  | { type: "increment" }
  | { type: "set"; value: number };

function counterReducer(state: number, action: CounterAction): number {
  switch (action.type) {
    case "increment":
      return state + 1;
    case "set":
      return action.value;
  }
}
```

Use discriminated actions so a reducer can narrow action-specific payloads. Give a custom hook a small, named return contract. A context whose value may be absent should be accessed through a custom hook that throws a clear error outside its provider, instead of leaking `T | undefined` across every consumer.

## Application

Keep the TypeScript contract at the component boundary. Use the framework-independent state models from [Module 3](../m03-typescript/typing-ui-state-forms-and-async-results.md) for props and hook results, then use React types only for JSX, DOM events, children, and ref interoperability.

## Common Mistakes

- Defaulting to `React.FC` because it appears in older examples, instead of writing the clearest component signature.
- Giving event handlers an implicit `any` or using a click event type for a change handler.
- Using `ReactNode` when a component requires exactly one inspectable React element.
- Writing a generic component with no constraint, then assuming every item has an `id` for its key.
- Publishing a context value that can be `undefined` without a provider-checking hook.
- Using a type assertion to claim that untrusted API data matches a prop type instead of validating it at the Module 3 boundary.

## Common Interview Questions

### Foundation

- Why is a plain function with typed props usually sufficient for a React component?
- When would a prop use `ReactNode`?

### Intermediate

- How would you type an input change handler and a form submit handler?
- How would you prevent a component from accepting both a text label and a render callback?

### Advanced and Follow-up

- How would you type a reusable list while preserving the caller's item type?
- When is `forwardRef` appropriate, and why should it not be the default?

### Code Prediction

Given a `List<Item extends { id: string }>` component, explain why passing items with only `{ name: string }` fails before rendering and how the constraint protects the list key.

## Practical Tasks

- Type a controlled input component with a `ReactNode` label and an input change handler.
- Refactor a component with incompatible optional props into a mutually exclusive union.
- Build a generic list that accepts a typed render callback and uses a stable key.
- Type a reducer action union and add a context hook that reports a missing provider clearly.

## Readiness Criteria

You can type React component boundaries, DOM events, render callbacks, generic component data, reducers, contexts, and custom-hook contracts without resorting to `any` or unsafe assertions.

## References

- [React TypeScript Cheatsheet: Basic prop types](https://react-typescript-cheatsheet.netlify.app/docs/basic/getting-started/basic_type_example/)
- [React: Passing props to a component](https://react.dev/learn/passing-props-to-a-component)
- [TypeScript Handbook: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
