# Module 3 - TypeScript

**Status:** Complete  
**Priority:** Critical  
**Prerequisites:** [Module 2 - JavaScript Language and Runtime](../m02-javascript-language-and-runtime/README.md)

## Scope

This module establishes TypeScript as the typed layer between JavaScript runtime behavior and the browser, API, and React modules that follow. It focuses on modeling values and boundaries precisely, using the compiler as a design tool, and recognizing where static types stop helping.

TypeScript types are erased from ordinary JavaScript output. They describe what code is expected to receive and return during development, but they do not validate JSON, API responses, form input, or other runtime data by themselves.

## Why This Matters in Interviews

A strong frontend candidate can read a type error as useful design feedback, model changing UI states without lying to the compiler, and explain why an apparently safe type assertion does not make external data trustworthy. Interviewers commonly probe structural typing, narrowing, generics, nullability, and the boundary between compile-time models and runtime values.

## Learning Outcomes

By the end of this module, you should be able to:

- Use inference and annotations deliberately without typing every expression redundantly.
- Explain structural typing, interfaces, type aliases, excess-property checks, and `readonly` data.
- Model alternatives with unions, intersections, literal types, optional properties, and discriminated unions.
- Narrow `unknown` values safely and use type guards and exhaustive checks.
- Explain the risks of `any`, assertions, non-null assertions, and careless compiler configuration.
- Model nullability and optional API fields precisely, including important differences from C#.
- Write reusable function and generic types using constraints, `keyof`, indexed access, mapped types, conditional types, and utility types.
- Type UI state, forms, callbacks, async results, and ASP.NET Core API contracts without overusing assertions.
- Validate untrusted runtime data at the boundary instead of treating a TypeScript annotation as validation.

## Topics

### 1. TypeScript Foundations

- [Type inference, annotations, and compiler feedback](type-inference-annotations-and-compiler-feedback.md)
- [Structural typing, interfaces, type aliases, and readonly data](structural-typing-interfaces-and-type-aliases.md)
- [Unions, intersections, literal types, and optional properties](unions-intersections-literals-and-optional-properties.md)

### 2. Narrowing and Nullability

- [Narrowing, type guards, discriminated unions, and exhaustive checks](narrowing-type-guards-and-discriminated-unions.md)
- [unknown, any, never, and type assertions](unknown-any-never-and-type-assertions.md)
- [Nullability and strict compiler settings](nullability-and-strict-compiler-settings.md)

### 3. Functions and Type-Level Reuse

- [Function types, overloads, and generics](function-types-overloads-and-generics.md)
- [keyof, typeof, indexed access, and utility types](keyof-typeof-indexed-access-and-utility-types.md)
- [Mapped and conditional types](mapped-and-conditional-types.md)

### 4. Application Boundaries

- [Typing UI state, forms, callbacks, and async results](typing-ui-state-forms-and-async-results.md)
- [API DTOs, validation errors, and runtime validation](api-dtos-and-runtime-validation.md)

### 5. Ecosystem Awareness

- [Modules, declaration files, enums, and variance](modules-declaration-files-enums-and-variance.md)

## Scope Boundaries

- JavaScript values, closures, promises, and event-loop behavior belong in [Module 2](../m02-javascript-language-and-runtime/README.md); TypeScript adds static modeling on top of those runtime rules.
- DOM events, `fetch`, browser storage, rendering, and API integration belong in Module 4 - Browser Platform and ASP.NET Core API Integration.
- React props, hooks, component generics, and React event types belong in Module 5 - React; this module covers framework-independent UI and async modeling.
- `tsconfig`, npm scripts, linting, bundling, and build delivery belong in Module 6 - Frontend Testing and Tooling; this module explains strictness concepts only where they affect type safety.

## Suggested Learning Sequence

1. Learn inference and annotations, then structural typing and the basic type-building blocks.
2. Model alternatives with unions and narrow them safely using control flow and discriminants.
3. Handle unknown data, assertions, nullability, and strict compiler feedback.
4. Build reusable function, generic, utility, mapped, and conditional types.
5. Apply those models to UI state and API boundaries, ending with runtime validation and ecosystem awareness.

## Practical Deliverables

- Refactor a loosely typed data model into a discriminated union with exhaustive handling.
- Repair a function that uses `any` or a non-null assertion to hide an unsafe assumption.
- Type a form state model with meaningful empty, editing, submitting, success, and error states.
- Model a paginated ASP.NET Core response and its field-level validation errors.
- Parse untrusted JSON through a runtime validation boundary and explain why the TypeScript type alone was insufficient.

## Interview Coverage

Each topic includes foundation, intermediate, and advanced questions plus a code-prediction prompt. Practise explaining what the compiler knows, what it can narrow, and what remains unchecked once JavaScript runs.

## References

- [TypeScript Handbook](https://www.typescriptlang.org/docs/handbook/intro.html)
- [TypeScript: TSConfig Reference](https://www.typescriptlang.org/tsconfig)
- [TypeScript Playground](https://www.typescriptlang.org/play)