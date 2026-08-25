# The Request Pipeline and Middleware

## Definition

The ASP.NET Core request pipeline is an ordered chain of **middleware** components, each able to inspect or modify the request/response and decide whether to call the next component. It's a direct application of the Decorator pattern to HTTP request handling — order is not cosmetic, it determines behavior.

```csharp
var app = builder.Build();

app.UseExceptionHandler("/error");
app.UseHttpsRedirection();
app.UseAuthentication();
app.UseAuthorization();
app.MapControllers();

app.Run();
```

## Alternatives & Trade-offs

The middleware pipeline centralizes cross-cutting concerns (exception handling, auth, logging) outside individual endpoints, so every request passes through the same well-defined chain instead of duplicating that logic in every handler. The trade-off is that pipeline order becomes a real source of bugs if misunderstood — unlike calling a few independent helper methods, middleware order changes what earlier and later components actually see.

## How It Works

### Order determines behavior

```csharp
app.UseAuthorization();      // WRONG ORDER: runs before authentication has identified the user
app.UseAuthentication();
```

```csharp
app.UseAuthentication();     // CORRECT: identify the user first
app.UseAuthorization();      // then decide if they're allowed to proceed
```

`UseAuthorization` needs `HttpContext.User` to already be populated by `UseAuthentication`; reversing the order means authorization checks run against an unauthenticated context and will incorrectly reject every request (or, worse, silently behave as if no policy applies, depending on configuration).

### The pipeline is a chain of delegates, each wrapping the next

```csharp
app.Use(async (context, next) =>
{
    Console.WriteLine("Before");
    await next(context); // calls the next middleware in the chain
    Console.WriteLine("After");
});
```

Because each middleware wraps the next, code before `await next()` runs on the way in (request phase), and code after runs on the way out (response phase) — the same "wrap and delegate" shape as the Decorator pattern.

### Short-circuiting

```csharp
app.Use(async (context, next) =>
{
    if (!context.Request.Headers.ContainsKey("X-Api-Key"))
    {
        context.Response.StatusCode = 401;
        return; // does NOT call next() — the pipeline stops here
    }
    await next(context);
});
```

A middleware that doesn't call `next()` short-circuits the pipeline; nothing registered after it runs for that request.

### Branching the pipeline

```csharp
app.MapWhen(ctx => ctx.Request.Path.StartsWithSegments("/admin"), adminApp =>
{
    adminApp.UseMiddleware<AdminOnlyMiddleware>();
});
```

## Application

Register middleware in the order the request actually needs to be processed: exception handling first (so it can catch everything downstream), then HTTPS redirection, then authentication, then authorization, then routing/endpoint execution last. Use short-circuiting for cross-cutting checks (API keys, health-check bypass) that shouldn't reach the rest of the pipeline.

## Common Mistakes

- Registering `UseAuthorization` before `UseAuthentication`, breaking every authorization check.
- Placing exception-handling middleware too late in the pipeline, so it can't catch exceptions thrown by middleware registered before it.
- Forgetting to call `next()` in custom middleware unintentionally, silently short-circuiting the pipeline and breaking every request that passes through it.
- Assuming middleware order doesn't matter because "it all runs anyway" — every middleware runs, but what each one sees (headers already added, user already authenticated, response already started) depends entirely on order.

## Common Interview Questions

### Basic
- What is middleware in ASP.NET Core, and what does the pipeline represent?
- What does calling `next()` inside a middleware do?

### Intermediate
- Why must `UseAuthentication` be registered before `UseAuthorization`?
- What does it mean for middleware to "short-circuit" the pipeline?

### Advanced
- How does the middleware pipeline resemble the Decorator pattern, and what does that analogy predict about ordering behavior?
- How would you branch the pipeline so a subset of requests get different middleware treatment (e.g., `/admin` routes)?

### Follow-up Questions
- What happens if a middleware forgets to call `next()`?
- Can middleware run code both before and after the rest of the pipeline executes?

### Code Prediction
```csharp
app.Use(async (context, next) => { Console.WriteLine("A-in"); await next(context); Console.WriteLine("A-out"); });
app.Use(async (context, next) => { Console.WriteLine("B-in"); await next(context); Console.WriteLine("B-out"); });
app.Run(async context => Console.WriteLine("Endpoint"));
```
In what order do these print for a single request?

## Practical Tasks

- Write custom middleware that logs request start/end times using the before/after `next()` pattern.
- Reproduce the authentication/authorization ordering bug and observe how it breaks a protected endpoint.
- Implement a short-circuiting API-key-check middleware that rejects unauthenticated requests before they reach any endpoint.

## Readiness Criteria

Explain the middleware pipeline as an ordered chain of delegates, correctly reason about ordering-dependent bugs, and implement custom middleware including short-circuiting behavior.

## References

### Microsoft Learn

- [ASP.NET Core Middleware](https://learn.microsoft.com/aspnet/core/fundamentals/middleware/)
- [Write custom ASP.NET Core middleware](https://learn.microsoft.com/aspnet/core/fundamentals/middleware/write)
