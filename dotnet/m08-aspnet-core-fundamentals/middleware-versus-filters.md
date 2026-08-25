# Middleware vs. Filters

## Definition

**Middleware** operates on every request that reaches the pipeline, before routing has necessarily even determined which endpoint will handle it. **Filters** (action filters, exception filters, authorization filters, result filters) operate within the MVC/controller pipeline specifically, with access to MVC-specific context (the action being invoked, model-binding results, the controller instance).

```csharp
// Middleware: runs for every request, has no idea which controller action (if any) will handle it
app.Use(async (context, next) => { await next(context); });

// Filter: runs only for MVC actions, has access to action-specific context
public class LogActionFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context) { }
    public void OnActionExecuted(ActionExecutedContext context) { }
}
```

## Alternatives & Trade-offs

Middleware is the right layer for concerns that apply uniformly across the whole app regardless of framework feature (authentication, CORS, exception handling, response compression) — it runs even for requests that don't hit an MVC action at all (static files, minimal API endpoints without filters support in older versions). Filters are the right layer for concerns specific to MVC controller actions — they can inspect model-binding results, action arguments, and the controller instance, which middleware cannot see because middleware runs before the framework has resolved any of that.

## How It Works

### What each layer can see

```csharp
// Middleware sees the raw HttpContext only
app.Use(async (context, next) =>
{
    var path = context.Request.Path; // fine
    // no access to "which action will run" or "what arguments were bound" — routing/binding hasn't happened yet
    await next(context);
});

// An action filter sees action-specific context
public class ValidateModelFilter : IActionFilter
{
    public void OnActionExecuting(ActionExecutingContext context)
    {
        if (!context.ModelState.IsValid)
            context.Result = new BadRequestObjectResult(context.ModelState); // short-circuits the action itself
    }
    public void OnActionExecuted(ActionExecutedContext context) { }
}
```

### Filter types and their scope

```
Authorization filters — run first, decide if the request is allowed to proceed
Resource filters       — run after authorization, can short-circuit before/after model binding
Action filters         — run immediately before/after the action method itself
Exception filters      — handle exceptions thrown by the action (MVC-scoped, narrower than global middleware exception handling)
Result filters         — run immediately before/after the action's result is executed (e.g., before a view or JSON result is written)
```

### Ordering: middleware wraps filters

```
Request -> [Middleware pipeline] -> [Routing] -> [Filter pipeline] -> [Action method] -> [Filter pipeline] -> [Middleware pipeline] -> Response
```

Middleware always runs outside the filter pipeline — a request must pass through middleware (including routing) before MVC filters ever get involved.

## Application

Use middleware for cross-cutting concerns needed regardless of whether an MVC action handles the request (authentication, CORS, static files, global exception handling, response compression). Use filters for concerns specific to controller actions that need model-binding or action-specific context — model validation shortcuts, action-specific authorization logic, or result post-processing.

## Common Mistakes

- Implementing a cross-cutting concern (like global exception handling) as a filter when it needs to also cover minimal API endpoints or static file requests that filters don't apply to.
- Implementing something as middleware when it genuinely needs action-specific context (bound model, action arguments) that isn't available yet at the middleware layer.
- Assuming filters run before middleware — they run nested inside the middleware pipeline, specifically inside the routing/endpoint-execution stage.
- Duplicating model validation logic in every action instead of centralizing it in a single action filter.

## Common Interview Questions

### Basic
- What's the main difference between middleware and filters?
- Name the different types of MVC filters.

### Intermediate
- Why can't middleware access action-specific context like bound model values?
- When would you choose a filter over middleware for cross-cutting logic?

### Advanced
- How does the filter pipeline nest inside the middleware pipeline, and what does that imply about what each layer can observe?
- How would you design a reusable validation-shortcut action filter, and what are its limits compared to validating in middleware?

### Follow-up Questions
- Do filters apply to minimal API endpoints the same way they apply to MVC controllers?
- Can an exception filter catch an exception thrown by middleware?

### Code Prediction
An exception is thrown inside custom middleware registered before `UseRouting()`. Will an MVC exception filter registered on a controller catch it? Why or why not?

## Practical Tasks

- Implement a global exception-handling middleware and an MVC exception filter, and identify which exceptions each one can and cannot catch.
- Build a reusable action filter that short-circuits invalid model state with a consistent `400` response.
- Diagram the full request path from middleware through filters to the action method for a single controller request.

## Readiness Criteria

Explain precisely what middleware and filters can each observe, choose the correct layer for a given cross-cutting concern, and reason about how filters nest inside the middleware pipeline.

## References

### Microsoft Learn

- [Filters in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/controllers/filters)
- [ASP.NET Core Middleware](https://learn.microsoft.com/aspnet/core/fundamentals/middleware/)
