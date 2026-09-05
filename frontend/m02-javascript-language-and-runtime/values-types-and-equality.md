# Values, Types, and Equality

## Definition

JavaScript values are primitives or objects. Primitives include `string`, `number`, `bigint`, `boolean`, `symbol`, `undefined`, and `null`; objects include arrays, functions, dates, maps, and ordinary objects. Equality compares either value or identity depending on the operands and operator.

## How It Works

- Primitive assignments copy a value. Object assignments copy a reference to the same object.
- Use `===` and `!==` by default. They compare without converting operand types.
- `Object.is` differs from strict equality only in edge cases, notably `NaN` and signed zero.
- `NaN !== NaN`; use `Number.isNaN` when checking for it.
- Two separately-created object or array literals are never strictly equal, even when their contents match.

## Application

Use strict equality in UI conditions and API-result handling. When comparing structured data, define the required meaning first: identity, one selected field, or a content comparison. Do not expect `===` to perform deep equality.

## Common Mistakes

- Expecting `{ id: 1 } === { id: 1 }` to be true.
- Treating `typeof null === "object"` as evidence that `null` is an object.
- Comparing a value to `NaN` with `===`.

## Common Interview Questions

### Foundation

- Which JavaScript values are primitives?
- What is the difference between `===` and `Object.is`?

### Intermediate

- Why are two arrays with identical entries not `===`?
- How would you test whether a value is `NaN`?

### Advanced and Follow-up

- When should a UI compare object identity rather than a record's `id`?

### Code Prediction

Predict the results of `0 === -0`, `Object.is(0, -0)`, `NaN === NaN`, and `Object.is(NaN, NaN)`.

## Practical Tasks

- Explain why a selected-item check fails after rebuilding an array from API data.
- Replace coercive comparisons in a UI condition with deliberate strict comparisons.

## Readiness Criteria

You can distinguish value and identity equality and choose an equality check that matches the requirement.

## References

- [MDN: JavaScript data types](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Data_structures)
- [MDN: Equality comparisons](https://developer.mozilla.org/docs/Web/JavaScript/Guide/Equality_comparisons_and_sameness)