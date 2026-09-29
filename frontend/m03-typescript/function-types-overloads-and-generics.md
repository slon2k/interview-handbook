# Function Types, Overloads, and Generics

## Definition

A function type describes a callable's parameters and return value. Overloads let one function expose several distinct supported call signatures, sharing a single implementation. Generics preserve a real relationship between a function's input and output types — the alternative to `any`, which would just discard that relationship entirely.

```typescript
function first<T>(items: T[]): T | undefined { return items[0]; }
const a = first([1, 2, 3]);     // inferred as number | undefined
const b = first(["x", "y"]);      // inferred as string | undefined — the SAME function, different preserved type
```

## Alternatives & Trade-offs

Typing a reusable helper with `any` compiles immediately for every input, but throws away all information about what actually went in and out — the caller gets no type-checking benefit at all from the function's return value. A generic costs a bit more upfront type-parameter syntax, but preserves the exact relationship between what's passed in and what comes out, which is precisely what `any` gives up for the sake of that initial convenience.

## How It Works

### A generic's type parameter is inferred from the actual argument — no manual annotation needed

```typescript
function wrapInArray<T>(value: T): T[] { return [value]; }

const numbers = wrapInArray(5);       // T inferred as number -> numbers: number[]
const strings = wrapInArray("hello");   // T inferred as string -> strings: string[]
```

Callers don't need to write `wrapInArray<number>(5)` explicitly — TypeScript infers `T` directly from the argument's actual type, the same contextual-inference idea from the type-inference topic, now applied to a function's own type parameter.

### Constraining a generic with `extends` — requiring just enough, not more

```typescript
function getId<T extends { id: string }>(item: T): string {
  return item.id; // safe: the constraint GUARANTEES every T has an `id: string`
}

getId({ id: "42", name: "Widget" }); // fine — has id AND more, which is allowed
getId({ name: "Widget" });             // ERROR: Property 'id' is missing
```

The constraint requires only what the function actually needs (`id`) — a caller's object can have any number of additional properties beyond that, exactly matching the structural-typing rules from earlier in this module.

### Overloads — several supported call shapes, one implementation

```typescript
function parseValue(value: string): number;             // overload signature 1
function parseValue(value: string, radix: number): number; // overload signature 2
function parseValue(value: string, radix?: number): number { // the actual implementation — handles BOTH cases
  return radix !== undefined ? parseInt(value, radix) : parseInt(value);
}

parseValue("42");      // matches overload 1
parseValue("2A", 16);    // matches overload 2
```

Callers only ever see the two declared overload signatures — the implementation signature (`radix?: number`) is not itself visible to callers and must simply be broad enough to genuinely handle every case the overloads promise.

### Choosing between a union, overloads, and a generic

```typescript
// A generic — output type depends directly on input type
function identity<T>(value: T): T { return value; }

// A union — the alternatives have genuinely DIFFERENT behavior, not just different types
function formatValue(value: string | number): string {
  return typeof value === "string" ? value.trim() : value.toFixed(2);
}

// Overloads — several distinct, named call shapes for the SAME conceptual operation
function createElement(tag: "input"): HTMLInputElement;
function createElement(tag: "div"): HTMLDivElement;
function createElement(tag: string): HTMLElement { return document.createElement(tag); }
```

A generic is right when the function's own behavior doesn't change based on the type — it just needs to *preserve* whatever type came in. A union is right when the alternatives genuinely need different handling (as `formatValue` does). Overloads are right when several distinct, named call shapes exist for one operation, especially when the relationship between argument and return type can't be expressed as cleanly with a single generic signature.

## Application

Use a generic whenever a function's output type depends directly on its input type, or when the same operation needs to work across several types without losing that relationship. Keep generic APIs small and easily inferable — a generic so complex that callers must annotate it manually is often less useful than a clear, concrete function. Reach for overloads specifically when a few distinct, well-named call shapes are clearer than one broad, harder-to-read signature.

## Common Mistakes

- Using `any` for a reusable collection or identity helper where a generic would preserve the actual input/output type relationship.
- Writing overload signatures whose shared implementation doesn't actually handle every case the overloads promise to support.
- Adding an unconstrained generic that provides no real type relationship, when a plain concrete type would have been just as safe and clearer.
- Choosing overloads for a case where the alternatives genuinely have different *behavior*, when a union with a type check inside the function would be more direct.

## Common Interview Questions

### Basic
- What is a generic type parameter, and what problem does it solve compared to `any`?
- When would you reach for a function overload?

### Intermediate
- How does a constraint like `T extends { id: string }` help a generic function, without over-restricting it?
- How would you type a callback that receives a row of data and returns a formatted label?

### Advanced
- Walk through why `first<T>(items: T[])` correctly infers `number | undefined` for a `number[]` argument and `string | undefined` for a `string[]` argument, using the same function.
- When is a union preferable to overloads or a generic for modeling a function's behavior?

### Follow-up Questions
- Are a function's overload signatures visible to a caller, or only the implementation signature?
- Can a generic function's type parameter always be inferred, or does it sometimes need to be specified explicitly?

### Code Prediction
```typescript
function wrap<T>(value: T): { value: T } { return { value }; }
const a = wrap(5);
const b = wrap("hello");
```
What is the inferred type of `a` and `b` respectively, and why does the same function produce two different, precise result types?

## Practical Tasks

- Replace an `any`-typed collection helper (like a "first element" or "last element" function) with a properly inferred generic.
- Add overloads to a small parsing function supporting two distinct, meaningfully different call shapes, backed by one safe implementation.
- Add a constraint to an unconstrained generic function so it can safely access a specific property every caller's argument must have.

## Readiness Criteria

Describe function contracts precisely, choose between overloads, unions, and generics based on the actual relationship being modeled, and preserve input/output type relationships instead of discarding them with `any`.

## References

- [TypeScript Handbook: More on Functions](https://www.typescriptlang.org/docs/handbook/2/functions.html)
- [TypeScript Handbook: Generics](https://www.typescriptlang.org/docs/handbook/2/generics.html)
