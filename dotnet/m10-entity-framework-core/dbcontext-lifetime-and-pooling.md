# DbContext Lifetime, Thread Safety, and Pooling

## Definition

`DbContext` is not thread-safe — a single instance must never be used by more than one operation concurrently. It's registered as **Scoped** by default (one instance per request), which is why constructor-injecting it into a Singleton causes the captive-dependency problem covered in Module 8. **DbContext pooling** (`AddDbContextPool`) reuses `DbContext` instances across requests instead of constructing a new one each time, reducing allocation overhead for high-throughput applications.

```csharp
builder.Services.AddDbContext<AppDbContext>(options => options.UseSqlServer(connectionString)); // Scoped by default
builder.Services.AddDbContextPool<AppDbContext>(options => options.UseSqlServer(connectionString)); // pooled
```

## Alternatives & Trade-offs

The default `AddDbContext` registration creates a fresh instance per scope — simple and safe, with a small allocation cost per request. `AddDbContextPool` reduces that allocation cost by returning instances to a pool for reuse instead of discarding them, which matters at high request volume, but requires that the context's constructor and configuration have no per-request state baked in beyond what EF Core resets automatically — a pooled context is reset between uses, but any custom state you add to it yourself is not automatically cleared.

## How It Works

### Why DbContext isn't thread-safe

```csharp
var context = new AppDbContext();
var task1 = context.Orders.Where(o => o.Status == "Pending").ToListAsync();
var task2 = context.Orders.Where(o => o.Status == "Shipped").ToListAsync();
await Task.WhenAll(task1, task2); // InvalidOperationException: a second operation was started on this context
                                    // before a previous operation completed
```

A single `DbContext` instance can only have one operation in flight at a time. Running two queries concurrently on the same instance — even with `Task.WhenAll` — throws, because EF Core's internal state isn't designed for concurrent access.

### The captive-dependency risk from Module 8, applied to DbContext specifically

```csharp
public class ReportCacheService // registered as Singleton
{
    private readonly AppDbContext _context; // WRONG: captures a scoped context inside a singleton
    public ReportCacheService(AppDbContext context) => _context = context;
}
```

The first request's `DbContext` gets captured into the singleton and reused (incorrectly) for the lifetime of the app — a `DbContext` designed to live for one request now lives forever, shared unsafely across every subsequent request. Resolve it per-use via `IServiceScopeFactory` instead, as shown in Module 8's `service-lifetimes.md`.

### DbContext pooling

```csharp
builder.Services.AddDbContextPool<AppDbContext>(options =>
    options.UseSqlServer(connectionString), poolSize: 128);
```

When a scope ends, instead of disposing the `AppDbContext` entirely, EF Core resets its internal state (clears the change tracker, resets scalar properties on the context itself set via its constructor parameters) and returns it to the pool for the next request to reuse — avoiding the cost of constructing a brand-new context and its internal services each time.

### What pooling doesn't reset automatically

```csharp
public class AppDbContext : DbContext
{
    public string? CurrentUserId; // custom mutable state you added yourself

    public AppDbContext(DbContextOptions<AppDbContext> options) : base(options) { }
}
```

If code sets `context.CurrentUserId` during one request and the context is pooled and reused, that field is *not* automatically cleared — only EF Core's own internal tracking state is reset. Custom per-request state added directly to a pooled context is a common source of subtle cross-request data leakage.

## Application

Always treat `DbContext` as single-threaded — never share one instance across concurrent operations, and resolve it fresh (or from the pool) per logical unit of work. Use `AddDbContextPool` for high-throughput APIs where context construction overhead is measurable, but avoid adding custom mutable state directly onto the context class if pooling is enabled, or reset it explicitly.

## Common Mistakes

- Running concurrent queries against the same `DbContext` instance, causing an `InvalidOperationException` at best or silently corrupted state at worst.
- Injecting a scoped `DbContext` into a singleton service, creating the captive-dependency bug from Module 8.
- Enabling `AddDbContextPool` while also adding custom mutable state directly to the `DbContext` subclass, causing state to leak between unrelated requests.
- Assuming pooling is always a performance win — for low-to-moderate traffic, the added complexity may not be worth it over the simpler `AddDbContext` registration.

## Common Interview Questions

### Basic
- Is `DbContext` thread-safe?
- What is `DbContext` pooling, and what problem does it solve?

### Intermediate
- What happens if two concurrent operations are started on the same `DbContext` instance?
- What's the risk of adding custom mutable state directly to a pooled `DbContext` subclass?

### Advanced
- How would you safely use a scoped `DbContext` from within a singleton-hosted background service?
- When does `DbContext` pooling actually provide a measurable benefit, and when is it unnecessary complexity?

### Follow-up Questions
- Does `DbContext` pooling reset a context's change tracker between uses?
- Can a `DbContext` be shared safely across multiple threads if each only reads (never writes)?

### Code Prediction
Given a pooled `AppDbContext` with a custom `public string? TenantId` field set once per request, what happens if request A sets `TenantId = "tenant-1"` and, due to pooling, request B receives the same underlying instance without explicitly resetting `TenantId`?

## Practical Tasks

- Reproduce the concurrent-operation exception by running two queries against the same `DbContext` instance with `Task.WhenAll`, then fix it using separate instances.
- Configure `AddDbContextPool` for an application and identify any custom state on the context that would need explicit resetting.
- Resolve a scoped `DbContext` safely from within a singleton-hosted background service using `IServiceScopeFactory`.

## Readiness Criteria

Explain why `DbContext` isn't thread-safe with a concrete failure example, correctly scope its lifetime via DI, and use pooling appropriately while avoiding custom-state leakage.

## References

### Microsoft Learn

- [DbContext lifetime, configuration, and initialization](https://learn.microsoft.com/ef/core/dbcontext-configuration/)
- [DbContext pooling](https://learn.microsoft.com/ef/core/performance/advanced-performance-topics#dbcontext-pooling)
