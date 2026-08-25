# The Dependency-Injection Container

## Definition

ASP.NET Core includes a built-in DI container (`IServiceProvider`) that resolves an application's dependency graph. Services are registered against `IServiceCollection` during startup and resolved automatically wherever they're requested — constructor parameters on controllers, minimal API handler parameters, or explicitly via `IServiceProvider.GetRequiredService<T>()`.

```csharp
builder.Services.AddScoped<IOrderRepository, EfOrderRepository>();
builder.Services.AddSingleton<IClock, SystemClock>();

public class OrdersController : ControllerBase
{
    public OrdersController(IOrderRepository repository) => _repository = repository; // resolved automatically
}
```

## Alternatives & Trade-offs

The built-in container covers the vast majority of application needs (constructor injection, the three standard lifetimes, keyed services) with no extra dependency. Third-party containers (Autofac, others) offer more advanced features — property injection, more flexible lifetime scoping, modules — at the cost of an added dependency; most ASP.NET Core teams stick with the built-in container unless a specific missing feature justifies switching.

## How It Works

### Registration and resolution

```csharp
builder.Services.AddScoped<IOrderService, OrderService>();

// Resolved automatically via constructor injection wherever IOrderService is requested
public class OrdersController : ControllerBase
{
    private readonly IOrderService _service;
    public OrdersController(IOrderService service) => _service = service;
}
```

### Resolving manually (rare, generally avoid where DI can do it automatically)

```csharp
var service = app.Services.GetRequiredService<IOrderService>(); // service locator style — avoid in application code
```

Manual resolution via `IServiceProvider` outside of startup/composition code is the service-locator anti-pattern (see Module 4's dependency-injection-and-inversion topic) — it hides a class's real dependencies from its constructor.

### Keyed services (.NET 8+)

```csharp
builder.Services.AddKeyedScoped<INotifier, EmailNotifier>("email");
builder.Services.AddKeyedScoped<INotifier, SmsNotifier>("sms");

public OrdersController([FromKeyedServices("email")] INotifier notifier) { }
```

Keyed services let multiple implementations of the same interface be registered and resolved by a string key, avoiding a hand-rolled factory for a common "pick an implementation by name" scenario.

### Registering with a factory for constructor arguments the container can't supply directly

```csharp
builder.Services.AddScoped<IOrderService>(sp =>
{
    var repository = sp.GetRequiredService<IOrderRepository>();
    return new OrderService(repository, apiKey: builder.Configuration["ExternalApi:Key"]!);
});
```

### Validating the dependency graph at startup

```csharp
builder.Host.UseDefaultServiceProvider(options =>
{
    options.ValidateOnBuild = true;  // fails fast at startup if a dependency can't be resolved
    options.ValidateScopes = true;   // catches captive-dependency mistakes (see service-lifetimes.md) at startup
});
```

## Application

Register application services against interfaces they implement, using constructor injection everywhere. Reach for a factory-based registration when a service needs configuration values or manual construction logic the container can't infer automatically. Enable `ValidateOnBuild`/`ValidateScopes` in development to catch misconfigured lifetimes and missing registrations before they surface as runtime errors in production.

## Common Mistakes

- Resolving services manually via `IServiceProvider` inside application code instead of using constructor injection, hiding real dependencies.
- Registering a dependency with the wrong lifetime relative to what it's injected into, causing captive-dependency bugs (see `service-lifetimes.md`).
- Not enabling `ValidateOnBuild`/`ValidateScopes`, letting a broken registration surface only when a specific code path first executes in production rather than at startup.
- Registering concrete classes instead of interfaces where testability or substitutability is actually needed, achieving DI's wiring convenience without the decoupling benefit.

## Common Interview Questions

### Basic
- What is the built-in ASP.NET Core DI container, and how are services registered?
- What's the difference between resolving a service via constructor injection versus `IServiceProvider.GetRequiredService`?

### Intermediate
- What are keyed services, and what problem do they solve?
- How would you register a service that needs a configuration value at construction time?

### Advanced
- What does `ValidateOnBuild`/`ValidateScopes` catch, and why is enabling it in development valuable?
- When, if ever, would you reach for a third-party DI container instead of the built-in one?

### Follow-up Questions
- Is resolving a service via `app.Services.GetRequiredService<T>()` outside of startup code considered good practice?
- Can the built-in container resolve open generic types?

### Code Prediction
A singleton service is registered with a factory that resolves a scoped `IOrderRepository` from the container at registration time (not per-request). With `ValidateScopes` enabled, what happens when the app starts?

## Practical Tasks

- Register a set of services with appropriate lifetimes and enable `ValidateOnBuild`/`ValidateScopes` to catch a deliberately introduced lifetime mismatch.
- Implement keyed services for a scenario with multiple implementations of the same interface selected at runtime.
- Refactor a manual `IServiceProvider.GetRequiredService` call in application code into proper constructor injection.

## Readiness Criteria

Explain how the built-in DI container resolves dependencies, use keyed services and factory-based registration appropriately, and use startup validation to catch misconfiguration early.

## References

### Microsoft Learn

- [Dependency injection in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/dependency-injection)
- [Keyed service registration](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection#keyed-services)
