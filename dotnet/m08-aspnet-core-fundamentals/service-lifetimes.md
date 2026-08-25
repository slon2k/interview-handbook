# Service Lifetimes: Transient, Scoped, and Singleton

## Definition

A service lifetime controls how long a registered instance lives and how widely it's shared: **Transient** creates a new instance every time it's requested; **Scoped** creates one instance per request (or per explicitly created scope); **Singleton** creates one instance for the entire application lifetime, shared across all requests.

```csharp
builder.Services.AddTransient<IOrderValidator, OrderValidator>();   // new instance every injection
builder.Services.AddScoped<IOrderRepository, EfOrderRepository>();  // one per request
builder.Services.AddSingleton<IClock, SystemClock>();               // one for the app's lifetime
```

## Alternatives & Trade-offs

Transient is safest by default (no shared state, no concurrency concerns) but has the highest allocation cost if the service is requested frequently. Scoped fits most application services and anything wrapping a per-request resource like `DbContext`. Singleton is cheapest to construct but must be thread-safe, since one instance is shared across every concurrent request — and it must never hold a reference to a scoped or transient service directly, or it will keep that instance alive far longer than intended (a "captive dependency").

## How It Works

### The captive dependency problem

```csharp
builder.Services.AddScoped<IOrderRepository, EfOrderRepository>(); // wraps a scoped DbContext
builder.Services.AddSingleton<ICache, MemoryCacheService>();

public class MemoryCacheService : ICache
{
    private readonly IOrderRepository _repository; // WRONG: singleton holding a scoped dependency
    public MemoryCacheService(IOrderRepository repository) => _repository = repository;
}
```

The container captures the *first* request's `EfOrderRepository` (and its underlying `DbContext`) into the singleton `MemoryCacheService`, then reuses that same instance forever — for every subsequent request. The `DbContext` is never disposed correctly and is shared across concurrent requests despite not being thread-safe. `ValidateScopes` (see `dependency-injection-container.md`) catches this at startup instead of letting it fail unpredictably in production.

### The correct pattern: resolve scoped dependencies per-use inside a singleton

```csharp
public class MemoryCacheService : ICache
{
    private readonly IServiceScopeFactory _scopeFactory;
    public MemoryCacheService(IServiceScopeFactory scopeFactory) => _scopeFactory = scopeFactory;

    public async Task RefreshAsync()
    {
        using var scope = _scopeFactory.CreateScope();
        var repository = scope.ServiceProvider.GetRequiredService<IOrderRepository>(); // fresh, per-use instance
        await repository.GetAllAsync();
    }
}
```

### Thread-safety requirement for singletons

```csharp
public class InMemoryCounter : ICounter
{
    private int _count; // shared across every concurrent request — must be accessed safely
    public void Increment() => Interlocked.Increment(ref _count); // atomic, safe under concurrency
}
```

A singleton with mutable state that isn't protected (no `Interlocked`, no lock) is a race condition waiting to happen, since every request shares the exact same instance concurrently.

### One `DbContext` per scope, not per repository call

```csharp
builder.Services.AddDbContext<AppDbContext>(); // registered as Scoped by default
// Multiple repositories injected in the same request share the same DbContext instance,
// which is what makes a single unit-of-work / SaveChanges() across them work correctly
```

## Application

Default to Scoped for most application services, especially anything touching `DbContext` or per-request state. Use Singleton for genuinely stateless services, configuration objects, or caches — with careful attention to thread-safety and never injecting a scoped/transient dependency directly into a singleton's constructor. Use Transient sparingly, for lightweight, stateless, cheaply-constructed services where sharing offers no benefit.

## Common Mistakes

- Injecting a scoped service directly into a singleton's constructor, creating a captive dependency.
- Registering a service with a lifetime that doesn't match its actual state requirements — e.g., a singleton holding per-request data that gets silently shared/corrupted across concurrent requests.
- Assuming Transient services never share state — if a transient service is stored somewhere long-lived by the code that received it (a field on a singleton, a static collection), it effectively becomes accidentally long-lived anyway.
- Forgetting that a mutable singleton must handle concurrent access safely, since it's genuinely shared across every simultaneous request.

## Common Interview Questions

### Basic
- What are the three built-in service lifetimes, and how do they differ?
- Why is `DbContext` typically registered as Scoped?

### Intermediate
- What is a captive dependency, and why does injecting a scoped service into a singleton cause one?
- How would you safely access a scoped dependency from within a singleton service?

### Advanced
- How does `ValidateScopes` detect captive dependencies at startup rather than at runtime?
- What thread-safety obligations does a singleton with mutable state have that a scoped service doesn't?

### Follow-up Questions
- Does registering a service as Transient guarantee it will never end up effectively long-lived?
- Can a scoped service safely depend on a singleton?

### Code Prediction
A singleton `ICache` captures a scoped `IOrderRepository` at construction. Two concurrent requests both resolve `ICache` and call a method that uses the captured repository. What's the risk, given that `DbContext` (which `IOrderRepository` wraps) isn't thread-safe?

## Practical Tasks

- Reproduce a captive-dependency bug (singleton holding a scoped service) and detect it using `ValidateScopes`.
- Fix a captive-dependency scenario using `IServiceScopeFactory` to resolve the scoped dependency per-use.
- Implement a thread-safe singleton counter using `Interlocked`, and explain why a plain `int` field would be unsafe.

## Readiness Criteria

Explain each lifetime's scope and correct use case, diagnose and fix captive-dependency bugs, and recognize the thread-safety obligations of a mutable singleton.

## References

### Microsoft Learn

- [Dependency injection service lifetimes](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection#service-lifetimes)
- [IServiceScopeFactory interface](https://learn.microsoft.com/dotnet/api/microsoft.extensions.dependencyinjection.iservicescopefactory)
