# Objects, Arrays, References, and Mutation

## Definition

Objects and arrays are mutable reference values. Assigning or passing one does not copy its contents; it gives another binding access to the same value. A shallow copy creates a new outer container but preserves references to nested values.

## How It Works

- Property assignment and mutating array methods such as `push`, `sort`, and `splice` change an existing value.
- Object spread (`{ ...value }`) and array spread (`[...values]`) create shallow copies.
- Nested objects remain shared after a shallow copy; copy each changed level to update nested data safely.
- Prefer non-mutating array methods such as `map`, `filter`, and `toSorted` when deriving UI data.
- `const` prevents rebinding a variable but does not make an object immutable.

## Application

Treat values held in React state, props, caches, and shared application state as immutable. Create a new object or array when expressing an update so consumers can reliably distinguish old state from new state.

## Common Mistakes

- Calling `sort` on a prop or state array and assuming it returns a copy.
- Shallow-copying an outer object then mutating a nested object.
- Treating `const` as deep immutability.

## Common Interview Questions

### Foundation

- What does it mean that objects are reference values?
- Which common array methods mutate their receiver?

### Intermediate

- Why can `{ ...user }` still allow an accidental nested mutation?
- How would you update one item in an array without mutating the original?

### Advanced and Follow-up

- Why does immutability matter for UI rendering and memoisation?

### Code Prediction

Given `const copy = { ...user }`, predict whether changing `copy.address.city` changes `user.address.city`.

## Practical Tasks

- Repair a reducer-style update that mutates a nested state object.
- Derive a sorted display list without changing the API result.

## Readiness Criteria

You can predict aliasing, distinguish shallow and deep copies, and express UI data updates without mutation.

## References

- [MDN: Working with objects](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Working_with_objects)
- [MDN: Array](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array)