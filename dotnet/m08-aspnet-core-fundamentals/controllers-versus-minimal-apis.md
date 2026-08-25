# Controllers vs. Minimal APIs

## Definition

**Controllers** (MVC-style, `ControllerBase`-derived classes with attribute routing) are the traditional ASP.NET Core approach: class-based, with built-in support for filters, model binding conventions, and a well-established structure. **Minimal APIs** define endpoints as inline delegates directly against `WebApplication`, aiming for less ceremony for smaller services.

```csharp
// Controller
[ApiController]
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    private readonly IOrderService _service;
    public OrdersController(IOrderService service) => _service = service;

    [HttpGet("{id:int}")]
    public async Task<IActionResult> Get(int id)
    {
        var order = await _service.GetAsync(id);
        return order is null ? NotFound() : Ok(order);
    }
}

// Minimal API
app.MapGet("/api/orders/{id:int}", async (int id, IOrderService service) =>
{
    var order = await service.GetAsync(id);
    return order is null ? Results.NotFound() : Results.Ok(order);
});
```

## Alternatives & Trade-offs

Controllers bring built-in structure (filters, conventions, `[ApiController]` behaviors like automatic model-validation `400` responses) that scales well for larger APIs with many related actions and shared cross-cutting concerns. Minimal APIs trade some of that built-in convention for less boilerplate and faster startup, which suits small services, microservices with few endpoints, or scenarios where startup performance genuinely matters (e.g., serverless/cold-start-sensitive deployments). Neither is objectively "better" — the choice depends on API size and how much of the MVC convention machinery is actually needed.

## How It Works

### `[ApiController]` behaviors controllers get "for free"

```csharp
[ApiController] // enables automatic 400 on invalid model state, binding source inference, problem-details responses
[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpPost]
    public IActionResult Create(CreateOrderRequest request) // invalid request automatically short-circuits to 400
    {
        // this line only runs if ModelState is already valid
        return Ok();
    }
}
```

Minimal APIs don't get this automatically — validation must be handled explicitly (manually, via a filter, or via a validation library) inside or around the handler.

### Minimal APIs with explicit validation

```csharp
app.MapPost("/api/orders", (CreateOrderRequest request, IValidator<CreateOrderRequest> validator) =>
{
    var result = validator.Validate(request);
    if (!result.IsValid) return Results.ValidationProblem(result.ToDictionary());
    return Results.Created($"/api/orders/1", request);
});
```

### Dependency injection works the same way in both

```csharp
// Controller: constructor injection
public OrdersController(IOrderService service) { }

// Minimal API: parameter injection, resolved per-request from the DI container
app.MapGet("/orders", (IOrderService service) => service.GetAllAsync());
```

### Sharing cross-cutting behavior in minimal APIs

```csharp
var orders = app.MapGroup("/api/orders").RequireAuthorization().WithOpenApi();
orders.MapGet("/{id:int}", GetOrder);
orders.MapPost("/", CreateOrder);
```

Route groups (see `routing.md`) are how minimal APIs recover some of the "shared configuration across related endpoints" convenience that controllers get from their class-level attributes.

## Application

Use controllers for larger APIs with many related actions, teams already familiar with MVC conventions, or where filters and `[ApiController]` behaviors provide real value. Use minimal APIs for small, focused services, prototypes, or scenarios prioritizing startup performance and minimal ceremony — and use route groups to keep them organized once the endpoint count grows.

## Common Mistakes

- Assuming minimal APIs automatically get `[ApiController]`-style automatic model validation — they don't, and skipping explicit validation leaves invalid requests reaching handler code.
- Choosing minimal APIs for a large, complex API and ending up re-implementing filter-like cross-cutting behavior manually and inconsistently across many route handlers.
- Assuming controllers are "legacy" — both styles are actively supported and maintained; the choice is about fit, not deprecation.
- Mixing both styles in one codebase without a clear rule for when each is used, making the codebase harder to navigate.

## Common Interview Questions

### Basic
- What's the main structural difference between controllers and minimal APIs?
- What does `[ApiController]` add automatically to a controller?

### Intermediate
- Why doesn't a minimal API automatically return `400` for invalid model state the way a controller with `[ApiController]` does?
- How do route groups help organize minimal APIs as they grow?

### Advanced
- What are the practical trade-offs (startup time, structure, team familiarity) that would push a team toward one style over the other for a new service?
- How would you replicate `[ApiController]`'s automatic validation behavior in a minimal API?

### Follow-up Questions
- Do controllers and minimal APIs use the same underlying routing engine?
- Can filters be applied to minimal API endpoints?

### Code Prediction
A `CreateOrderRequest` with a missing required field is posted to a controller action decorated with `[ApiController]`, and separately to an equivalent minimal API endpoint with no explicit validation. What status code does each return, and why do they differ?

## Practical Tasks

- Implement the same small CRUD API twice — once with controllers, once with minimal APIs — and compare the amount of boilerplate.
- Add explicit validation to a minimal API endpoint to match the automatic `400` behavior controllers get from `[ApiController]`.
- Organize a set of minimal API endpoints into route groups with shared authorization and tags.

## Readiness Criteria

Explain the structural and behavioral differences between controllers and minimal APIs, including what `[ApiController]` provides automatically, and justify a style choice for a given API's size and requirements.

## References

### Microsoft Learn

- [Controller-based APIs](https://learn.microsoft.com/aspnet/core/web-api/)
- [Minimal APIs overview](https://learn.microsoft.com/aspnet/core/fundamentals/minimal-apis)
