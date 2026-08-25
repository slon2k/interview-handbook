# Validation

## Definition

Validation checks that incoming request data satisfies required rules (required fields, ranges, formats, cross-field consistency) before it reaches business logic. ASP.NET Core supports **data annotations** (attribute-based, simple rules) and integrates with third-party libraries like **FluentValidation** for more complex or reusable validation logic.

```csharp
public class CreateOrderRequest
{
    [Required]
    public int CustomerId { get; set; }

    [Range(1, 100)]
    public int Quantity { get; set; }
}
```

## Alternatives & Trade-offs

Data annotations are quick to apply directly on a DTO and integrate automatically with `[ApiController]`'s model-state validation, but become awkward for rules spanning multiple fields or requiring external lookups (e.g., "does this customer ID actually exist"). FluentValidation (or a hand-written validator) separates validation logic from the DTO entirely, scales better for complex rules and reuse across similar DTOs, at the cost of an extra abstraction and explicit wiring.

## How It Works

### Data annotations with `[ApiController]`

```csharp
public class CreateOrderRequest
{
    [Required, StringLength(200)]
    public string CustomerName { get; set; } = "";

    [Range(1, int.MaxValue)]
    public int Quantity { get; set; }
}

[ApiController]
public class OrdersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateOrderRequest request)
    {
        // ModelState is already validated by the time this line runs;
        // [ApiController] automatically returns 400 with validation details if invalid
        return Ok();
    }
}
```

### Cross-field validation with `IValidatableObject`

```csharp
public class DateRangeRequest : IValidatableObject
{
    public DateTime Start { get; set; }
    public DateTime End { get; set; }

    public IEnumerable<ValidationResult> Validate(ValidationContext context)
    {
        if (End < Start)
            yield return new ValidationResult("End must not precede Start", new[] { nameof(End) });
    }
}
```

### FluentValidation for complex, reusable rules

```csharp
public class CreateOrderValidator : AbstractValidator<CreateOrderRequest>
{
    public CreateOrderValidator()
    {
        RuleFor(x => x.CustomerId).GreaterThan(0);
        RuleFor(x => x.Quantity).InclusiveBetween(1, 100);
        RuleFor(x => x).Must(HaveConsistentDates).WithMessage("End must not precede Start");
    }

    private bool HaveConsistentDates(CreateOrderRequest request) => true; // example
}
```

FluentValidation validators are typically registered in DI and invoked explicitly (or via an integration package) rather than relying purely on attribute-driven `ModelState`.

### Validation happens at the boundary, not instead of domain invariants

```csharp
// Request validation: is this a well-formed request?
[Range(1, 100)] public int Quantity { get; set; }

// Domain invariant: does this violate a business rule, regardless of how the request arrived?
public void SetQuantity(int quantity)
{
    if (quantity <= 0) throw new ArgumentOutOfRangeException(nameof(quantity)); // guard clause, defense in depth
}
```

Request-level validation catches malformed input early and cheaply; domain-level guard clauses (see `guard-clauses.md` in Module 4) protect invariants regardless of entry point — both matter, and one doesn't replace the other.

## Application

Use data annotations for simple, self-contained rules directly on request DTOs. Use FluentValidation (or a similar library) once rules become complex, need to be reused across DTOs, or require external context (database lookups, conditional rules based on other fields). Keep domain-level invariant checks in place regardless of request-level validation, since not all code paths that construct a domain object go through HTTP request validation.

## Common Mistakes

- Relying solely on request-level validation and skipping domain-level invariant checks, leaving objects constructible in an invalid state through any non-HTTP code path (background jobs, other services calling the domain directly).
- Putting complex, multi-field, or database-dependent validation into data annotations, which aren't well suited for it, instead of moving to a dedicated validator.
- Returning inconsistent validation error shapes across different endpoints, making client-side error handling harder than necessary.
- Validating in the controller action body manually instead of relying on the framework's automatic model-state validation, duplicating what `[ApiController]` already provides for free.

## Common Interview Questions

### Basic
- What are data annotations, and how do they integrate with `[ApiController]`?
- When would you reach for FluentValidation instead of data annotations?

### Intermediate
- How would you validate a rule that spans two fields on the same request object?
- Why isn't request-level validation a substitute for domain-level invariant checks?

### Advanced
- How would you design consistent validation error response shapes across many endpoints (tying into `ProblemDetails`)?
- How would you validate a rule that requires a database lookup (e.g., "does this customer exist") without blocking simple, fast request-level checks?

### Follow-up Questions
- Does `[ApiController]`'s automatic validation apply to minimal API endpoints?
- Can FluentValidation validators be unit tested independently of any HTTP context?

### Code Prediction
A `CreateOrderRequest` with `Quantity = -5` is posted to a controller decorated with `[ApiController]` and a `[Range(1, 100)]` attribute on `Quantity`. Does the action method body ever execute? What response does the client receive?

## Practical Tasks

- Add data annotations to a request DTO and verify `[ApiController]`'s automatic `400` response for invalid input.
- Implement a FluentValidation validator for a request with a cross-field rule and a rule requiring a simulated database lookup.
- Design a consistent validation error response shape reusable across multiple endpoints.

## Readiness Criteria

Choose between data annotations and FluentValidation appropriately, implement cross-field and external-lookup validation, and explain why request-level validation doesn't replace domain-level invariant enforcement.

## References

### Microsoft Learn

- [Model validation in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/models/validation)

### Other

- [FluentValidation documentation](https://docs.fluentvalidation.net/)
