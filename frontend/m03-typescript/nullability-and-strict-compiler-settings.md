# Nullability and Strict Compiler Settings

## Definition

With `strictNullChecks` enabled, `string` and `string | undefined` are genuinely different types — a plain `string` can never secretly be `null`/`undefined` unless the type explicitly says so. Strict compiler settings as a whole turn categories of unsafe assumption (implicit `any`, unchecked array access, loose function parameter compatibility) into visible compile errors, instead of letting them surface later as runtime failures.

```typescript
function greet(name: string) { console.log(name.toUpperCase()); }
greet(undefined); // ERROR with strictNullChecks: Argument of type 'undefined' is not assignable to parameter of type 'string'
```

## Alternatives & Trade-offs

Disabling strict null checks avoids fixing a codebase's existing nullability errors immediately, but means `string` silently accepts `null`/`undefined` everywhere, reintroducing exactly the "is this actually there" uncertainty the whole point of TypeScript's nullability model exists to eliminate. Enabling strictness (ideally from the very start of a project) surfaces every place a value might genuinely be absent as a compile error, forcing a deliberate decision at each one — more upfront fixing, but a model that can actually be trusted afterward.

## How It Works

### An optional property introduces `undefined` into every use of that field

```typescript
type User = { name: string; nickname?: string };

function greet(user: User) {
  console.log(user.nickname.toUpperCase()); // ERROR: Object is possibly 'undefined'
}

function greetFixed(user: User) {
  console.log((user.nickname ?? user.name).toUpperCase()); // fixed: fall back when nickname is absent
}
```

The `?` on `nickname` means every read of it is typed as `string | undefined`, not just `string` — TypeScript forces a decision (a fallback, a guard, an early return) at the exact point the value is used, rather than letting a missing nickname crash later, deep inside some unrelated formatting code.

### `noImplicitAny` — catching an unannotated value before it silently becomes `any`

```typescript
// WITHOUT noImplicitAny: `value` implicitly becomes `any` with no annotation and no warning
function process(value) { return value.whatever.deeply.nested; } // compiles silently, unsafely

// WITH noImplicitAny enabled: this is now a compile ERROR, forcing an explicit type
function process(value: unknown) { /* must narrow before using `value` at all */ }
```

### `noUncheckedIndexedAccess` — array/object index access isn't automatically guaranteed present

```typescript
function getFirst(items: string[]): string {
  return items[0]; // WITHOUT noUncheckedIndexedAccess: type is `string` — but items COULD be empty at runtime!
}

// WITH noUncheckedIndexedAccess enabled:
function getFirstStrict(items: string[]): string | undefined {
  return items[0]; // now correctly typed as `string | undefined`, since an empty array has no index 0 at all
}
```

Without this flag, indexing into an array is typed as always succeeding — even though an empty array at runtime would make `items[0]` genuinely `undefined`, a mismatch between the type system's promise and reality that this flag closes.

### TypeScript's nullability is entirely static — it proves nothing about the actual runtime value

```typescript
interface ApiUser { name: string; } // claims name is ALWAYS present

async function fetchUser(): Promise<ApiUser> {
  const response = await fetch("/api/user");
  return response.json(); // response.json() returns `any` — TypeScript trusts this annotation with NO verification
}

const user = await fetchUser();
console.log(user.name.toUpperCase()); // if the actual API response omits `name`, this crashes at runtime —
                                        // the ApiUser type promised something the real data never guaranteed
```

Declaring `Promise<ApiUser>` doesn't make the server's actual response conform to that shape — it's a compile-time claim, not a runtime check, exactly like the type-assertion risk from [`unknown`, `any`, `never`, and type assertions](unknown-any-never-and-type-assertions.md). [API DTOs, validation errors, and runtime validation](api-dtos-and-runtime-validation.md) covers validating this boundary for real.

### C# nullable reference types are conceptually similar, but not the same rules

```
Both TypeScript's strictNullChecks and C#'s nullable reference types are primarily STATIC
ANALYSIS — neither changes runtime behavior on its own. But the specific rules differ
(TypeScript's `?` on a PROPERTY means "may be absent"; C#'s `?` on a TYPE means "may be null"
for an always-present property) — don't assume C# nullability intuitions transfer mechanically.
```

## Application

Enable strict null checks (and the broader `strict` flag bundle) from the start of a project, and fix the resulting model rather than weakening the setting to make errors disappear. Represent loading, missing, and present data deliberately using the actual type system, so UI code doesn't accumulate `!` assertions and unchecked indexing as a workaround.

## Common Mistakes

- Disabling strict null checks because existing code has errors, instead of fixing the underlying model.
- Confusing an optional property (`field?`) with a property that's always present but sometimes holds the value `undefined` — a genuinely different modeling choice.
- Assuming an array index access always returns a real element, when the array could be empty at runtime.
- Treating a TypeScript type as proof that a server response, DOM query, or user input is actually present and correctly shaped at runtime.

## Common Interview Questions

### Basic
- What does `strictNullChecks` change about how `string` and `undefined` relate?
- What does `noImplicitAny` prevent?

### Intermediate
- Why might accessing `array[0]` be unsafe even when the array's type is `string[]`?
- What does `noUncheckedIndexedAccess` specifically protect against?

### Advanced
- How would you migrate an existing, non-strict codebase to strict mode without simply hiding every new error behind assertions?
- Why doesn't declaring a function's return type as `Promise<User>` guarantee the actual API response matches that shape?

### Follow-up Questions
- Is TypeScript's nullability checking a runtime guarantee, or purely a compile-time analysis?
- Do C#'s nullable reference types and TypeScript's `strictNullChecks` follow identical rules?

### Code Prediction
```typescript
function getFirstChar(items: string[]): string {
  return items[0].charAt(0);
}
getFirstChar([]);
```
With `noUncheckedIndexedAccess` enabled, does this compile? What happens at runtime if it's called with an empty array regardless?

## Practical Tasks

- Enable strict null checks on a small sample project and fix each resulting compile error deliberately rather than suppressing it.
- Model an optional API field whose absence is meaningfully different from an explicit empty value, and handle both cases correctly.
- Enable `noUncheckedIndexedAccess` and fix a function that assumed an array access always succeeds.

## Readiness Criteria

Explain strict nullability precisely, choose relevant strict compiler flags deliberately, and represent absent values honestly instead of suppressing the compiler with assertions.

## References

- [TypeScript: strictNullChecks](https://www.typescriptlang.org/tsconfig/strictNullChecks.html)
- [TypeScript: TSConfig strict](https://www.typescriptlang.org/tsconfig/strict.html)
- [TypeScript: noUncheckedIndexedAccess](https://www.typescriptlang.org/tsconfig/noUncheckedIndexedAccess.html)
