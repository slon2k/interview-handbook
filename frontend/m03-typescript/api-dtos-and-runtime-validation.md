# API DTOs, Validation Errors, and Runtime Validation

## Definition

An API DTO is a compile-time model of the data a client expects to send or receive. Runtime validation checks whether actual JSON or user-controlled data matches that model. TypeScript annotations alone perform no runtime check.

This lesson owns type-level transport modeling and the runtime trust boundary. [Module 4's API-contract lesson](../m04-browser-platform-and-aspnet-core-api-integration/api-dtos-pagination-and-validation-errors.md) owns HTTP-specific choices such as status handling, pagination conventions, and how an ASP.NET Core API communicates validation failures.

## How It Works

- Model transport records, nested data, nullable fields, and stable client error types explicitly.
- Treat `response.json()` and other external values as untrusted until parsed or validated at a boundary.
- A schema-validation library such as Zod, Valibot, or an equivalent can parse unknown data and return a validated value or a structured failure.
- Keep transport DTOs separate from view models when the UI needs derived labels, defaults, or a different shape.
- Map schema-library failures into a stable client error model rather than scattering parser details through components.

## Application

Validate at the API client boundary, then let the rest of the application consume the validated type. This localizes uncertainty and gives the UI a deliberate response for malformed data instead of a later property-access failure.

## Common Mistakes

- Writing `const user = response.json() as User` and calling it validation.
- Trusting an API because the backend is “typed.” JSON on the wire can still be incomplete, changed, or malicious.
- Reusing a DTO as a display model and scattering formatting assumptions through components.
- Returning an unstructured string for every server failure, losing field-level error information.

## Common Interview Questions

### Foundation

- Why does a TypeScript interface not validate JSON?
- What is the purpose of a DTO?

### Intermediate

- Where should runtime validation happen in a frontend application?
- How would you model paginated data and field-level validation errors?

### Advanced and Follow-up

- How would you handle a backend response that is syntactically valid JSON but violates the expected contract?
- When should a transport DTO become a separate view model?

### Code Prediction

Given `const user = JSON.parse(text) as User`, explain what happens when the JSON omits `id` and why the assertion does not throw.

## Practical Tasks

- Define a transport DTO, a separate view model, and the mapper that converts between them.
- Add a runtime schema boundary for an unknown JSON response and map schema failure to a user-facing error state.

## Readiness Criteria

You can distinguish a static DTO from runtime validation, design a stable API-client boundary, and preserve useful parse failures without leaking schema-library details into the UI.

## References

- [TypeScript Handbook: Type assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions)
- [ASP.NET Core: Handle errors in ASP.NET Core web APIs](https://learn.microsoft.com/aspnet/core/web-api/handle-errors)
- [Zod documentation](https://zod.dev/)