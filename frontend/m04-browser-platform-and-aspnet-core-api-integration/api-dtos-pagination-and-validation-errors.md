# API DTOs, Pagination, and Validation Errors

## Definition

Pagination conventions, HTTP error responses, and field-level validation errors are API contract choices the browser client must handle deliberately. TypeScript DTO modeling and runtime validation are covered in [Module 3](../m03-typescript/api-dtos-and-runtime-validation.md); this lesson focuses on how an ASP.NET Core HTTP API communicates those models.

## How It Works

- A paginated endpoint needs an agreed body envelope or documented headers for metadata such as `page`, `pageSize`, `totalCount`, and next/previous links or flags.
- ASP.NET Core commonly returns RFC 7807-style `ProblemDetails` for failures. A validation response can include an `errors` dictionary mapping field names to message arrays.
- The client should distinguish an expected validation failure from authentication, authorization, conflict, and unexpected server failures.
- The frontend should map transport results into a UI-friendly model when display fields or naming differ.

```typescript
type ProblemDetails = {
  type?: string;
  title?: string;
  status?: number;
  detail?: string;
  instance?: string;
};

type ValidationProblemDetails = ProblemDetails & {
  errors: Record<string, string[]>;
};
```

## Application

Agree the contract with the API owner before building the client. For a list, decide where pagination metadata lives. For a failed write, preserve field-level validation messages so the form can show actionable feedback. Map `401` and `403` through the auth flow, `409` through a deliberate conflict path, and unknown `5xx` responses through a recoverable generic failure.

## Common Mistakes

- Flattening nested errors into a single string and losing the information the user needs to fix the form.
- Assuming pagination metadata is in the body when the API publishes it in headers, or vice versa.
- Treating every non-success status as a retryable network failure.
- Reusing DTO types directly for display fields that need conversion or formatting.

## Common Interview Questions

### Foundation

- Why is pagination metadata important for a client-side list?
- What is the role of `ProblemDetails` in an ASP.NET Core API?

### Intermediate

- How would you map `ValidationProblemDetails` onto field-level form errors?
- When should pagination metadata live in an envelope instead of HTTP headers?

### Advanced and Follow-up

- How should the client distinguish `400` validation, `401`, `403`, `409`, and `500` responses?

### Code Prediction

Given a `400` response with an `errors` dictionary and a `409` response with generic `ProblemDetails`, explain how each should change the UI differently.

## Practical Tasks

- Compare a body-envelope and header-based pagination contract, then document which one a client uses.
- Map an ASP.NET Core validation response to field errors without losing multiple messages for a field.

## Readiness Criteria

You can explain pagination conventions, map `ProblemDetails` and validation failures into deliberate UI behavior, and distinguish status-specific failures from transport failures.

## References

- [ASP.NET Core: Handle errors in ASP.NET Core web APIs](https://learn.microsoft.com/aspnet/core/web-api/handle-errors)
- [MDN: Fetch API](https://developer.mozilla.org/en-US/docs/Web/API/Fetch_API)
- [TypeScript Handbook: Type assertions](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html#type-assertions)
