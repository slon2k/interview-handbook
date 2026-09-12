# API DTOs, Pagination, and Validation Errors

## Definition

A client-side DTO is a typed representation of what the browser expects from an HTTP API. Pagination metadata and field-level validation errors are part of the same contract boundary and should be modeled explicitly rather than guessed from ad hoc objects.

## How It Works

- A DTO describes the shape of a payload returned from the server, including field names, optional properties, and nested records.
- Pagination often includes metadata such as `page`, `pageSize`, `totalCount`, and next/previous links or flags.
- ASP.NET Core validation errors may be returned as a structured error payload or a `ProblemDetails`-style response that includes fields or a dictionary of validation messages.
- The frontend should map transport DTOs into a UI-friendly model, especially when the display layer requires derived fields or different naming.
- The browser is not a validation layer; it still needs runtime checks at the API boundary.

## Application

Model server contracts deliberately. When a backend response contains validation details or pagination metadata, make that structure visible in the client types so the UI can render loading, empty, error, and retry states consistently.

## Common Mistakes

- Treating a response object as though it is already validated because a TypeScript interface exists.
- Flattening nested errors into a single string and losing the information the user needs to fix the form.
- Assuming all paginated responses have identical metadata or field names.
- Reusing DTO types directly for display fields that need conversion or formatting.

## Common Interview Questions

### Foundation

- What is a DTO?
- Why is pagination metadata important for a client-side list?

### Intermediate

- How would you model a paginated ASP.NET Core response and validation errors?
- When should the client map a DTO to a view model?

### Advanced and Follow-up

- What happens if the backend returns a syntactically valid JSON payload that violates the expected contract?
- Why is runtime validation still necessary even when the UI is typed?

### Code Prediction

Given a JSON payload with missing required fields or a validation error dictionary, explain why the TypeScript type alone does not make the payload safe and how the client should handle it.

## Practical Tasks

- Model a paginated result with per-page metadata and a list of records.
- Type a validation error contract for a form submission and map it to a user-friendly error state.

## Readiness Criteria

You can explain DTO modeling, pagination contracts, validation error handling, and the difference between compile-time typing and runtime validation at the API boundary.

## References

- [ASP.NET Core: Handle errors in ASP.NET Core web APIs](https://learn.microsoft.com/aspnet/core/web-api/handle-errors)
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [TypeScript Handbook: Type assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions)
