# Mapped and Conditional Types

## Definition

Mapped types iterate over a set of property keys to create a new object type. Conditional types select one type or another based on an assignability test and can infer part of a type with `infer`.

## How It Works

- A mapped type such as `{ [K in keyof T]: boolean }` preserves the keys of `T` while changing their values.
- Mapped types can add or remove modifiers such as `readonly` and optionality.
- Conditional types use `T extends U ? Yes : No` and can distribute over a naked union type parameter.
- `infer` captures a related type inside a conditional pattern, such as an array element or promise result.
- Many utility types are built from mapped and conditional types; understanding the mechanism helps when reading library declarations.

## Application

Use mapped types for systematic model transformations such as field metadata or patch state. Use conditional types when a reusable type genuinely depends on its input shape; prefer a named simpler type when the result becomes difficult to explain.

## Common Mistakes

- Writing type-level machinery where a runtime function or a simple explicit type would be clearer.
- Forgetting that conditional types can distribute across unions.
- Assuming a mapped type changes the runtime object.
- Creating deeply recursive types that slow the compiler and confuse users.

## Common Interview Questions

### Foundation

- What does a mapped type do?
- What is a conditional type?

### Intermediate

- How would you make every property of a model nullable or optional?
- What does `infer` capture?

### Advanced and Follow-up

- Why can a conditional type behave differently for a union than expected?
- When should a type-level abstraction be replaced with a simpler named type?

### Code Prediction

Predict the result of `type Flags<T> = { [K in keyof T]: boolean }` for a model containing `id` and `name`.

## Practical Tasks

- Create a mapped type for field-level dirty or touched metadata.
- Use a conditional type to extract an element type from an array or a resolved value from a promise.

## Readiness Criteria

You can read and write modest mapped and conditional types and recognize when type-level complexity is no longer helping the design.

## References

- [TypeScript Handbook: Mapped types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
- [TypeScript Handbook: Conditional types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)