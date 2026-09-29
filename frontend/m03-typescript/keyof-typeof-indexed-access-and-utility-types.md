# `keyof`, `typeof`, Indexed Access, and Utility Types

## Definition

These type operators *derive* new types from existing ones instead of duplicating them by hand. `keyof T` produces a union of `T`'s valid property keys. `typeof value` (in a type position) captures a value's static type. Indexed access (`T["key"]`) selects a specific property's type. Utility types (`Partial`, `Pick`, `Omit`, `Record`, and others) express common structural transformations without rewriting the shape from scratch.

```typescript
type User = { id: string; email: string; displayName: string };
type UserKey = keyof User;         // "id" | "email" | "displayName"
type EmailType = User["email"];      // string
```

## Alternatives & Trade-offs

Manually writing out a derived type (a second, hand-copied version of `User` with two fields removed) works but silently drifts out of sync the moment `User` itself changes — nothing forces the two to stay consistent. Deriving the type with `keyof`, indexed access, or a utility type costs a bit more unfamiliar syntax upfront, but guarantees the derived type tracks the source model automatically — if `User` changes, every type derived from it via these operators updates correctly without any manual edit.

## How It Works

### `keyof` — a union of an object type's valid keys

```typescript
type User = { id: string; email: string; displayName: string };
type UserKey = keyof User; // "id" | "email" | "displayName"

function getProperty<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key]; // safe: K is CONSTRAINED to be an actual key of T, so obj[key] can't fail
}

const user: User = { id: "1", email: "a@b.com", displayName: "Alice" };
getProperty(user, "email");    // fine — "email" IS a valid key of User
getProperty(user, "unknown");   // ERROR: Argument of type '"unknown"' is not assignable to parameter of type 'keyof User'
```

Constraining `K extends keyof T` (rather than accepting a plain `string` for the key) means the compiler catches an invalid property name at the call site — `getProperty(user, "unknown")` is rejected before the code ever runs, instead of returning `undefined` silently at runtime the way a plain-string-keyed lookup would.

### `typeof` in a type position — capturing a value's own static type

```typescript
const defaultSettings = { theme: "light", fontSize: 14 };
type Settings = typeof defaultSettings; // { theme: string; fontSize: number } — derived directly from the VALUE

function applySettings(settings: Settings) { }
```

This `typeof` is a completely different thing from JavaScript's runtime `typeof` operator (Module 2) — this one only exists in a type position, at compile time, and captures the *static type* of `defaultSettings`, not anything about its value at runtime.

### Indexed access — selecting one property's type, or a union of several

```typescript
type User = { id: string; email: string; age: number };
type EmailType = User["email"];        // string
type IdOrEmail = User["id" | "email"];   // string | string  ->  string
type AllValues = User[keyof User];         // string | string | number  ->  string | number
```

### Common utility types, and why `Partial<T>` alone is often too broad

```typescript
type User = { id: string; email: string; displayName: string; createdAt: string };

type UserUpdate = Partial<Omit<User, "id" | "createdAt">>; // every field optional, EXCEPT id/createdAt are gone entirely
type EditableUser = Pick<User, "id" | "email" | "displayName">; // only these three fields, all required
```

`Partial<User>` alone would make `id` and `createdAt` optional too — fields that should never be client-writable at all. Composing `Partial` with `Omit` first removes those fields entirely, so an "update" shape can never even attempt to include them, rather than merely making them optional and hoping nobody sets them.

### Other frequently-used utility types

```typescript
type ReadonlyUser = Readonly<User>;               // every property becomes readonly
type UserRecord = Record<string, User>;              // an object type indexed by arbitrary string keys, each holding a User
function fetchUser(id: string): Promise<User> { /* ... */ return null as any; }
type FetchUserReturn = ReturnType<typeof fetchUser>;   // Promise<User> — derived directly from the function itself
type FetchUserParams = Parameters<typeof fetchUser>;    // [id: string]
```

## Application

Derive related types from one stable source model using `keyof`, indexed access, and utility types, rather than hand-copying and maintaining a second, separately-drifting version. Compose utility types deliberately (`Partial<Omit<T, ...>>`) so the resulting shape actually matches the real boundary — not just "every field optional," which is often broader than what the operation should genuinely accept.

## Common Mistakes

- Using plain `string` for a property-key parameter where `keyof T` would catch an invalid key at compile time instead of failing silently at runtime.
- Confusing runtime `typeof` (a JavaScript operator returning a string like `"string"` or `"object"`) with type-position `typeof` (a TypeScript-only construct capturing a value's static type).
- Applying `Partial<T>` alone to a model when the actual domain boundary needs some fields to be entirely absent, not merely optional.
- Chaining `Partial`, `Omit`, and `Pick` so many times that the resulting type no longer clearly communicates what boundary it actually represents.

## Common Interview Questions

### Basic
- What does `keyof` produce, given an object type?
- What's the difference between runtime `typeof` and type-position `typeof`?

### Intermediate
- How would you type a function that safely reads a property from an object using a caller-provided key?
- When would you reach for `Pick` versus `Omit`?

### Advanced
- Why can a generic `getProperty<T, K extends keyof T>` catch an invalid key at compile time when a plain `getProperty(obj, key: string)` cannot?
- Why is `Partial<User>` often too broad for modeling an "update" operation's accepted fields?

### Follow-up Questions
- Does `keyof` include inherited properties from a type's base, or only its own?
- Can `ReturnType` and `Parameters` be applied to an arrow function the same way as a regular function?

### Code Prediction
```typescript
type Product = { sku: string; price: number; inStock: boolean };
type ProductKey = keyof Product;
function readField<T, K extends keyof T>(obj: T, key: K): T[K] { return obj[key]; }
const p: Product = { sku: "A1", price: 9.99, inStock: true };
const value = readField(p, "price");
```
What is the inferred type of `value`? What would happen if `readField(p, "weight")` were called instead?

## Practical Tasks

- Type a reusable `getProperty` helper that safely preserves the selected property's exact type, rejecting invalid keys at compile time.
- Derive an edit-form model from a read-only DTO using `Pick`, justifying every field included or omitted.
- Define an update DTO composing `Partial` and `Omit` so identifier and server-managed fields can never even be included, not just made optional.

## Readiness Criteria

Derive related types from a single source model using `keyof`, indexed access, and utility types, and compose utility types deliberately so the resulting shape matches the actual boundary rather than being broader than necessary.

## References

- [TypeScript Handbook: Keyof Type Operator](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html)
- [TypeScript Handbook: Indexed Access Types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html)
- [TypeScript Handbook: Utility Types](https://www.typescriptlang.org/docs/handbook/utility-types.html)
