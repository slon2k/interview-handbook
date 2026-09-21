# Unions, Intersections, Literal Types, and Optional Properties

## Definition

Union types describe a value that may be one of several alternatives. Intersection types combine requirements. Literal types restrict values to exact strings, numbers, or booleans. Optional properties describe a property that may be absent, which is not always identical to a property present with value `undefined`.

## How It Works

- A value of type `A | B` can be used only in ways valid for both alternatives until it is narrowed.
- `A & B` requires the value to satisfy both types and is useful for composing object capabilities.
- Literal unions model finite choices such as `"idle" | "loading" | "success" | "error"` more precisely than `string`.
- `as const` prevents a stable value from widening and lets a literal union be derived from a configuration object or tuple.
- Optional properties use `?`; with `exactOptionalPropertyTypes`, assigning explicit `undefined` can be treated differently from omitting the property.
- `readonly` tuples and literal inference can preserve a narrow contract, while broad annotations can widen it.

## Application

Use unions to represent alternatives honestly instead of making every field optional. Use intersections for composition, not as a replacement for a domain model that should be a discriminated union.

### Deriving a literal union from values

```typescript
const fieldNames = ["email", "password"] as const;
type FieldName = typeof fieldNames[number];
// "email" | "password"

const routes = {
  home: "/",
  settings: "/settings"
} as const;

type RouteName = keyof typeof routes;
type RoutePath = typeof routes[RouteName];
```

Use this pattern when the runtime configuration is the source of truth. It prevents a separately maintained string union from drifting away from the values the application actually uses.

## Common Mistakes

- Treating `A | B` as if every property from both types is available.
- Modeling mutually exclusive states as one object with many optional fields.
- Using `string` where a finite literal union would catch invalid values.
- Omitting `as const` and accidentally widening a fixed set of values to `string[]` or `string`.
- Assuming an optional property always exists with value `undefined`.

## Common Interview Questions

### Foundation

- What is the difference between a union and an intersection?
- Why are literal types useful?

### Intermediate

- How would you model a component that supports one of several modes?
- What is the difference between an absent optional property and an explicit `undefined`?

### Advanced and Follow-up

- Why is a discriminated union safer than a type with several optional status-specific fields?

### Code Prediction

Predict which properties are safely accessible on a value typed as `{ id: number } | { name: string }` before and after a type guard.

## Practical Tasks

- Replace a model with five optional fields by a union of valid states.
- Define a literal union for a finite UI status and reject an invalid status at compile time.
- Derive a field-name union from an `as const` tuple, then use it to type a validation-error map.

## Readiness Criteria

You can choose between union, intersection, literal, optional, and readonly types based on the valid states of the domain.

## References

- [TypeScript Handbook: Unions and intersections](https://www.typescriptlang.org/docs/handbook/unions-and-intersections.html)
- [TypeScript Handbook: Narrowing](https://www.typescriptlang.org/docs/handbook/2/narrowing.html)