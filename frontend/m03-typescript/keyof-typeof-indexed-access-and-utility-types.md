# `keyof`, `typeof`, Indexed Access, and Utility Types

## Definition

TypeScript type operators derive types from existing values and types. `keyof` produces a type of valid property keys, `typeof` captures the static type of a value, and indexed access selects a property or element type. Utility types transform common shapes without repeating them.

## How It Works

- `keyof User` produces the union of keys that can be used to index `User`.
- `typeof runtimeValue` refers to the value's static type in a type position; it does not inspect the runtime value.
- `User["id"]` or `User[keyof User]` selects property types.
- `Partial`, `Required`, `Readonly`, `Pick`, `Omit`, `Record`, `ReturnType`, and `Parameters` express common transformations.
- Generic key parameters such as `K extends keyof T` preserve the relationship between an object and one of its valid keys.

## Application

Derive types from stable source models instead of manually duplicating them. Use these operators for form field maps, table columns, selectors, and adapters where a key and its value must remain correlated.

### Utility types should reflect a real boundary

```typescript
type User = {
  id: string;
  email: string;
  displayName: string;
  createdAt: string;
};

type UserUpdate = Partial<Omit<User, "id" | "createdAt">>;
type EditableUser = Pick<User, "id" | "email" | "displayName">;
```

`Partial<User>` is often too broad because it makes immutable identifiers and server-managed fields writable. Derive an update shape from the fields the operation actually accepts, and introduce a named type when the boundary is important to the domain.

## Common Mistakes

- Confusing `typeof value` at runtime with `typeof Value` in a type position.
- Using `string` for a key when `keyof T` can prevent invalid property access.
- Applying `Partial` to a model when the domain actually requires distinct create and update types.
- Assuming utility types validate or transform runtime objects.
- Chaining `Partial` and `Omit` until the resulting type no longer communicates the operation it represents.

## Common Interview Questions

### Foundation

- What does `keyof` produce?
- What is the difference between runtime `typeof` and type-position `typeof`?

### Intermediate

- How would you type a function that reads a valid key from an object?
- When would you use `Pick` or `Omit`?

### Advanced and Follow-up

- Why can a generic `get<T, K extends keyof T>` preserve more safety than `get(object, key: string)`?

### Code Prediction

Predict the result type of `type Value = User[keyof User]` and explain why an arbitrary string cannot safely index `User`.

## Practical Tasks

- Type a reusable `getProperty` helper while preserving the selected property's type.
- Derive an edit form model from a read-only DTO and justify every omitted field.
- Define an update DTO that permits only client-editable fields and explain why `Partial<ReadDto>` would be too broad.

## Readiness Criteria

You can derive related types from a source model and preserve valid key/value relationships without pretending the runtime object changed.

## References

- [TypeScript Handbook: Keyof type operator](https://www.typescriptlang.org/docs/handbook/2/keyof-types.html)
- [TypeScript Handbook: Indexed access types](https://www.typescriptlang.org/docs/handbook/2/indexed-access-types.html)
- [TypeScript Handbook: Utility types](https://www.typescriptlang.org/docs/handbook/utility-types.html)