# Modules, Declaration Files, Enums, and Variance

## Definition

TypeScript extends JavaScript's module system with type-only imports/exports. A **declaration file** (`.d.ts`) describes the types of code that already exists elsewhere — a JavaScript library, a global browser API, generated code — without implementing anything itself. **Enums** create named values that exist at runtime, unlike most other TypeScript constructs. **Variance** describes when one function or generic type can safely substitute for another.

```typescript
import type { User } from "./types"; // type-only import — guaranteed to produce NO runtime JavaScript at all
```

## Alternatives & Trade-offs

A string literal union (`"admin" | "editor" | "viewer"`) requires no runtime representation at all — it's purely a compile-time construct that vanishes entirely once compiled. A numeric or string `enum` creates an actual runtime object with real JavaScript output, which is necessary if the ecosystem you're working in (a library, a generated API client) specifically expects an enum — but for ordinary application-level finite values, it's usually more machinery than a literal union needs, for no corresponding benefit.

## How It Works

### `import type` — making a type-only dependency explicit

```typescript
import type { User } from "./types"; // this import is GUARANTEED to be erased entirely — zero runtime trace

import { User } from "./types"; // ambiguous to a reader: is `User` a type, a value, or both?
```

Explicitly marking a type-only import removes any ambiguity about whether that import has runtime weight, and lets some build tools optimize more confidently, knowing for certain the import can be safely elided.

### Declaration files describe code you didn't write — and can be wrong

```typescript
// some-untyped-library.d.ts
declare module "some-untyped-library" {
  export function process(input: string): number; // a CLAIM about what this library does — not verified by TypeScript at all
}
```

```typescript
import { process } from "some-untyped-library";
const result = process(42); // if the .d.ts is WRONG about process's real signature, TypeScript happily
                               // accepts this call anyway — the declaration file IS the trust boundary here
```

A declaration file is exactly as trustworthy as whoever wrote it — an inaccurate `.d.ts` describing a library incorrectly makes an otherwise fully type-checked application unsafe at exactly the points that rely on it, with the compiler having no way to know the description is wrong.

### `declare` — telling the compiler a runtime value exists, without proving it

```typescript
declare const __APP_VERSION__: string; // asserts this global exists at runtime — e.g., injected by a build tool

console.log(__APP_VERSION__.toUpperCase()); // compiles fine — but if the build tool ISN'T actually configured
                                               // to inject this global, this crashes at runtime with no earlier warning
```

`declare` is a promise to the compiler, not a runtime guarantee — if the described global genuinely doesn't exist when the code runs, this is a runtime `ReferenceError`, not something TypeScript could have caught.

### Enums vs. literal unions — real runtime output vs. purely compile-time

```typescript
enum Role { Admin, Editor, Viewer } // produces an ACTUAL JavaScript object at runtime, with numeric values by default

console.log(Role.Admin); // 0 — a real runtime value
console.log(Role[0]);       // "Admin" — enums also support REVERSE mapping by default, another runtime behavior

type RoleLiteral = "admin" | "editor" | "viewer"; // compiles away to NOTHING — purely a compile-time construct
```

A numeric enum's implicit reverse mapping and its exact emitted JavaScript shape are real, ongoing interop details to be aware of — a literal union has none of this because it simply doesn't exist once compiled, which is exactly why literal unions are usually the simpler, preferred default for application-level finite values.

### Strict function variance — catching a callback that can't safely handle every input

```typescript
type Handler = (event: MouseEvent) => void;

function attach(handler: Handler) { }

const specificHandler = (event: { clientX: number }) => { }; // ACCEPTS LESS than a full MouseEvent needs
attach(specificHandler); // with strict function types: ERROR — this handler can't safely handle every real MouseEvent
```

Strict function type checking catches a callback whose parameter type is narrower than what it will actually be called with — `specificHandler` only claims to need `{ clientX: number }`, but if it were actually invoked with a full `MouseEvent`, it might work by luck, or might not, depending on what else the (unwritten) implementation assumes.

## Application

Use `import type`/`export type` to make type-only dependencies explicit wherever a build tool or a reader benefits from that clarity. Treat every declaration file as a trust boundary — verify one describing an untyped or generated library rather than assuming it's automatically correct. Default to literal unions for application-level finite values; reach for an enum specifically when an existing ecosystem (a generated client, a specific library) already expects one.

## Common Mistakes

- Importing a type as if it were a runtime value, or assuming `import type` produces JavaScript output when it's specifically guaranteed not to.
- Adding an ambient `declare` for a global without actually verifying the value exists at runtime in every environment the code will run in.
- Choosing an enum out of habit for a simple, application-level finite value where a literal union would be simpler and have zero runtime footprint.
- Passing a callback that only handles a narrower type than what it will genuinely be invoked with, when strict function type checking would have caught the mismatch.

## Common Interview Questions

### Basic
- What is a declaration file, and what problem does it solve?
- What does `import type` change compared to an ordinary import?

### Intermediate
- When would you choose a literal union over an enum for a finite set of values?
- What specific problem does strict function type checking prevent?

### Advanced
- How can an inaccurate `.d.ts` file make an otherwise fully type-checked application unsafe?
- Walk through why a numeric enum's default reverse mapping is a real runtime behavior, not just a compile-time convenience.

### Follow-up Questions
- Does a type-only import ever appear in the compiled JavaScript output?
- Are string enums subject to the same reverse-mapping behavior as numeric enums?

### Code Prediction
```typescript
enum Status { Active, Inactive }
console.log(Status.Active);
console.log(Status[0]);
console.log(typeof Status);
```
What does each line actually log at runtime, and what does this reveal about an enum's real JavaScript output compared to a literal union?

## Practical Tasks

- Write a minimal declaration file for a small, untyped JavaScript helper, and verify the resulting type boundary behaves as expected.
- Replace an enum used only for a fixed set of API string values with an equivalent literal union, and confirm no runtime behavior actually depended on the enum's own object shape.
- Reproduce the strict-function-variance error with a callback narrower than its actual call site requires, and fix its parameter type.

## Readiness Criteria

Distinguish type-only from runtime module behavior precisely, recognize the trust-boundary risk of an inaccurate declaration file, and explain enum-versus-literal-union and function-variance trade-offs at a working-awareness level.

## References

- [TypeScript Handbook: Modules](https://www.typescriptlang.org/docs/handbook/2/modules.html)
- [TypeScript Handbook: Declaration Files](https://www.typescriptlang.org/docs/handbook/declaration-files/introduction.html)
- [TypeScript Handbook: Type Compatibility](https://www.typescriptlang.org/docs/handbook/type-compatibility.html)
