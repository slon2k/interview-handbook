# Function Types, Overloads, and Generics

## Definition

Function types describe parameters and return values. Overloads expose several supported call signatures with one implementation. Generics preserve relationships between input and output types instead of replacing precise types with `any`.

## How It Works

- Function type syntax describes callable values, including callback parameters and return types.
- Optional and rest parameters affect which calls are valid; parameter variance determines whether a callback can safely be substituted.
- Overload signatures describe the public API; the implementation signature must handle every overload but is not directly visible to callers.
- A generic type parameter is inferred from arguments when possible and can be constrained with `extends`.
- Use a generic when the output type depends on an input type or when several types share one operation; use a union when the alternatives have different behavior.

## Application

Type callbacks and reusable data helpers at their public boundaries. Keep generic APIs small and inferable; a complicated generic that callers must manually annotate can be less useful than a clear concrete function.

## Common Mistakes

- Using `any` where a generic would preserve a value's type.
- Writing overloads whose implementation does not handle every declared case.
- Adding unconstrained generics that provide no meaningful relationship.
- Choosing overloads when a discriminated union would be clearer.

## Common Interview Questions

### Foundation

- What is a generic type parameter?
- When would you use an overload?

### Intermediate

- How does a constraint such as `T extends { id: string }` help?
- How would you type a callback that receives a row and returns a label?

### Advanced and Follow-up

- When is a union preferable to overloads or a generic?

### Code Prediction

Given `function first<T>(items: T[]): T | undefined`, predict the inferred return type when it receives `User[]` and when it receives `string[]`.

## Practical Tasks

- Replace an `any`-based identity or collection helper with an inferred generic.
- Add overloads to a parser with two deliberate input forms and one safe implementation.

## Readiness Criteria

You can describe function contracts, choose between overloads, unions, and generics, and preserve input/output relationships safely.

## References

- [TypeScript Handbook: More on functions](https://www.typescriptlang.org/docs/handbook/2/functions.html)
- [TypeScript Handbook: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)