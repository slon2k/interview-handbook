# Model Binding

## Definition

Model binding is the process ASP.NET Core uses to populate action/handler method parameters from an incoming request — route values, query string, headers, form data, or a JSON body — converting raw request data into strongly-typed .NET objects automatically.

```csharp
[HttpGet("{id:int}")]
public IActionResult Get(int id, [FromQuery] bool includeItems) => Ok();
// id comes from the route, includeItems comes from ?includeItems=true
```

## Alternatives & Trade-offs

Automatic model binding removes a huge amount of manual parsing code (`Request.Query["x"]`, `int.Parse(...)`) and centralizes conversion/error handling. The trade-off is that binding source inference (deciding automatically whether a parameter comes from the route, query, or body) can be surprising if not well understood, and binding failures for complex types can be harder to diagnose than a hand-written parse with an explicit error message.

## How It Works

### Binding source inference

```csharp
[HttpPost]
public IActionResult Create(CreateOrderRequest request) // complex type -> inferred from the request body
{ }

[HttpGet]
public IActionResult Search(string status, int page) // simple types -> inferred from the query string
{ }
```

`[ApiController]`-decorated controllers infer binding sources by convention: complex types default to the body, simple types default to route values or the query string. Explicit attributes (`[FromBody]`, `[FromQuery]`, `[FromRoute]`, `[FromHeader]`, `[FromForm]`) override the inference when needed.

### Binding a complex type from multiple sources

```csharp
public class CreateOrderRequest
{
    public int CustomerId { get; set; }
    public List<OrderItem> Items { get; set; } = new();
}

[HttpPost("{customerId:int}/orders")]
public IActionResult Create([FromRoute] int customerId, [FromBody] CreateOrderRequest request)
{
    // customerId comes from the route even though the body also has a CustomerId property —
    // these are two separate values unless the code explicitly reconciles them
}
```

### Binding failures

```csharp
[HttpGet]
public IActionResult Get(int id) => Ok(); // request: GET /orders/abc

// "abc" cannot bind to int -> with [ApiController], this automatically produces a 400
// without [ApiController], `id` would be left as default(int) == 0, silently wrong
```

This is one reason `[ApiController]`'s automatic `400` behavior (see `controllers-versus-minimal-apis.md`) matters — without it, a binding failure for a value type can silently default instead of surfacing an error.

### Minimal API binding

```csharp
app.MapPost("/orders", (CreateOrderRequest request, [FromServices] IOrderService service) => { });
// complex type parameters bind from the body by default; [FromServices] explicitly requests DI injection
// instead of treating the parameter as request data
```

## Application

Rely on binding-source inference for straightforward cases, and use explicit `[From...]` attributes whenever a parameter's source would otherwise be ambiguous or when overriding the default inference (e.g., pulling a complex filter object from the query string instead of the body). Always be aware of whether a value-type binding failure produces a clear error or a silent default.

## Common Mistakes

- Relying on binding-source inference in ambiguous cases without verifying it picked the source actually intended, especially when mixing route and body values for the same conceptual field.
- Not realizing that without `[ApiController]`'s automatic validation, a failed binding for a simple type can silently produce a default value (`0`, `null`, `false`) instead of an error.
- Forgetting `[FromServices]` on a minimal API parameter intended for DI, causing the framework to attempt to bind it from request data instead.
- Binding directly to EF Core entities instead of dedicated request DTOs, which can allow unintended fields (like an `IsAdmin` flag) to be set via mass assignment/over-posting.

## Common Interview Questions

### Basic
- What is model binding, and what request sources can it bind from?
- What does `[FromBody]` do?

### Intermediate
- How does ASP.NET Core infer a parameter's binding source when no attribute is specified?
- What happens if a route value can't be converted to the target parameter's type?

### Advanced
- Why is binding directly to domain/EF Core entities considered risky, and what pattern avoids it?
- How would you bind a complex filter object from query-string parameters instead of the request body?

### Follow-up Questions
- Does model binding happen before or after routing has matched an endpoint?
- Can a single action bind values from more than one source at once?

### Code Prediction
`GET /orders/abc` is sent to an action with parameter `int id`, on a controller without `[ApiController]`. What value does `id` end up with inside the method body, and why is this more dangerous than an outright `400` error?

## Practical Tasks

- Add explicit `[From...]` attributes to a set of ambiguous parameters and verify the correct binding source.
- Reproduce the silent-default-on-binding-failure behavior on a controller without `[ApiController]`, then fix it.
- Refactor an action that binds directly to an EF Core entity to instead bind to a dedicated request DTO.

## Readiness Criteria

Explain binding-source inference and its failure modes precisely, use explicit binding attributes correctly, and recognize the mass-assignment risk of binding directly to domain entities.

## References

### Microsoft Learn

- [Model binding in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/models/model-binding)
- [Custom model binding](https://learn.microsoft.com/aspnet/core/mvc/advanced/custom-model-binding)
