# Nullability and Strict Compiler Settings

## Definition

TypeScript can model `null` and `undefined` explicitly when strict null checking is enabled. Strict compiler settings make unsafe assumptions visible, turning many runtime failures into earlier design decisions.

## How It Works

- With `strictNullChecks`, `string` and `string | undefined` are different types.
- Optional properties and optional parameters introduce possible `undefined`; nullish checks or narrowing are required before use.
- `noImplicitAny` prevents unannotated values from silently becoming `any`.
- `strictFunctionTypes`, `noUncheckedIndexedAccess`, and `exactOptionalPropertyTypes` expose different classes of unsound assumptions.
- TypeScript's nullability is static and does not make a server response, DOM lookup, or user input non-null at runtime.
- C# nullable reference types are also primarily compiler analysis, but the languages differ in syntax, runtime behavior, and type-system rules; do not transfer assumptions mechanically.

## Application

Enable strictness early and fix the model rather than weakening the setting. Represent loading, missing, and present data deliberately so UI code does not accumulate `!` assertions and unchecked indexing.

## Common Mistakes

- Disabling strict null checks because existing code has errors.
- Confusing an optional property with a property whose value is always present but may be `undefined`.
- Assuming an array index returns a guaranteed element.
- Treating TypeScript nullability as runtime validation.

## Common Interview Questions

### Foundation

- What does `strictNullChecks` change?
- How are `null` and `undefined` represented in a TypeScript type?

### Intermediate

- Why might an array access be unsafe even when the array type is `User[]`?
- What does `noUncheckedIndexedAccess` protect against?

### Advanced and Follow-up

- How would you migrate a non-strict codebase without hiding all errors behind assertions?

### Code Prediction

Predict whether `const name: string = user.name` compiles when `name` is optional and strict null checking is enabled.

## Practical Tasks

- Enable strict null checks in a sample configuration and repair the resulting model errors.
- Model an optional API field whose absence differs from an explicit empty value.

## Readiness Criteria

You can explain strict nullability, choose relevant strict flags, and represent absent values without pretending they are always present.

## References

- [TypeScript Handbook: Strict null checks](https://www.typescriptlang.org/tsconfig/strictNullChecks.html)
- [TypeScript: TSConfig strict](https://www.typescriptlang.org/tsconfig/strict.html)