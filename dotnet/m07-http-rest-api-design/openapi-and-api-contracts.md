# OpenAPI/Swagger and API Contracts

## Definition

OpenAPI (formerly "Swagger") is a specification format for describing an HTTP API's contract — its endpoints, request/response shapes, status codes, and authentication requirements — in a machine-readable document (usually JSON or YAML), independent of implementation language or framework.

```yaml
paths:
  /orders/{id}:
    get:
      summary: Get an order by ID
      parameters:
        - name: id
          in: path
          required: true
          schema: { type: integer }
      responses:
        '200':
          description: Order found
          content:
            application/json:
              schema: { $ref: '#/components/schemas/Order' }
        '404':
          description: Order not found
```

## Alternatives & Trade-offs

A machine-readable contract enables generated client SDKs, generated server stubs, automated documentation, and contract testing — all without hand-written, drift-prone documentation. The trade-off is upkeep: an OpenAPI document must stay in sync with the actual implementation, either by generating the document from code (common in ASP.NET Core) or generating the code from the document ("design-first" — writing the OpenAPI spec first, then generating server/client scaffolding from it). Hand-maintained documentation with no contract format at all is simpler initially but drifts from reality with no way to detect it automatically.

## How It Works

### Code-first vs. design-first

```
Code-first:   Write the API implementation; a tool inspects it (attributes, types, XML comments)
              and generates the OpenAPI document automatically.

Design-first: Write the OpenAPI document first, as the API's contract; generate server stubs
              and client SDKs from it, then implement the stub logic.
```

Code-first is more common for internal teams iterating quickly; design-first is more common when multiple teams (or external partners) need to agree on a contract before either side starts implementing.

### What a good contract captures beyond just endpoints

```yaml
components:
  schemas:
    Order:
      type: object
      required: [id, customerId, total]
      properties:
        id: { type: integer }
        customerId: { type: integer }
        total: { type: number, format: decimal }
        status: { type: string, enum: [Pending, Shipped, Cancelled] }
```

A complete contract documents required vs. optional fields, enums, and formats — not just "there's a `GET` here" — which is what makes generated clients and validation actually useful rather than just a list of URLs.

### Contract as a testing tool

A generated OpenAPI document can be diffed between versions to detect breaking changes (a required field becoming optional is safe; a field being removed or renamed is breaking) before they reach consumers — a lightweight form of contract testing.

## Application

Maintain an OpenAPI document for any API with external consumers, multiple teams, or generated client needs. Use code-first generation for fast-moving internal APIs; consider design-first when the contract itself needs sign-off from multiple parties before implementation begins.

## Common Mistakes

- Treating the generated OpenAPI document as automatically correct without checking that it actually reflects intended behavior (e.g., a response type that's technically accurate but omits a field that's always present in practice).
- Not versioning or diffing the contract, so breaking changes reach consumers without warning.
- Hand-writing documentation instead of a real OpenAPI document, losing the ability to generate clients or validate the contract automatically.
- Under-specifying schemas (everything typed as a loose object with no required fields or enums), producing a contract that's technically present but not actually useful for generated clients or validation.

## Common Interview Questions

### Basic
- What is OpenAPI, and what problem does it solve?
- What's the difference between "code-first" and "design-first" API contracts?

### Intermediate
- How can an OpenAPI document be used to detect breaking API changes before release?
- Why might a design-first approach be chosen over code-first for a multi-team API?

### Advanced
- How would you set up automated contract-diffing in a CI pipeline to catch breaking changes?
- What makes an OpenAPI schema "useful" for generated clients versus just technically present?

### Follow-up Questions
- Does having an OpenAPI document guarantee the implementation matches it?
- Can OpenAPI describe authentication requirements, not just endpoints and schemas?

### Code Prediction
A required field is removed from a response schema in a new API version, but the OpenAPI document isn't updated to reflect it. What breaks first — clients using a generated SDK, clients parsing raw JSON, or neither — and why does this illustrate the risk of manually-maintained documentation versus a generated, verified contract?

## Practical Tasks

- Write an OpenAPI document (or generate one from an existing API) for a small set of endpoints, including request/response schemas and status codes.
- Compare a code-first-generated OpenAPI document against the actual API behavior and identify any drift.
- Design a process for detecting breaking contract changes between two versions of an OpenAPI document.

## Readiness Criteria

Explain what OpenAPI provides beyond documentation, distinguish code-first from design-first workflows, and reason about how a contract can be used to catch breaking changes before they reach consumers.

## References

### Other

- [OpenAPI Specification](https://spec.openapis.org/oas/latest.html)
- [Swagger official documentation](https://swagger.io/docs/)
