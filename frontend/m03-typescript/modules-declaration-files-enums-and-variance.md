# Modules, Declaration Files, Enums, and Variance

## Definition

TypeScript extends JavaScript modules with type-only exports and declarations describing code that exists elsewhere. Declaration files model JavaScript libraries. Enums create named runtime values, while variance describes when generic or function types can safely substitute for one another.

## How It Works

- `import type` and `export type` make type-only dependencies explicit and avoid implying a runtime import.
- `.d.ts` files describe the types of JavaScript libraries, global APIs, or generated code without implementing them.
- Ambient declarations use `declare` to tell the compiler that a value exists at runtime; an incorrect declaration creates unsoundness.
- String literal unions are often simpler and more interoperable than enums; enums also emit runtime JavaScript and have their own reverse-mapping and interop behavior.
- Function parameter and return positions have variance rules; strict function type checking catches callbacks that cannot safely handle every input they may receive.

## Application

Prefer explicit module boundaries and literal unions for API values unless a runtime enum is required by an existing ecosystem. Treat declaration files as trust boundaries: verify them when they describe untyped or generated code.

## Common Mistakes

- Importing a type as a runtime value or assuming `import type` creates JavaScript output.
- Adding an ambient declaration without ensuring the value really exists at runtime.
- Choosing enums by habit when a literal union is enough.
- Passing a callback that accepts a narrower type than the caller may provide.

## Common Interview Questions

### Foundation

- What is a declaration file?
- What does `import type` change?

### Intermediate

- When would you choose a literal union over an enum?
- What problem does strict function variance prevent?

### Advanced and Follow-up

- How can an inaccurate `.d.ts` file make a typed application unsafe?

### Code Prediction

Predict whether a type-only import appears in emitted JavaScript and what happens when an ambient declaration describes a missing runtime global.

## Practical Tasks

- Add a minimal declaration file for a small untyped JavaScript helper and test its boundary.
- Replace an unnecessary enum used only for API string values with a literal union.

## Readiness Criteria

You can distinguish type-only and runtime module behavior, recognize declaration-file risks, and explain enum and variance trade-offs at awareness level.

## References

- [TypeScript Handbook: Modules](https://www.typescriptlang.org/docs/handbook/2/modules.html)
- [TypeScript Handbook: Declaration files](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html)
- [TypeScript Handbook: Type compatibility](https://www.typescriptlang.org/docs/handbook/type-compatibility.html)