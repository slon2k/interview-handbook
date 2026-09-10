# Type Inference, Annotations, and Compiler Feedback

## Definition

TypeScript can infer many types from initializers, return expressions, and surrounding context. An annotation states an intended type explicitly. The compiler compares the inferred or declared model with how the value is used and reports mismatches before runtime.

## How It Works

- `const count = 3` infers a useful literal-aware type in some contexts; `let count = 3` generally widens to `number` because reassignment is allowed.
- Function parameters usually need annotations because callers provide their values.
- Return types can be inferred, but explicit public return types document and constrain a function's contract.
- Contextual typing lets TypeScript infer a callback parameter from the array or API receiving the callback.
- An annotation checks a value; it does not convert the value or validate data received at runtime.
- Let inference describe local implementation details and annotate boundaries, exported APIs, and ambiguous values.

## Application

Use compiler errors as feedback about an unclear or inconsistent model. Prefer a useful correction to silencing the error with `any` or an assertion; the goal is to make invalid states harder to express while keeping ordinary code readable.

## Common Mistakes

- Adding annotations to every local variable and obscuring the expression's meaning.
- Assuming `const` makes an object immutable.
- Treating a type annotation on parsed JSON as runtime validation.
- Omitting parameter types from exported functions and accepting accidental `any` through configuration.

## Common Interview Questions

### Foundation

- What is type inference?
- When should a function parameter or return value be annotated?

### Intermediate

- Why can `let` and `const` infer different types for the same initializer?
- What is contextual typing?

### Advanced and Follow-up

- How would you use an explicit return type to prevent a public function from accidentally widening its contract?

### Code Prediction

Predict the inferred types of `const value = "ready"`, `let mutableValue = "ready"`, and `const values = []` under strict TypeScript settings.

## Practical Tasks

- Add boundary annotations to a small data-loading module while leaving obvious local types inferred.
- Replace an `any` introduced by a missing annotation with a type that reflects the actual input.

## Readiness Criteria

You can distinguish inference from annotation, use both at appropriate boundaries, and explain why neither changes or validates runtime values.

## References

- [TypeScript Handbook: Everyday types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [TypeScript Handbook: Type inference](https://www.typescriptlang.org/docs/handbook/type-inference.html)