# Components, JSX, Props, and Rendering

## Definition

A React component is a function that receives inputs, usually called props, and returns a description of UI. JSX is syntax that lets component code describe element structure while remaining JavaScript and TypeScript.

## How It Works

- A component receives props from its parent and returns elements, other components, or `null`.
- JSX expressions are evaluated during rendering; they do not create a separate template language runtime.
- Conditional rendering can use normal JavaScript expressions, but `0` and empty strings have different rendering behavior from `false` and `null`.
- Event handlers are passed as functions and are invoked by React in response to user interaction.
- Lists should provide stable keys so React can associate rendered items with their identity.

## Application

Keep components focused on describing a coherent piece of UI and its inputs. Use typed props to make required data and callback contracts visible, and keep display-only transformations close to the component boundary. JSX should still emit semantic HTML: a React component does not make a `div` an appropriate replacement for a native button, label, or landmark. See [Module 1: Semantic HTML](../m01-web-platform-foundations/semantic-html-and-document-structure.md).

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

Type required and optional props deliberately, and type a handler by the DOM element that emits it. See [TypeScript components, props, events, and generics](typescript-components-props-events-and-generics.md) for render callbacks, generic components, reducers, context, and ref interoperability.

## Common Mistakes

- Calling a handler while rendering instead of passing a function.
- Using array indexes as keys for reorderable or insertable lists.
- Mutating props or state while building JSX.
- Assuming a falsy value such as `0` will disappear from a conditional expression.
- Making a component responsible for unrelated data loading, layout, and domain decisions.
- Leaving a component's props or event callback untyped, forcing callers to infer its contract from its implementation.
- Using a generic `div` for a semantic control because JSX makes it convenient to attach an event handler.

## Common Interview Questions

### Foundation

- What is a React component?
- What is JSX transformed into conceptually?
- How are props different from state?

### Intermediate

- Why must list keys be stable and unique among siblings?
- Why does `{count && <Badge />}` render `0` when `count` is zero?
- What is the difference between typing a click handler and typing an input change handler?

### Advanced and Follow-up

- How would you type a component that accepts either children or a render callback?
- When should a large component be split into smaller components?
- How would you type props that permit either a text label or a render callback, but not both?

### Code Prediction

Given a list that uses the array index as a key and allows insertion at the beginning, predict which row keeps a local input value after the insertion and explain why.

## Practical Tasks

- Convert a loosely typed component into a typed function component with explicit props and callback types.
- Review a list component and replace unstable keys with domain identifiers.
- Replace a clickable `div` with a typed native button while preserving its visual styling.

## Readiness Criteria

You can describe the relationship between JSX, props, state, rendering, event handlers, and stable list identity, and you can spot common component-boundary mistakes.

## References

- [React: Your first component](https://react.dev/learn/your-first-component)
- [React: Passing props to a component](https://react.dev/learn/passing-props-to-a-component)
- [React: Rendering lists](https://react.dev/learn/rendering-lists)
