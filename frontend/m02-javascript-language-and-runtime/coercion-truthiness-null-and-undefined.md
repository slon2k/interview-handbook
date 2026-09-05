# Coercion, Truthiness, Null, and Undefined

## Definition

Coercion converts a value to another type during an operation. JavaScript conditions convert values to boolean, giving every value a truthy or falsy result. `undefined` usually represents an absent or uninitialised value; `null` is an explicit empty value.

## How It Works

- Falsy values are `false`, `0`, `-0`, `0n`, `""`, `null`, `undefined`, and `NaN`; arrays and objects are truthy.
- `==` performs coercion before comparing; `===` does not. Use `==` only when its conversion behaviour is explicitly intended.
- `||` falls back for any falsy left value; `??` falls back only for `null` or `undefined`.
- Optional chaining (`?.`) stops only when its left side is nullish, not when it is `0`, `false`, or an empty string.

## Application

Use `??` for defaults where `0`, `false`, and an empty string are valid data. Use explicit predicates for business rules instead of relying on truthiness when a value's allowed range matters.

## Common Mistakes

- Using `||` for a default and replacing a meaningful `0`.
- Treating an empty array as falsy.
- Using `typeof value === "object"` without separately excluding `null`.

## Common Interview Questions

### Foundation

- Which values are falsy?
- How do `null` and `undefined` differ?

### Intermediate

- Why is `count || 10` unsafe when zero is valid?
- When is `??` preferable to `||`?

### Advanced and Follow-up

- How would you model an optional API field whose empty string has meaning?

### Code Prediction

Predict `0 || 10`, `0 ?? 10`, `"" || "fallback"`, and `"" ?? "fallback"`.

## Practical Tasks

- Repair display defaults that incorrectly replace zero quantities and empty input.
- Replace ambiguous truthiness checks with conditions that state the intended rule.

## Readiness Criteria

You can predict common coercions and select nullish or truthy fallback behaviour deliberately.

## References

- [MDN: Truthy](https://developer.mozilla.org/docs/Glossary/Truthy)
- [MDN: Nullish coalescing](https://developer.mozilla.org/docs/Web/JavaScript/Reference/Operators/Nullish_coalescing)