# Component Identity, Composition, and State

## Definition

React preserves state according to a component's position and identity in the rendered tree. Composition and lifting state are ways to place behavior and ownership where it can be shared without making every component depend on a global store.

## How It Works

- React associates state with a component type and its position in the tree.
- Changing a component's key or position can cause React to preserve, reset, or move state depending on the resulting identity.
- Composition passes UI or behavior through props such as `children` instead of forcing a parent to know every internal detail.
- Lifting state moves shared state to the nearest common owner that can coordinate the relevant updates.
- Derived values should usually be calculated during render rather than stored separately, unless there is a clear external synchronization reason.

## Application

Choose state ownership by asking which components need to read and update the value. Use composition to keep component boundaries flexible and avoid turning a simple feature into a web of callbacks or context dependencies.

## Common Mistakes

- Storing a value that can be derived from existing props or state.
- Resetting state accidentally by changing a key or component type.
- Lifting all state to the top of the application instead of the nearest shared owner.
- Using context to solve a local composition problem.
- Passing too many unrelated props because component responsibilities are unclear.

## Common Interview Questions

### Foundation

- What determines whether React preserves component state?
- What does lifting state mean?

### Intermediate

- When is composition preferable to prop drilling or context?
- Why is duplicated derived state dangerous?

### Advanced and Follow-up

- How would you intentionally reset a form when its record ID changes?
- How do you choose the nearest appropriate state owner for a value used by sibling components?

### Code Prediction

Given a form rendered as `<Editor key={recordId} />`, explain what happens to the form state when `recordId` changes and why the key changes the result.

## Practical Tasks

- Refactor a component that stores duplicated filtered data into one source of truth and a derived render value.
- Design a parent/child API using composition instead of passing layout-specific flags through several layers.

## Readiness Criteria

You can explain React state identity, composition, lifting state, derived values, and deliberate state ownership without defaulting to global state.

## References

- [React: Preserving and resetting state](https://react.dev/learn/preserving-and-resetting-state)
- [React: Passing props to a component](https://react.dev/learn/passing-props-to-a-component)
- [React: Sharing state between components](https://react.dev/learn/sharing-state-between-components)
