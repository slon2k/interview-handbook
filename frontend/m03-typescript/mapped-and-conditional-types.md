# Mapped and Conditional Types

## Definition

A mapped type iterates over a set of property keys (usually `keyof` some source type) to build a new object type, optionally transforming each property's value type or modifiers along the way. A conditional type selects between two types based on an assignability test (`T extends U ? Yes : No`), and can capture part of a matched type with `infer`.

```typescript
type Flags<T> = { [K in keyof T]: boolean }; // same KEYS as T, every VALUE replaced with boolean

type Model = { id: string; name: string };
type ModelFlags = Flags<Model>; // { id: boolean; name: boolean }
```

## Alternatives & Trade-offs

Writing a transformed type by hand (manually copying `Model`'s keys into a new `ModelFlags` type) works for a small, stable model, but silently drifts out of sync the moment `Model` gains or loses a field. A mapped type derives the transformation directly from the source, staying correct automatically — at the cost of being less immediately readable to someone unfamiliar with the `[K in keyof T]` syntax, and genuinely harder to maintain past a certain point of complexity, which is exactly why "simplify to a named type" is a real, worthwhile escape hatch.

## How It Works

### Mapped types can add or remove modifiers, not just transform values

```typescript
type Model = { id: string; name?: string };

type AllRequired<T> = { [K in keyof T]-?: T[K] };  // the `-?` REMOVES optionality from every property
type AllOptional<T> = { [K in keyof T]?: T[K] };    // makes every property optional
type AllReadonly<T> = { readonly [K in keyof T]: T[K] }; // adds readonly to every property

type Strict = AllRequired<Model>; // { id: string; name: string } — name is no longer optional
```

`Partial<T>` and `Required<T>` (from [`keyof`, `typeof`, indexed access, and utility types](keyof-typeof-indexed-access-and-utility-types.md)) are themselves just built-in mapped types using exactly this `+?`/`-?` modifier syntax — understanding the mechanism here explains what those utility types are actually doing underneath.

### Conditional types — choosing a type based on an assignability check

```typescript
type IsString<T> = T extends string ? "yes" : "no";

type A = IsString<string>;  // "yes"
type B = IsString<number>;    // "no"
```

### `infer` — capturing a related type inside a conditional pattern

```typescript
type ElementType<T> = T extends (infer Item)[] ? Item : never;

type A = ElementType<string[]>; // string — `infer Item` captured the array's element type
type B = ElementType<number>;      // never — number isn't an array shape at all, so the condition fails

type Awaited2<T> = T extends Promise<infer Value> ? Value : T;
type C = Awaited2<Promise<string>>; // string — extracted from inside the Promise
```

`infer` lets a conditional type "reach into" a matched structure and pull out a piece of it — this is precisely the mechanism TypeScript's own built-in `Awaited<T>` utility type uses to extract a Promise's resolved value type.

### Conditional types distribute over a union — a subtle, easy-to-miss behavior

```typescript
type ToArray<T> = T extends any ? T[] : never;

type Result = ToArray<string | number>; // string[] | number[] — NOT (string | number)[] !
```

When a conditional type's checked type parameter is a "naked" union (not wrapped in anything), TypeScript applies the conditional to *each member of the union separately* and unions the results back together — producing `string[] | number[]`, which is a meaningfully different (and usually more useful) type than `(string | number)[]` would have been.

## Application

Use a mapped type for a systematic transformation applied uniformly across a model's fields — field-level metadata (dirty/touched flags), a patch/update shape, or a readonly view. Use a conditional type when a reusable type's shape genuinely needs to depend on its input — extracting an array's element type, or a Promise's resolved value. Replace either with a simpler, named type the moment it becomes hard to explain to someone reading it.

## Common Mistakes

- Reaching for mapped/conditional type machinery where a plain, explicit named type would be clearer and equally correct.
- Forgetting that a conditional type distributes over a "naked" union type parameter, producing a union of transformed types rather than one transformed union.
- Assuming a mapped type changes anything about the actual runtime object, when it only describes a new compile-time shape.
- Writing deeply recursive or overly generic type-level logic that slows the compiler and becomes difficult for anyone else to read or modify.

## Common Interview Questions

### Basic
- What does a mapped type do, given a source object type?
- What is a conditional type, in its basic form?

### Intermediate
- How would you make every property of a model optional, or every property readonly, using a mapped type?
- What does `infer` capture inside a conditional type?

### Advanced
- Walk through why `ToArray<string | number>` produces `string[] | number[]` rather than `(string | number)[]`.
- When should a type-level abstraction (a mapped or conditional type) be replaced with a simpler, explicitly named type?

### Follow-up Questions
- Are `Partial<T>` and `Required<T>` themselves implemented using mapped types?
- Does a mapped type require its source to be an object type, or can it operate on other shapes too?

### Code Prediction
```typescript
type UnwrapArray<T> = T extends (infer Item)[] ? Item : T;
type A = UnwrapArray<number[]>;
type B = UnwrapArray<string>;
```
What is the resulting type of `A` and `B` respectively, and why does `B` not change at all?

## Practical Tasks

- Create a mapped type producing field-level "dirty" or "touched" boolean metadata from an existing form model.
- Use a conditional type with `infer` to extract an array's element type or a Promise's resolved value type.
- Reproduce the union-distribution behavior of a naked conditional type and explain the resulting type to a colleague.

## Readiness Criteria

Read and write modest mapped and conditional types, correctly predict union-distribution behavior, and recognize when type-level complexity should be simplified into a plain named type instead.

## References

- [TypeScript Handbook: Mapped Types](https://www.typescriptlang.org/docs/handbook/2/mapped-types.html)
- [TypeScript Handbook: Conditional Types](https://www.typescriptlang.org/docs/handbook/2/conditional-types.html)
