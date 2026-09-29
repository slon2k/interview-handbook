# Type Inference, Annotations, and Compiler Feedback

## Definition

TypeScript can **infer** a type from an initializer, a return expression, or surrounding context, without ever being told explicitly. An **annotation** states an intended type directly. The `satisfies` operator sits between the two: it checks a value against a type without widening the value's own inferred type the way an annotation would. The compiler compares whichever model it has against how the value is actually used, and reports a mismatch before the code ever runs.

```typescript
const status = "ready";        // inferred as the LITERAL type "ready", not the wider `string`
let mutableStatus = "ready";     // inferred as the WIDER type `string`, because reassignment is allowed
function greet(name: string) { } // parameter annotated explicitly — callers provide this value, so it can't be inferred
```

## Alternatives & Trade-offs

Relying on inference everywhere is less to type and stays accurate automatically as an implementation changes, but function parameters can't be inferred at all (there's no value yet to infer from) and exported/public functions benefit from an explicit contract that doesn't shift accidentally as internal code changes. Annotating everything is maximally explicit but adds noise to code whose type is already perfectly obvious from its initializer — the practical convention is to let inference handle local, obvious values and annotate boundaries (parameters, exported return types, ambiguous values) deliberately.

## How It Works

### `const` vs. `let` — the same literal value, two different inferred types

```typescript
const status = "ready";     // type: "ready" (a specific literal) — this binding can never be reassigned to anything else
let mutableStatus = "ready";  // type: string (widened) — because mutableStatus COULD later be reassigned to any string
```

TypeScript widens a `let`'s inferred type to the general `string`/`number`/`boolean` because reassignment is legal — inferring the narrower literal type `"ready"` would incorrectly reject `mutableStatus = "pending"` later on, even though that's a perfectly valid reassignment.

### Contextual typing — inferring a callback's parameter from where it's used

```typescript
const numbers = [1, 2, 3];
numbers.map(n => n * 2); // `n` is inferred as `number`, with NO annotation needed — TypeScript already
                           // knows numbers is number[], and infers the callback parameter from that context
```

TypeScript doesn't need `n: number` written explicitly here because it already knows the type of `numbers` and works backward from that to infer what kind of value `map`'s callback will receive — this is why callback parameters usually don't need annotations inside `.map()`, `.filter()`, and similar calls.

### `satisfies` — checking a value against a type without widening it

```typescript
type Point = { x: number; y: number };

const annotated: Point = { x: 1, y: 2 };       // an ANNOTATION — widens the type to exactly Point, nothing more specific
const inferred = { x: 1, y: 2 };                 // pure INFERENCE — type is { x: number; y: number }, no check against Point at all
const checked = { x: 1, y: 2 } satisfies Point;    // CHECKED against Point, but keeps its own precise inferred type
```

```typescript
const palette = {
  red: [255, 0, 0],
  green: [0, 255, 0]
} satisfies Record<string, [number, number, number]>;

palette.red;         // type is [number, number, number] — NARROWED to this specific key, unlike an annotation would allow
palette.blue;          // ERROR: Property 'blue' does not exist — caught immediately, because "blue" was never a key at all
```

An explicit `: Record<string, [number, number, number]>` annotation on `palette` would widen every access to the *general* `Record` shape — `palette.red` would just be `[number, number, number]` with no memory of which specific keys actually exist, and `palette.blue` would incorrectly be allowed to compile as valid. `satisfies` verifies the value conforms to the constraint while preserving its own narrower, literal inferred type — exactly the missing piece for the `as const` configuration-object pattern from [unions, intersections, literal types, and optional properties](unions-intersections-literals-and-optional-properties.md), letting a config object be checked against a shape *and* keep its precise, specific keys and value types simultaneously.

### An annotation checks a value — it never converts or validates it

```typescript
function processId(id: string) { }
processId(42 as unknown as string); // COMPILES — the annotation only ever checked the STATIC type;
                                       // at runtime, `id` really is the number 42, not a string at all
```

An annotation is a compile-time promise checked against how a value is used in source code — it has no runtime effect whatsoever. This becomes critical at any real trust boundary (parsed JSON, a form field, an API response), covered in [API DTOs, validation errors, and runtime validation](api-dtos-and-runtime-validation.md) — annotating a parsed value as `User` doesn't make the actual runtime object conform to that shape.

### `const` doesn't mean deeply immutable

```typescript
const user = { name: "Alice" };
user.name = "Bob";      // COMPILES FINE — const only prevents reassigning the VARIABLE `user` itself
user = { name: "Carol" }; // does NOT compile — this is what const actually prevents
```

### Explicit return types protect a public function's contract from silently widening

```typescript
function getStatus(): "idle" | "loading" | "success" { // explicit — locks the contract
  if (Math.random() > 0.5) return "loading";
  return "idle"; // if someone LATER adds `return "done"` by mistake, THIS annotation catches it immediately
}
```

Without the explicit return type, adding a new, unintended return value somewhere in the function body would silently widen the inferred return type instead of producing a compiler error at the point of the actual mistake.

## Application

Let inference handle local, obvious values — don't annotate `const message = "hello"`. Annotate function parameters (required, since there's no value to infer from), and annotate exported/public function return types deliberately, so the function's contract stays locked even as its implementation changes internally. Reach for `satisfies` specifically when a value needs to be checked against a shape *and* keep its own precise, narrower inferred type — a plain annotation would widen it away.

## Common Mistakes

- Annotating every local variable redundantly, adding visual noise to code whose type is already obvious from its initializer.
- Assuming `const` makes an object's contents immutable, when it only prevents reassigning the variable binding itself.
- Treating a type annotation on parsed JSON or another external value as if it performed real runtime validation.
- Omitting explicit return types from exported functions, letting the contract silently widen as the implementation changes later.
- Using a plain annotation on a configuration object when `satisfies` was needed, silently widening away the specific keys and losing precise per-key type checking.

## Common Interview Questions

### Basic
- What is type inference, and how does it differ from an annotation?
- Why do function parameters usually need explicit type annotations?
- What does the `satisfies` operator check, and how is that different from an ordinary annotation?

### Intermediate
- Why does `let status = "ready"` infer a wider type than `const status = "ready"`?
- What is contextual typing, and how does it let array callback parameters go unannotated?

### Advanced
- How does an explicit return type on an exported function protect its contract from silently widening later?
- Walk through why `const user = { name: "Alice" }; user.name = "Bob";` compiles, while `user = { name: "Carol" }` does not.
- Walk through why `satisfies Record<string, [number, number, number]>` preserves specific per-key types while an equivalent `: Record<...>` annotation would not.

### Follow-up Questions
- Does a type annotation perform any conversion or validation of the actual runtime value?
- Can a `let` variable ever have a literal type inferred, the way `const` does?

### Code Prediction
```typescript
const a = "ready";
let b = "ready";
const c: string[] = [];
```
What is the inferred type of each of `a`, `b`, and `c`, and why does `a` differ from `b` despite an identical initializer?

## Practical Tasks

- Add explicit boundary annotations (parameters, exported return types) to a small module while leaving obvious local values inferred.
- Reproduce the `const`-doesn't-mean-immutable mistake and fix the actual intended restriction using `readonly` or `Object.freeze`.
- Remove an explicit return type from a function, introduce a new unintended return branch, and observe how the mistake surfaces differently with and without the annotation.
- Convert a configuration object typed with a plain annotation into one using `satisfies`, and verify that per-key precise types are now preserved.

## Readiness Criteria

Distinguish inference, annotation, and `satisfies` precisely, explain literal-type widening for `let` versus `const`, and use each of the three deliberately at boundaries rather than defaulting to annotation everywhere.

## References

- [TypeScript Handbook: Everyday Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)
- [TypeScript Handbook: Type Inference](https://www.typescriptlang.org/docs/handbook/type-inference.html)
- [TypeScript: The satisfies Operator](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-4-9.html#the-satisfies-operator)
