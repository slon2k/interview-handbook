# Narrowing, Type Guards, Discriminated Unions, and Exhaustive Checks

## Definition

Narrowing refines a broad type to a more specific type using runtime checks that TypeScript understands. Type guards include `typeof`, `in`, equality checks, `instanceof`, and user-defined predicates. A discriminated union uses a shared literal field to identify its current alternative.

## How It Works

- Control-flow analysis tracks checks through branches, early returns, and assignments.
- A user-defined guard such as `value is User` tells the compiler what is true when the function returns `true`; the implementation must actually uphold that claim.
- A `switch` over a discriminant makes state-specific fields available in each branch.
- An `assertNever` helper can make a switch fail to compile when a new union member is not handled.
- Narrowing is based on runtime evidence; a type assertion bypasses that evidence instead of creating it.

## Application

Use discriminated unions for async state, API results, and UI modes. Make impossible states difficult to represent and make adding a new state produce compiler errors at every required handling point.

## Common Mistakes

- Narrowing on a field that is not actually reliable at runtime.
- Writing a type guard that returns `true` without checking the claimed shape.
- Adding a default branch that silently ignores a new discriminated-union member.
- Confusing a type assertion with validation.

## Common Interview Questions

### Foundation

- What is type narrowing?
- How does a discriminated union work?

### Intermediate

- How would you write a type guard for unknown API data?
- How can a switch be made exhaustive?

### Advanced and Follow-up

- What makes a user-defined type guard dangerous when its implementation is incomplete?

### Code Prediction

Given a `Result` union discriminated by `status`, identify which fields are available in each `switch` branch and what an `assertNever` call catches.

## Practical Tasks

- Model loading, success, empty, and error states as a discriminated union.
- Add exhaustive handling to a state switch and observe the compiler when a new state is introduced.

## Readiness Criteria

You can narrow safely from unions or `unknown`, write honest guards, and use exhaustive checks to protect evolving state models.

## References

- [TypeScript Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
- [TypeScript Handbook: Discriminated unions](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions)