# Array Methods and Immutable Data Transformations

## Definition

Array methods express common collection operations. Some derive a new result, while others change the existing array. Choosing the correct method makes UI data transformations readable and protects source data from unintended mutation.

## How It Works

- `map` transforms every element; `filter` keeps matching elements; `find` returns the first matching element or `undefined`.
- `some` and `every` test predicates; `includes` tests membership using SameValueZero equality.
- `reduce` combines entries into one result but is not a substitute for clearer `map`, `filter`, or loops.
- `forEach` returns `undefined` and is for side effects, not derived values.
- `sort`, `reverse`, `splice`, `push`, and `pop` mutate; prefer `toSorted`, `toReversed`, `toSpliced`, or a copy when available.
- Callbacks receive value, index, and array, but use only the parameters that make the transformation understandable.

## Application

Build display models, filtered results, selected-item updates, and grouped data with non-mutating transformations. Keep transformations pure so they are safe to reuse in rendering and tests.

## Common Mistakes

- Using `forEach` where `map` is required.
- Mutating with `sort` before rendering a prop or state array.
- Calling `reduce` for a simple lookup or filter.
- Forgetting that `find` can return `undefined`.

## Common Interview Questions

### Foundation

- How do `map`, `filter`, and `find` differ?
- Which array methods mutate their receiver?

### Intermediate

- How would you update one record in an array immutably?
- When is `reduce` appropriate?

### Advanced and Follow-up

- Why do pure transformations make React rendering and testing easier?

### Code Prediction

Predict the return values of `forEach`, `map`, and `find` for a callback that matches no items.

## Practical Tasks

- Transform API records into sorted display rows without mutating the source array.
- Replace a side-effecting loop with the clearest array transformation.

## Readiness Criteria

You can choose an array method by its result shape and derive UI data without accidental mutation.

## References

- [MDN: Indexed collections](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Indexed_collections)
- [MDN: Array methods](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Global_Objects/Array)