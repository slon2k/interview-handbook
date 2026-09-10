# Structural Typing, Interfaces, Type Aliases, and Readonly Data

## Definition

TypeScript is structurally typed: a value is assignable when it has the required shape, regardless of whether it explicitly declares that it implements a named type. Interfaces and type aliases both describe shapes, while `readonly` communicates that a property or collection should not be mutated through that type.

## How It Works

- An object with additional properties can usually be assigned to a variable requiring fewer properties, because the required structure is present.
- Object literals receive stricter excess-property checks at direct assignment and function calls; storing the literal first can change that check.
- Interfaces are open to declaration merging; type aliases can describe unions, tuples, primitives, and intersections directly.
- `readonly` is a compile-time restriction on writes, not a deep runtime freeze.
- Structural compatibility is convenient for adapters and test doubles but can allow accidental compatibility between conceptually different IDs or records.

## Application

Use interfaces or aliases to name meaningful boundaries, not every incidental object. Add branded or discriminated representations when two values share a primitive shape but must not be mixed accidentally.

## Common Mistakes

- Assuming TypeScript nominally distinguishes two object types with the same fields.
- Expecting `readonly` to prevent runtime mutation.
- Being surprised by excess-property checks only on fresh object literals.
- Using interfaces and aliases as if they have materially different runtime behavior; both disappear at runtime.

## Common Interview Questions

### Foundation

- What does structural typing mean?
- How do interfaces and type aliases differ?

### Intermediate

- Why can a variable containing extra properties be assignable where a fresh object literal is rejected?
- What does `readonly` guarantee?

### Advanced and Follow-up

- How would you prevent a `UserId` string from being passed where an `OrderId` is required?

### Code Prediction

Given a function requiring `{ id: number }`, predict why a variable with `{ id: 1, label: "A" }` is accepted while a fresh literal with an unexpected misspelled property may be rejected.

## Practical Tasks

- Model a DTO and a view model as distinct named shapes without duplicating unrelated fields.
- Design a type-safe identifier representation for two domain IDs that are both strings at runtime.

## Readiness Criteria

You can explain structural compatibility, excess-property checks, interface/type-alias trade-offs, and the limits of `readonly`.

## References

- [TypeScript Handbook: Object types](https://www.typescriptlang.org/docs/handbook/2/objects.html)
- [TypeScript Handbook: Type compatibility](https://www.typescriptlang.org/docs/handbook/type-compatibility.html)