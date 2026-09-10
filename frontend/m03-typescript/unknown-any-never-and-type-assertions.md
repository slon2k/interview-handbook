# `unknown`, `any`, `never`, and Type Assertions

## Definition

`unknown` represents a value whose type is not yet known and requires narrowing before use. `any` disables most checking. `never` represents an impossible value or a function that does not complete normally. Assertions tell the compiler to trust the author but do not perform runtime conversion or validation.

## How It Works

- Prefer `unknown` for parsed JSON, caught errors, plugin values, and other untrusted inputs.
- `any` is contagious: operations on it are accepted and can make unsafe values flow through otherwise typed code.
- A function returning `never` may throw forever or contain an infinite loop; an `assertNever` helper uses it for exhaustiveness.
- `value as User` changes the static view only. It does not check that `value` has `User`'s fields.
- Non-null assertions (`value!`) suppress a possible nullish error and should be reserved for an invariant proven elsewhere.

## Application

Treat assertions as boundary escape hatches that deserve a nearby explanation or a stronger runtime check. Prefer narrowing, validation, or a better model over making the compiler quiet.

## Common Mistakes

- Replacing every type error with `as SomeType`.
- Using `any` for a difficult generic or library boundary without containing it.
- Assuming `as number` converts a string to a number.
- Using `!` for a value whose presence depends on user input or an API response.

## Common Interview Questions

### Foundation

- What is the difference between `unknown` and `any`?
- What does a type assertion do at runtime?

### Intermediate

- When is `never` useful in application code?
- How would you handle an error caught from an unknown source?

### Advanced and Follow-up

- How would you contain an unavoidable third-party `any` without spreading it through the application?

### Code Prediction

Predict which operations compile on `unknown`, `any`, and `never`, and explain what happens when `as User` is applied to an object missing required fields.

## Practical Tasks

- Replace an `any` API response with `unknown` and add a narrow validation path.
- Remove non-null assertions from a UI model by representing absent and present states explicitly.

## Readiness Criteria

You can choose `unknown`, `any`, `never`, narrowing, validation, or an assertion based on the actual trust boundary and risk.

## References

- [TypeScript Handbook: The `unknown` type](https://www.typescriptlang.org/docs/handbook/2/functions.html#unknown)
- [TypeScript Handbook: The `never` type](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking)
- [TypeScript Handbook: Type assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions)