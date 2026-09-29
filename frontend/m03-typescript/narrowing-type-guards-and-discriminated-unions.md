# Narrowing, Type Guards, Discriminated Unions, and Exhaustive Checks

## Definition

Narrowing refines a broad type (usually a union) down to a more specific one, using a runtime check the compiler actually understands — `typeof`, `in`, equality comparisons, `instanceof`, or a user-defined type guard. A discriminated union uses a shared literal field (the "discriminant") to identify which alternative a value currently is, letting a `switch` on that field narrow the rest of the object's shape per branch.

```typescript
function printId(id: string | number) {
  if (typeof id === "string") {
    console.log(id.toUpperCase()); // narrowed to `string` inside this branch — safe
  } else {
    console.log(id.toFixed(2)); // narrowed to `number` here — safe
  }
}
```

## Alternatives & Trade-offs

A type assertion (`value as User`) tells the compiler what to believe with zero runtime verification — fast to write, but provides no actual safety if the assumption is wrong. Narrowing via a real runtime check costs a few more lines, but the resulting type is genuinely backed by evidence the check just performed — the compiler isn't taking your word for it, it's watching the actual control flow.

## How It Works

### Control-flow analysis tracks narrowing through branches and early returns

```typescript
function getLength(value: string | string[]): number {
  if (Array.isArray(value)) {
    return value.length; // narrowed to string[] — .length is an array property here
  }
  return value.length; // narrowed to string — .length is a string property here instead
}
```

TypeScript tracks exactly which branch of an `if` you're in and narrows accordingly — the *same* property access (`.length`) is valid in both branches, but for two entirely different reasons, resolved correctly by the compiler in each case.

### Discriminated unions — a shared literal field drives exhaustive narrowing

```typescript
type Result =
  | { status: "loading" }
  | { status: "success"; data: string[] }
  | { status: "error"; message: string };

function render(result: Result) {
  switch (result.status) {
    case "loading": return "Loading...";
    case "success": return result.data.join(", "); // `data` is available ONLY in this branch
    case "error": return result.message;              // `message` is available ONLY in this branch
  }
}
```

Accessing `result.data` outside the `"success"` case would be a compile error — the discriminant field (`status`) is what lets TypeScript narrow the *rest* of the object's shape per branch, making each case's specific fields available only where they're actually guaranteed to exist.

### `assertNever` — turning a missed case into a compile error

```typescript
function assertNever(value: never): never {
  throw new Error(`Unhandled case: ${JSON.stringify(value)}`);
}

function render(result: Result) {
  switch (result.status) {
    case "loading": return "Loading...";
    case "success": return result.data.join(", ");
    case "error": return result.message;
    default: return assertNever(result); // if a new Result variant is added later and NOT handled above,
                                           // `result` here is no longer `never` -> compile ERROR right here
  }
}
```

If someone later adds a fourth `Result` variant (say, `{ status: "empty" }`) but forgets to add its `case`, the `default` branch's `result` is no longer narrowed all the way down to `never` — it's now the unhandled variant, and passing it to `assertNever` produces an immediate compile error pointing exactly at the gap, rather than a silent runtime bug reaching production.

### A user-defined type guard is only as safe as its own implementation

```typescript
function isUser(value: unknown): value is User { // the `value is User` return type is a PROMISE to the compiler
  return typeof value === "object" && value !== null; // WRONG: doesn't actually check for User's specific fields!
}

const maybeUser: unknown = { foo: "bar" };
if (isUser(maybeUser)) {
  console.log(maybeUser.email); // TypeScript trusts the guard and lets this compile — but `email` doesn't actually exist!
}
```

The compiler takes a `value is User` type guard's word for it — if the guard's actual runtime check doesn't genuinely verify every field the type claims, this is functionally identical to an unsafe type assertion, just wrapped in a function that looks more trustworthy.

## Application

Model async state, API results, and UI modes as discriminated unions with a real literal discriminant field, then narrow via `switch`. Add an `assertNever` default case so introducing a new union member without handling it everywhere becomes a compile error, not a silent gap. Write user-defined type guards that genuinely verify every field their claimed type requires.

## Common Mistakes

- Narrowing on a field that isn't actually reliable at runtime (an optional field that might be missing on real data, not just in the type).
- Writing a user-defined type guard that returns `true` without actually checking the shape it claims to guarantee.
- Adding a `default` branch that silently ignores or swallows a new discriminated-union member instead of using `assertNever` to catch it at compile time.
- Confusing a type assertion (`as User`, no verification at all) with genuine narrowing (a real runtime check the compiler can trust).

## Common Interview Questions

### Basic
- What is type narrowing?
- How does a discriminated union work, and what role does the discriminant field play?

### Intermediate
- How would you write a type guard for unknown, untrusted API data?
- How can a `switch` over a discriminated union be made exhaustive, and what does `assertNever` actually catch?

### Advanced
- Walk through what makes a user-defined type guard genuinely unsafe when its implementation doesn't verify every field it claims.
- Why does adding a new variant to a discriminated union, without updating every relevant `switch`, get caught at compile time when `assertNever` is used?

### Follow-up Questions
- Does narrowing change the underlying runtime value in any way, or only what the compiler believes about it?
- Can a discriminated union's discriminant be something other than a string literal?

### Code Prediction
```typescript
type Shape = { kind: "circle"; radius: number } | { kind: "square"; side: number };
function area(shape: Shape) {
  if (shape.kind === "circle") {
    return Math.PI * shape.side ** 2; // deliberate mistake
  }
  return shape.side ** 2;
}
```
What compile error occurs here, and how does the discriminant `kind` reveal the mistake?

## Practical Tasks

- Model loading, success, empty, and error states as a discriminated union with a literal `status` field.
- Add an `assertNever` default case to an existing switch, then introduce a new union variant and observe the compiler catch the unhandled case.
- Write a genuine type guard for unknown API data that actually verifies every field of the claimed type, not just that the value is a non-null object.

## Readiness Criteria

Narrow safely from a union or `unknown` using real runtime checks, write type guards whose implementation actually backs their claimed type, and use exhaustive checks to protect an evolving discriminated union from silently missed cases.

## References

- [TypeScript Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)
- [TypeScript Handbook: Discriminated Unions](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#discriminated-unions)
