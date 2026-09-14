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

Keep components focused on describing a coherent piece of UI and its inputs. Use typed props to make required data and callback contracts visible, and keep display-only transformations close to the component boundary.

## Common Mistakes

- Calling a handler while rendering instead of passing a function.
- Using array indexes as keys for reorderable or insertable lists.
- Mutating props or state while building JSX.
- Assuming a falsy value such as `0` will disappear from a conditional expression.
- Making a component responsible for unrelated data loading, layout, and domain decisions.

## Common Interview Questions

### Foundation

- What is a React component?
- What is JSX transformed into conceptually?
- How are props different from state?

### Intermediate

- Why must list keys be stable and unique among siblings?
- Why does `{count && <Badge />}` render `0` when `count` is zero?

### Advanced and Follow-up

- How would you type a component that accepts either children or a render callback?
- When should a large component be split into smaller components?

### Code Prediction

Given a list that uses the array index as a key and allows insertion at the beginning, predict which row keeps a local input value after the insertion and explain why.

## Practical Tasks

- Convert a loosely typed component into a typed function component with explicit props and callback types.
- Review a list component and replace unstable keys with domain identifiers.

## Readiness Criteria

You can describe the relationship between JSX, props, state, rendering, event handlers, and stable list identity, and you can spot common component-boundary mistakes.

## References

- [React: Your first component](https://react.dev/learn/your-first-component)
- [React: Passing props to a component](https://react.dev/learn/passing-props-to-a-component)
- [React: Rendering lists](https://react.dev/learn/rendering-lists)
