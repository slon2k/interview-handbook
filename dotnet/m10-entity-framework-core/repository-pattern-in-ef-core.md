# Repository Pattern in EF Core

## Definition

This topic covers the EF-Core-specific angle of the repository pattern already introduced generally in [Module 4](../m04-oop-design/repository.md): whether and how to wrap `DbContext`/`DbSet<T>` behind a repository interface, and specifically how that interacts with change tracking and `DbContext`'s scoped lifetime.

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id);
    Task AddAsync(Order order);
}

public class EfOrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;
    public EfOrderRepository(AppDbContext context) => _context = context;

    public Task<Order?> GetByIdAsync(int id) => _context.Orders.FirstOrDefaultAsync(o => o.Id == id);
    public Task AddAsync(Order order) { _context.Orders.Add(order); return Task.CompletedTask; } // no SaveChanges here
}
```

## Alternatives & Trade-offs

`DbContext` is already a unit-of-work-plus-repository combination — `DbSet<T>` gives you a queryable, updatable collection per entity type, and `SaveChanges()` commits everything as one operation, which is largely what a hand-rolled repository plus unit-of-work pattern would also give you. Wrapping it in an additional repository interface can still be worth it for a narrower, intention-revealing API and a seam for tests that don't need a real database (see Module 4's general treatment) — but for a straightforward CRUD-heavy application, it can also just be an extra layer forwarding to `DbContext` with little additional value. The decision should be revisited here specifically because of how it interacts with `DbContext`'s scoped lifetime and change tracking, not just as a restatement of the general trade-off.

## How It Works

### Where `SaveChanges()` belongs when a repository exists

```csharp
// If AddAsync calls SaveChanges internally, every repository method becomes its own unit of work —
// which breaks the ability to combine multiple repository calls into one atomic operation
public class EfOrderRepository : IOrderRepository
{
    public async Task AddAsync(Order order)
    {
        _context.Orders.Add(order);
        await _context.SaveChangesAsync(); // now this repository method is its own transaction boundary
    }
}
```

```csharp
// Better: repositories stage changes; a separate call (often on the DbContext itself, or a distinct
// IUnitOfWork abstraction) commits — preserving the ability to combine multiple repository operations
// into one SaveChanges call, matching DbContext's natural unit-of-work behavior
public class EfOrderRepository : IOrderRepository
{
    public Task AddAsync(Order order) { _context.Orders.Add(order); return Task.CompletedTask; }
}

// Calling code combines multiple repository operations, then commits once:
await orderRepository.AddAsync(order);
await inventoryRepository.AdjustStockAsync(productId, -quantity);
await context.SaveChangesAsync(); // one atomic unit of work across both repositories
```

### Repository lifetime must match `DbContext`'s scoped lifetime

```csharp
builder.Services.AddDbContext<AppDbContext>();           // Scoped by default
builder.Services.AddScoped<IOrderRepository, EfOrderRepository>(); // must also be Scoped, sharing the same DbContext per request
```

Registering a repository wrapping `DbContext` with any lifetime other than Scoped (or shorter) risks the exact captive-dependency problem covered in Module 8 — a repository living longer than the `DbContext` it wraps, or wrapping a `DbContext` that's already been disposed.

### Change tracking leaking through a "clean" repository abstraction

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id); // returns a tracked entity by default — is that intentional?
}
```

A repository interface that looks storage-agnostic can still leak EF Core-specific behavior (tracked vs. no-tracking results) to its callers implicitly, unless the interface's contract is explicit about which queries return tracked entities meant for later mutation versus read-only projections.

## Application

If introducing a repository over EF Core, design it so `SaveChanges()` (or an explicit unit-of-work abstraction) remains the single commit point, letting multiple repository operations combine into one atomic unit of work — don't let each repository method manage its own transaction independently. Match the repository's DI lifetime to `DbContext`'s scoped lifetime. Revisit Module 4's general repository trade-off specifically in light of how much `DbContext` already provides on its own before deciding a wrapping layer is worth it.

## Common Mistakes

- Calling `SaveChanges()` inside every repository method, turning each into its own transaction boundary and losing the ability to combine several repository calls into one atomic operation.
- Registering a repository with a lifetime that doesn't match `DbContext`'s scoped lifetime, risking captive-dependency or disposed-context bugs.
- Building a repository abstraction that claims to hide EF Core, while still leaking tracked-vs-untracked query behavior implicitly to its callers.
- Reintroducing this whole discussion from scratch instead of recognizing it as the EF-Core-specific instance of the general trade-off already covered in Module 4.

## Common Interview Questions

### Basic
- Does `DbContext` already function as a repository and unit of work on its own?
- Where should `SaveChanges()` be called when a repository abstraction exists over EF Core?

### Intermediate
- Why does calling `SaveChanges()` inside every repository method undermine the unit-of-work benefit `DbContext` naturally provides?
- What DI lifetime should a repository wrapping `DbContext` use, and why?

### Advanced
- How would you design a repository interface that's explicit about which methods return tracked entities versus read-only projections?
- Given `DbContext`'s built-in unit-of-work behavior, what's the strongest remaining argument for introducing a repository layer over it?

### Follow-up Questions
- Can multiple repositories share the same `DbContext` instance within one request?
- Does introducing a repository interface over EF Core automatically make business logic more testable?

### Code Prediction
Given two repository calls — `orderRepository.AddAsync(order)` and `inventoryRepository.AdjustStockAsync(...)` — where each internally calls its own `SaveChanges()`, what happens if the second call fails after the first has already committed?

## Practical Tasks

- Refactor a repository that calls `SaveChanges()` internally on every method into one where a single external call commits multiple repository operations atomically.
- Register a repository and its underlying `DbContext` with matching Scoped lifetimes and verify no captive-dependency warning occurs with `ValidateScopes`.
- Design a repository interface that's explicit about tracked versus no-tracking query methods.

## Readiness Criteria

Design a repository over EF Core that preserves `DbContext`'s natural unit-of-work behavior, match DI lifetimes correctly, and connect this EF-specific discussion back to Module 4's general repository trade-off rather than treating it as unrelated.

## References

### Microsoft Learn

- [DbContext class](https://learn.microsoft.com/dotnet/api/microsoft.entityframeworkcore.dbcontext)

### Other

- [Repository pattern (Module 4)](../m04-oop-design/repository.md)
