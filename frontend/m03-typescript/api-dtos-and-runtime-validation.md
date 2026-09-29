# API DTOs, Validation Errors, and Runtime Validation

## Definition

An API DTO is a compile-time model of the data a client expects to send or receive. Runtime validation is a genuinely separate concern: actually checking whether real JSON — which TypeScript has never seen and cannot verify — actually matches that model. A TypeScript annotation alone performs no runtime check whatsoever; this is the single most consequential idea in this entire module.

```typescript
interface User { id: string; email: string; }

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  return response.json(); // response.json() returns `any` — TypeScript trusts this claim with ZERO verification
}
```

## Alternatives & Trade-offs

Asserting an API response's shape (`response.json() as User`, or trusting a declared `Promise<User>` return type) costs nothing to write and compiles instantly, but provides no actual protection if the real response is malformed, has an unexpected shape, or omits a field entirely — the failure just moves later, to whatever code eventually reads the missing field. Validating the response with a schema library at the boundary costs a small amount of runtime work and a schema definition to maintain, but means a malformed response produces one clear, catchable failure at the exact point of entry, instead of an unpredictable crash somewhere downstream.

## How It Works

### Why `response.json() as User` isn't validation at all

```typescript
interface User { id: string; email: string; }

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const user = (await response.json()) as User; // COMPILES — but performs literally zero verification
  return user;
}

const user = await fetchUser("42");
console.log(user.email.toUpperCase()); // if the real API response omitted `email` entirely, this CRASHES here,
                                          // far away from fetchUser, with no earlier warning of any kind
```

The assertion tells TypeScript "trust me, this is a `User`" — it does not inspect the actual parsed object at all. If the real response is `{ id: "42" }` with no `email` field, this code compiles perfectly and then crashes at the very first place `email` is actually used, which could be an entirely different file.

### Validating at the boundary with a schema library

```typescript
import { z } from "zod";

const UserSchema = z.object({ id: z.string(), email: z.string().email() });
type User = z.infer<typeof UserSchema>; // the TypeScript type is DERIVED from the schema — one source of truth

async function fetchUser(id: string): Promise<User> {
  const response = await fetch(`/api/users/${id}`);
  const json = await response.json();
  return UserSchema.parse(json); // THROWS immediately, with a clear error, if json doesn't actually match the schema
}
```

`UserSchema.parse()` genuinely inspects the parsed JSON at runtime and either returns a value the compiler can trust *because it was actually checked*, or throws a clear, immediate error naming exactly what didn't match — instead of silently accepting anything and deferring the failure to wherever the bad data eventually gets used.

### Mapping schema failures into a stable, client-facing error model

```typescript
function parseUser(json: unknown): { success: true; data: User } | { success: false; error: string } {
  const result = UserSchema.safeParse(json); // safeParse: never throws, returns a discriminated result instead
  if (result.success) {
    return { success: true, data: result.data };
  }
  return { success: false, error: "Received malformed user data from the server" };
}
```

Wrapping the schema library's own result shape in the application's own stable error model means the rest of the codebase never needs to know or care which specific validation library is being used underneath — a later library swap wouldn't ripple out into every component that handles a validation failure.

### Keeping the transport DTO separate from a UI-specific view model

```typescript
type UserDto = { id: string; email: string; createdAt: string }; // exactly what the API sends

type UserViewModel = { id: string; email: string; memberSince: string }; // formatted for DISPLAY

function toViewModel(dto: UserDto): UserViewModel {
  return { id: dto.id, email: dto.email, memberSince: new Date(dto.createdAt).toLocaleDateString() };
}
```

## Application

Treat every value crossing an external trust boundary — a `fetch` response, `localStorage`, a URL parameter — as `unknown` until it's actually validated, using a schema library (Zod, Valibot, or an equivalent) rather than a type assertion. Map a validation library's own failure shape into a stable, application-specific error model instead of scattering library-specific details throughout components. Keep a transport DTO separate from a view model whenever the UI's display needs (formatting, derived labels) diverge from the raw API shape.

## Common Mistakes

- Writing `const user = response.json() as User` and treating that assertion as if it were actual validation.
- Trusting an API response's shape purely because "the backend is typed," when JSON on the wire is untyped, can change, and can be malformed regardless of what the backend's own C# types say.
- Reusing a transport DTO directly as a display model, scattering formatting logic and assumptions throughout components instead of centralizing them in one mapping function.
- Returning an unstructured, generic string for every server failure, discarding field-level validation detail the API actually provided.

## Common Interview Questions

### Basic
- Why doesn't a TypeScript interface validate incoming JSON?
- What is the actual purpose of a DTO, versus what a runtime validator does?

### Intermediate
- Where in a frontend application should runtime validation happen?
- How would you map a schema library's own failure result into a stable, application-facing error model?

### Advanced
- How would you handle a backend response that's syntactically valid JSON but violates the expected contract (a missing or renamed field)?
- When does a transport DTO need to become a separate view model, and what problem does keeping them merged cause over time?

### Follow-up Questions
- Does `response.json()` throw if the response body isn't valid JSON at all, or only when it doesn't match the expected shape?
- Should every single API response be validated, even ones from a trusted, first-party backend?

### Code Prediction
```typescript
interface User { id: string; email: string; }
async function fetchUser(): Promise<User> {
  const res = await fetch("/api/user");
  return res.json() as User;
}
const user = await fetchUser();
console.log(user.email.length);
```
If the actual API response is `{ "id": "42" }` with no `email` field at all, at what point does this code actually fail — at the `as User` assertion, or somewhere else? Where, specifically?

## Practical Tasks

- Define a transport DTO, a separate view model, and the explicit mapping function that converts between them.
- Add a schema-based runtime validation boundary for an API response currently only using a type assertion.
- Map a schema library's validation failure into a stable, application-specific error type, without leaking library-specific error details into UI components.

## Readiness Criteria

Explain precisely why a static DTO type performs no runtime validation, design a genuine schema-validation boundary for untrusted API responses, and keep transport and display models appropriately separate.

## References

- [TypeScript Handbook: Type Assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions)
- [Zod documentation](https://zod.dev/)
- [ASP.NET Core: Handle Errors in ASP.NET Core Web APIs](https://learn.microsoft.com/aspnet/core/web-api/handle-errors)
