# Destructuring, Spread, Rest, Optional Chaining, and Nullish Coalescing

## Definition

Modern JavaScript syntax makes common data access and function signatures concise. Destructuring extracts values, spread expands an iterable or object, rest collects remaining values, optional chaining handles nullish paths, and nullish coalescing provides a fallback for absent values.

## How It Works

- Object and array destructuring bind selected properties or positions; defaults apply only when the extracted value is `undefined`.
- Object spread copies own enumerable properties into a new shallow object. Later properties win on key conflicts.
- Array spread expands iterable elements and produces a new shallow array.
- Rest parameters collect trailing function arguments into an array; rest properties collect remaining object properties.
- `value?.property` and `value?.()` return `undefined` when the guarded value is nullish.
- `left ?? right` evaluates `right` only when `left` is `null` or `undefined`.

## Application

Use these constructs to make data dependencies clear, not to compress complex logic. Destructure only fields a function needs, use spread for shallow immutable updates, and use `?.`/`??` for genuinely optional external data.

## Common Mistakes

- Assuming spread performs a deep clone.
- Using `||` when a valid value can be falsy.
- Destructuring a possibly nullish value without a guard or default.
- Combining `??` with `||` or `&&` without parentheses, which JavaScript intentionally rejects.

## Common Interview Questions

### Foundation

- What is the difference between spread and rest syntax?
- How do optional chaining and nullish coalescing work?

### Intermediate

- How would you update one shallow object property without mutation?
- When does a destructuring default apply?

### Advanced and Follow-up

- Why can spreading an API model still leave nested mutation shared?

### Code Prediction

Predict the result of `{ ...defaults, ...input }` when both objects contain the same property and of `input?.address?.city ?? "Unknown"` for absent fields.

## Practical Tasks

- Extract a component's needed fields without leaking the entire model through its API.
- Apply a shallow immutable update and identify when a nested copy is still required.

## Readiness Criteria

You can use modern syntax to express optional data and shallow updates without hiding mutation or business rules.

## References

- [MDN: Destructuring assignment](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment)
- [MDN: Optional chaining](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Optional_chaining)