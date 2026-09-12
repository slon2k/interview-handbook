# OpenAPI/Swagger Clients vs. Handwritten Clients

## Definition

Frontend projects often consume APIs using either generated clients from an OpenAPI/Swagger document or handwritten wrappers around `fetch`. Each approach has trade-offs in speed, correctness, and customisation.

## How It Works

- OpenAPI generation produces TypeScript clients or request builders from an API contract.
- Generated clients can speed up adoption, provide consistency, and reduce manual typing errors.
- Handwritten clients allow custom retry, logging, auth injection, and domain-specific behavior that generated code may not capture cleanly.
- Generated code can be stale if the contract is out of date; handwritten code may drift without a central schema.
- The better choice depends on team maturity, API stability, code ownership, and the need for custom business logic.

## Application

For stable, widely used APIs, a generated client can reduce hand-written boilerplate. For custom flows, auth policies, or domain-specific error handling, a small handwritten boundary may be easier to reason about and evolve.

## Common Mistakes

- Assuming generated code is automatically correct without checking that the OpenAPI document is accurate.
- Writing a custom client that duplicates logic already available in a generated API client.
- Ignoring the fact that generated clients can hide backend quirks and require version management.
- Letting the UI call raw `fetch` everywhere instead of centralizing the API boundary.

## Common Interview Questions

### Foundation

- What is OpenAPI?
- Why might a generated client be useful?

### Intermediate

- When would you prefer a handwritten client over a generated one?
- How does the API boundary reduce duplication across the app?

### Advanced and Follow-up

- What are the trade-offs between keeping a generated client “as is” and adapting it with wrappers?
- How would you validate that the generated client matches the real backend contract?

### Code Prediction

A generated client returns error objects shaped according to the OpenAPI schema, whereas a handwritten client may translate backend errors into a domain-specific result. Explain when you would prefer each model.

## Practical Tasks

- Compare a raw `fetch` wrapper against a generated OpenAPI client for a common CRUD scenario.
- Decide whether custom retry or auth logic should live in the client wrapper or in each consuming component.

## Readiness Criteria

You can explain the use cases for generated and handwritten API clients and justify an integration style based on API stability, custom behavior, and team constraints.

## References

- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Swagger UI](https://swagger.io/tools/swagger-ui/)
- [Microsoft Learn: OpenAPI in ASP.NET Core](https://learn.microsoft.com/aspnet/core/tutorials/web-api-help-pages-using-openapi?view=aspnetcore-9.0)
