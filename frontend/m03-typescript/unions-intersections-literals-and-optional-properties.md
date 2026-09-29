# Unions, Intersections, Literal Types, and Optional Properties

## Definition

A **union** (`A | B`) describes a value that's one of several alternatives. An **intersection** (`A & B`) requires a value to satisfy every combined requirement at once. A **literal type** restricts a value to one exact string, number, or boolean, rather than the wider primitive. An **optional property** (`field?`) may be entirely absent, which is a distinct concept from a property that's always present but sometimes explicitly `undefined`.

```typescript
type Status = "idle" | "loading" | "success" | "error"; // a literal union — far more precise than plain `string`
type Draggable = { onDragStart: () => void };
type Resizable = { onResize: () => void };
type Widget = Draggable & Resizable; // must have BOTH onDragStart and onResize
```

## Alternatives & Trade-offs

Modeling a set of mutually exclusive states as one object with several optional fields (`{ data?: T; error?: string; loading?: boolean }`) is quick to sketch out, but permits combinations that can never actually happen in practice (`loading: true` alongside a populated `error`) and forces every consumer to defensively guard against those impossible states anyway. A [discriminated union](narrowing-type-guards-and-discriminated-unions.md) or even a simple literal union for a status field costs a bit more upfront modeling, but makes invalid combinations genuinely unrepresentable, which the compiler then enforces on your behalf.

## How It Works

### A union value is usable only in ways valid for every alternative — until narrowed

```typescript
function printId(id: string | number) {
  console.log(id.toUpperCase()); // ERROR: Property 'toUpperCase' does not exist on type 'number'
                                    // — toUpperCase is only valid for the STRING alternative, not both
}
```

Before [narrowing](narrowing-type-guards-and-discriminated-unions.md), TypeScript only allows operations that are safe for *every* member of the union — accessing `.toUpperCase()` would be unsafe if `id` actually turned out to be the `number` alternative at runtime.

### Intersections combine requirements — useful for composing capabilities

```typescript
type Timestamped = { createdAt: string };
type Named = { name: string };
type Record = Timestamped & Named; // must have BOTH createdAt AND name

const entry: Record = { createdAt: "2026-01-01", name: "Example" }; // both required fields present
```

### Deriving a literal union from real configuration data with `as const`

```typescript
const fieldNames = ["email", "password"] as const; // WITHOUT as const, this would just infer string[]
type FieldName = typeof fieldNames[number]; // "email" | "password" — derived directly from the array's actual values

const routes = { home: "/", settings: "/settings" } as const;
type RouteName = keyof typeof routes;      // "home" | "settings"
type RoutePath = typeof routes[RouteName];    // "/" | "/settings"
```

Without `as const`, `fieldNames` would widen to plain `string[]`, and `FieldName` would just become `string` — losing all the precision `as const` preserves. Deriving the type *from* the actual runtime array means the type can never silently drift out of sync with the values the application genuinely uses. `as const` alone doesn't *check* the object against a shape, though — it only locks in the literal types already present. [Type inference, annotations, and compiler feedback](type-inference-annotations-and-compiler-feedback.md) covers `satisfies`, which combines both: checking a configuration object against a required shape while still preserving its precise, narrower keys and values.

### Optional property vs. explicit `undefined` — not automatically the same thing

```typescript
type Draft = { note?: string };

const a: Draft = {};                // note is ENTIRELY ABSENT
const b: Draft = { note: undefined }; // note is PRESENT, with the value undefined

"note" in a; // false
"note" in b; // true — a real, if subtle, distinction that matters for form libraries checking "was this field touched"
```

With `exactOptionalPropertyTypes` enabled, TypeScript treats these as genuinely different states — assigning explicit `undefined` to an optional property becomes its own distinct, checkable case rather than being silently equivalent to leaving it out.

## Application

Use a literal union to precisely model a finite set of valid string/number values instead of the much wider `string`/`number`. Use `as const` to derive a literal union directly from real configuration data, keeping the type and the data it describes from ever drifting apart. Model mutually exclusive states as a proper union rather than a bag of optional fields that permits impossible combinations.

## Common Mistakes

- Modeling a set of mutually exclusive states as one object with many optional fields, permitting combinations that can never actually occur.
- Using plain `string` where a literal union would catch an invalid value at compile time.
- Omitting `as const`, causing a fixed set of values to widen to `string[]` or `string` and lose their precision.
- Assuming an optional property is always present with the value `undefined`, when it may be entirely absent instead — a real distinction some form and validation libraries depend on.

## Common Interview Questions

### Basic
- What's the difference between a union type and an intersection type?
- Why are literal types more precise than `string` or `number`?

### Intermediate
- How would you model a component that supports exactly one of several named modes?
- What's the difference between an absent optional property and one explicitly set to `undefined`?

### Advanced
- Why is a discriminated union (or even a plain literal-union status field) safer than a type with several independently optional, status-specific fields?
- How does `as const` change the inferred type of an array literal, and why does that matter for deriving a field-name union from real data?

### Follow-up Questions
- Can a value typed as `A | B` safely call a method that only exists on `A`, before any narrowing has occurred?
- Does `exactOptionalPropertyTypes` change anything about ordinary JavaScript's own `undefined` handling at runtime?

### Code Prediction
```typescript
const values = ["small", "medium", "large"];
const valuesConst = ["small", "medium", "large"] as const;
type SizeA = typeof values[number];
type SizeB = typeof valuesConst[number];
```
What is the inferred type of `SizeA` versus `SizeB`, and why does `as const` make the difference?

## Practical Tasks

- Replace a model with several independently optional fields with a proper union representing its actual valid states.
- Define a literal union for a finite UI status field and verify an invalid string value is rejected at compile time.
- Derive a field-name union from an `as const` array, then use it to type a validation-error map keyed by that union.

## Readiness Criteria

Choose between union, intersection, literal, and optional types based on a domain's actual valid states, and use `as const` deliberately to keep a derived type in sync with real configuration data.

## References

- [TypeScript Handbook: Unions and Intersection Types](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#union-types)
- [TypeScript Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
