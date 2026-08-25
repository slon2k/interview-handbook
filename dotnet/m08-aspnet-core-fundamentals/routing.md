# Routing

## Definition

Routing matches an incoming request's URL and HTTP method to the endpoint (a controller action or minimal API handler) that should process it, extracting route parameters along the way. ASP.NET Core uses **endpoint routing**, where routes are resolved once (`UseRouting`) and executed later (`UseEndpoints`/`MapControllers`/`Map...`), so other middleware can inspect the matched endpoint before it actually runs.

```csharp
app.MapGet("/orders/{id:int}", (int id) => $"Order {id}");

[Route("api/[controller]")]
public class OrdersController : ControllerBase
{
    [HttpGet("{id:int}")]
    public IActionResult Get(int id) => Ok(new { id });
}
```

## Alternatives & Trade-offs

Attribute routing (used on controllers) keeps a route definition next to the action it maps to, which is easy to read for CRUD-heavy controllers with many similarly-shaped routes. Minimal API routing (`app.MapGet`, etc.) defines routes centrally in `Program.cs`, which is more concise for smaller APIs but can become unwieldy if a large number of endpoints all live in one file without further organization (route groups help here).

## How It Works

### Route templates and constraints

```csharp
app.MapGet("/orders/{id:int}", (int id) => $"Order {id}");        // {id:int} constrains to integers
app.MapGet("/orders/{id:int:min(1)}", (int id) => $"Order {id}"); // combined constraints
app.MapGet("/orders/{orderId}/items/{itemId}", (int orderId, int itemId) => $"{orderId}/{itemId}");
```

A request to `/orders/abc` won't match `{id:int}` at all — it falls through to the next matching route or a `404`, rather than reaching the handler with an invalid value.

### Route precedence and ambiguity

```csharp
app.MapGet("/orders/{id}", (string id) => "generic");
app.MapGet("/orders/pending", () => "pending"); // more specific literal segment — matched first for this exact path
```

ASP.NET Core's routing engine orders matches by specificity (literal segments beat parameterized ones), so `/orders/pending` correctly hits the second route rather than being captured by `{id}`.

### Route groups (minimal APIs)

```csharp
var orders = app.MapGroup("/orders").WithTags("Orders");
orders.MapGet("/{id:int}", (int id) => $"Order {id}");
orders.MapPost("/", (Order order) => Results.Created($"/orders/{order.Id}", order));
```

Route groups let minimal APIs share a prefix, tags, and even filters/middleware across a set of related endpoints without repeating configuration.

### Endpoint metadata

```csharp
app.MapGet("/orders/{id:int}", (int id) => $"Order {id}")
   .RequireAuthorization()
   .WithName("GetOrder");
```

Routing attaches metadata (authorization requirements, OpenAPI info, rate-limiting policy) to the matched endpoint, which later middleware (like `UseAuthorization`) reads to decide how to treat the request.

## Application

Use attribute routing for controller-based APIs with many related actions sharing a resource path. Use minimal API route groups for smaller or more focused APIs, or when a lightweight functional style is preferred. Use route constraints (`:int`, `:guid`, `:min()`) to reject malformed input at the routing layer before it reaches application code.

## Common Mistakes

- Defining ambiguous routes that could match the same request, relying on registration order instead of explicit constraints or more specific literal segments.
- Not using route constraints, letting malformed input (a non-numeric ID) reach the handler as a raw string, requiring manual validation that a constraint would have handled for free.
- Forgetting that `UseRouting()` and `UseEndpoints()`/the endpoint-mapping calls must be positioned correctly relative to other middleware (e.g., authorization must run after routing has matched an endpoint, since it needs to read that endpoint's metadata).
- Mixing controller and minimal API routing inconsistently across a codebase without a clear convention for which style is used where.

## Common Interview Questions

### Basic
- What is endpoint routing, and how does route matching work?
- What does a route constraint like `{id:int}` do?

### Intermediate
- How does ASP.NET Core resolve ambiguity between a literal route segment and a parameterized one?
- What is a route group, and what problem does it solve for minimal APIs?

### Advanced
- How does endpoint metadata (attached during routing) get used by later middleware like authorization?
- Why must routing happen before authorization in the middleware pipeline?

### Follow-up Questions
- Can route constraints replace all manual input validation for route parameters?
- Do attribute-routed controllers and minimal API endpoints share the same underlying routing engine?

### Code Prediction
Given `app.MapGet("/orders/{id}", ...)` and `app.MapGet("/orders/pending", ...)` registered in either order, does a request to `/orders/pending` ever get incorrectly captured by the `{id}` route? Why or why not?

## Practical Tasks

- Add route constraints to a set of endpoints and verify malformed input is rejected with `404` before reaching the handler.
- Reorganize a flat list of minimal API endpoints into route groups with shared prefixes and authorization requirements.
- Diagnose an ambiguous-route scenario and resolve it using constraints or more specific segments.

## Readiness Criteria

Explain endpoint routing and its relationship to the middleware pipeline, use route constraints and groups effectively, and diagnose route ambiguity.

## References

### Microsoft Learn

- [Routing in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/routing)
- [Route to controller actions](https://learn.microsoft.com/aspnet/core/mvc/controllers/routing)
