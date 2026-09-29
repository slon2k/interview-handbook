# Structural Typing, Interfaces, Type Aliases, and Readonly Data

## Definition

TypeScript is **structurally** typed: a value is assignable wherever it has the required shape, regardless of what it was originally named or declared as. **Interfaces** and **type aliases** both describe shapes — interfaces can be reopened and merged; aliases can additionally describe unions, tuples, and primitives directly. `readonly` is a compile-time restriction on writes, not a runtime freeze.

```typescript
interface Point { x: number; y: number; }
type PointAlias = { x: number; y: number };

function logPoint(p: Point) { console.log(p.x, p.y); }
logPoint({ x: 1, y: 2, label: "origin" }); // works via a variable, structurally identical, extra property allowed
```

## Alternatives & Trade-offs

Nominal typing (as in C# or Java, where a type must explicitly declare that it implements an interface) prevents two unrelated types with identical shapes from being accidentally interchangeable — TypeScript's structural typing doesn't offer that protection automatically, since compatibility is judged purely by shape. This trade-off makes structural typing very convenient for adapters, test doubles, and working with data from various sources (anything with the right shape just works), at the cost of allowing two conceptually different values — a `UserId` and an `OrderId`, both just `string` underneath — to be passed interchangeably with no compiler complaint at all.

## How It Works

### Structural compatibility — shape is everything

```typescript
interface HasName { name: string; }

function greet(entity: HasName) { console.log(`Hello, ${entity.name}`); }

const dog = { name: "Rex", breed: "Labrador" };
greet(dog); // COMPILES — dog has a `name: string`, which is all HasName actually requires;
             // `dog` never declared "implements HasName" anywhere, and doesn't need to
```

### Excess-property checks — stricter for a fresh object literal than for a stored variable

```typescript
function createUser(config: { name: string }) { }

createUser({ name: "Alice", age: 30 }); // ERROR: 'age' does not exist in type '{ name: string }'
                                          // — fresh object LITERALS get an extra, stricter check

const config = { name: "Alice", age: 30 };
createUser(config); // COMPILES FINE — same shape, but passed via a variable, which skips the stricter literal check
```

This asymmetry is deliberate: a fresh object literal passed directly is almost certainly a typo or a misunderstanding of the expected shape (why would you write `age` if the function never uses it?), so TypeScript flags it — but a variable might legitimately carry extra properties for other purposes elsewhere, so that stricter check is relaxed once it's stored first.

### `readonly` prevents a compile-time write — nothing about runtime

```typescript
interface Config { readonly apiUrl: string; }

function updateConfig(config: Config) {
  config.apiUrl = "https://new-url.com"; // ERROR: Cannot assign to 'apiUrl' because it is a read-only property
}
```

```javascript
// But at RUNTIME, after TypeScript compiles away, nothing stops this from a plain JS caller:
const config = { apiUrl: "https://old-url.com" };
config.apiUrl = "https://changed-anyway"; // works completely fine in the compiled JavaScript output
```

`readonly` is purely a TypeScript-level restriction the compiler enforces against *TypeScript* code that tries to write through it — it has no runtime enforcement at all once compiled, and provides no protection against a plain JavaScript caller (or a bypass like an object cast) mutating the value anyway.

### Branding two structurally-identical string types apart

```typescript
type UserId = string & { readonly __brand: "UserId" };
type OrderId = string & { readonly __brand: "OrderId" };

function getUser(id: UserId) { }

declare const orderId: OrderId;
getUser(orderId); // ERROR — even though both are `string` underneath, the brand makes them incompatible
```

A "branded type" adds a phantom property that only exists at the type level (never actually present at runtime) specifically to defeat structural compatibility for two values that happen to share the same primitive shape but represent conceptually different things — exactly the `UserId`/`OrderId` mixup structural typing alone would otherwise allow.

## Application

Use interfaces or type aliases to name meaningful, reused boundaries — not every incidental inline object shape. Reach for branded types (or a discriminated union) specifically when two values share a primitive shape but must never be interchangeable, since plain structural typing won't catch that mistake on its own.

## Common Mistakes

- Assuming TypeScript nominally distinguishes two types with identical fields, when structural typing makes them freely interchangeable.
- Expecting `readonly` to prevent a runtime mutation, when it's purely a compile-time check with zero effect once compiled to JavaScript.
- Being surprised that excess-property checks apply to a fresh object literal but not to the same shape passed via a stored variable.
- Passing a `UserId` where an `OrderId` was expected (or vice versa) with no compiler complaint, because both are plain `string` underneath with no branding to distinguish them.

## Common Interview Questions

### Basic
- What does structural typing mean, and how does it differ from nominal typing?
- What does `readonly` actually guarantee, and what doesn't it guarantee?

### Intermediate
- Why is a fresh object literal rejected by excess-property checks while an identical variable is accepted?
- When would you choose an interface over a type alias, or vice versa?

### Advanced
- How would you prevent a `UserId` string from being passed where an `OrderId` string is required, given that both are structurally identical `string`s?
- Walk through why `readonly` provides no actual runtime protection once TypeScript compiles to JavaScript.

### Follow-up Questions
- Can two completely unrelated interfaces be assignable to each other if their shapes happen to match?
- Does an interface support describing a union type the way a type alias does?

### Code Prediction
```typescript
function process(config: { id: number }) { }
const value = { id: 1, extra: "data" };
process(value);
process({ id: 1, extra: "data" });
```
Which of these two calls compiles, and which doesn't? Why does storing the object in a variable first change the outcome?

## Practical Tasks

- Model a DTO and a separate view model as distinct named shapes without duplicating unrelated fields between them.
- Design a branded-type representation for two domain IDs that are both plain strings at runtime, and verify they're no longer interchangeable.
- Reproduce the excess-property-check asymmetry and explain to a colleague why one form compiles and the other doesn't.

## Readiness Criteria

Explain structural compatibility and its risks precisely, correctly predict excess-property-check behavior for literals versus variables, and use branded types where structural typing alone would allow an unsafe mixup.

## References

- [TypeScript Handbook: Object Types](https://www.typescriptlang.org/docs/handbook/2/objects.html)
- [TypeScript Handbook: Type Compatibility](https://www.typescriptlang.org/docs/handbook/type-compatibility.html)
