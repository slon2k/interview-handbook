# `unknown`, `any`, `never`, and Type Assertions

## Definition

`unknown` represents a value whose type genuinely isn't known yet and must be narrowed before use. `any` disables type checking for that value entirely. `never` represents a value that can't logically occur — an impossible case, or the return type of a function that never completes normally. A type assertion (`value as T`) tells the compiler to trust the author's claim about a value's type; it performs no runtime check or conversion at all.

```typescript
let safe: unknown = fetchData();     // must be narrowed before use
let unsafe: any = fetchData();          // usable immediately, with NO checking at all — the risk
```

## Alternatives & Trade-offs

`any` requires no narrowing at all and lets code compile immediately, which is exactly the problem — it's "contagious," silently disabling checking for everything it touches, so one `any` can spread type-unsafety through otherwise well-typed code with no warning at any of the points it flows through. `unknown` requires the extra step of narrowing before it can be used at all, which is more upfront friction but means the type system genuinely protects every subsequent use — the compiler simply refuses to let you do anything with an `unknown` value until you've proven what it actually is.

## How It Works

### `any` is contagious — it spreads unsafety silently

```typescript
function parseConfig(json: string) {
  const config: any = JSON.parse(json); // JSON.parse's return type is actually `any` by default!
  return config.settings.theme.toUpperCase(); // COMPILES — no error anywhere, even if `settings` doesn't exist at all
}
```

Every property access on `config` here compiles without complaint, no matter how deep or how wrong — `any` doesn't just skip checking the value itself, it disables checking on everything derived from it too, all the way down the chain.

### `unknown` requires proof before any operation is allowed

```typescript
function parseConfig(json: string) {
  const config: unknown = JSON.parse(json);
  return config.settings; // ERROR: Object is of type 'unknown' — must narrow FIRST

  if (typeof config === "object" && config !== null && "settings" in config) {
    return config.settings; // now allowed — narrowed based on an ACTUAL runtime check
  }
}
```

`unknown` forces exactly the discipline `any` skips — you cannot access any property or call any method until you've genuinely proven, via a real check, what the value actually is.

### `never` — for functions that don't return, and for exhaustiveness checks

```typescript
function fail(message: string): never {
  throw new Error(message); // never actually RETURNS a value — execution always exits via throw
}

function infiniteLoop(): never {
  while (true) { } // also never returns — a different way of never completing normally
}
```

This is the same `never`-typed `assertNever` helper from the narrowing topic — `never` there specifically represents "every possible case has already been handled, so nothing should ever reach this point."

### A type assertion changes only what the compiler believes — never the runtime value

```typescript
const value: unknown = "hello";
const asNumber = value as number; // COMPILES — but the actual runtime value is STILL the string "hello"
console.log(asNumber.toFixed(2)); // runtime CRASH: toFixed is not a function — the assertion lied, and nothing caught it
```

`as number` never converts anything — it's purely a compile-time instruction telling TypeScript "treat this as a `number` from here on," with zero effect on what the value actually is at runtime. If the assertion is wrong, the compiler has no way to know, and the failure surfaces later, at the point of actual use, as an ordinary runtime error.

### Non-null assertions (`!`) — the same risk, applied to nullability specifically

```typescript
function getUser(id: string): User | undefined { /* ... */ }

const user = getUser("42")!; // asserts "this is DEFINITELY not undefined" — with no actual check
console.log(user.name); // if getUser really did return undefined, this crashes at runtime with no earlier warning
```

## Application

Use `unknown` by default for anything genuinely untrusted — parsed JSON, a caught exception, a third-party plugin value — and narrow it with a real check before use. Reserve `any` for situations where it's truly unavoidable (an untyped third-party library with no available types), and contain it immediately at that boundary rather than letting it spread. Treat every type assertion and non-null assertion as a claim that needs a nearby justification, not a routine way to silence an error.

## Common Mistakes

- Replacing a type error with `as SomeType` reflexively, instead of narrowing or fixing the actual underlying model mismatch.
- Using `any` for a difficult generic or library boundary without containing it, letting the unsafety spread through everything that touches it.
- Assuming `as number` performs an actual conversion, when a string value asserted as a number is still a string at runtime and will fail at the point of actual numeric use.
- Using `!` (non-null assertion) on a value whose presence genuinely depends on user input or an API response, rather than on an invariant that's actually guaranteed elsewhere.

## Common Interview Questions

### Basic
- What's the difference between `unknown` and `any`?
- What does a type assertion actually do at runtime?

### Intermediate
- Why is `any` described as "contagious"? What does that mean concretely?
- When is `never` useful in ordinary application code, beyond just theoretical completeness?

### Advanced
- Walk through why `JSON.parse(json).settings.theme` compiles with no errors at all, and what type `JSON.parse` actually returns by default.
- How would you contain an unavoidable third-party `any` so it doesn't spread unsafety through the rest of a typed application?

### Follow-up Questions
- Does using `as` ever throw at the point the assertion is written, if the claim turns out to be wrong?
- Is `unknown` ever usable without narrowing it first?

### Code Prediction
```typescript
const value: unknown = "42";
const asNum = value as number;
console.log(typeof asNum);
console.log(asNum + 1);
```
What does `typeof asNum` actually report at runtime, despite the assertion claiming it's a `number`? What does `asNum + 1` evaluate to?

## Practical Tasks

- Replace an `any`-typed API response with `unknown`, then add a narrowing check before any property is accessed.
- Find and remove a non-null assertion from a UI model, replacing it with an explicit check for the absent case instead.
- Contain an unavoidable third-party `any` at a single boundary function, so its unsafety doesn't spread into the rest of a typed module.

## Readiness Criteria

Choose between `unknown`, `any`, `never`, narrowing, and assertions based on the actual trust boundary involved, and explain precisely why `any` is more dangerous than `unknown` despite both technically bypassing full type safety.

## References

- [TypeScript Handbook: Unknown Type](https://www.typescriptlang.org/docs/handbook/release-notes/typescript-3-0.html#the-unknown-type)
- [TypeScript Handbook: Exhaustiveness Checking](https://www.typescriptlang.org/docs/handbook/2/narrowing.html#exhaustiveness-checking)
- [TypeScript Handbook: Type Assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions)
