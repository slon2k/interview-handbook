# Repositories and Units of Work — the Architecture-Level View

## Definition

This topic is the third and final angle on the repository pattern in this handbook, specifically about *where it sits in a layered/hexagonal architecture* and how it relates to a **unit of work** (a boundary grouping several changes into one atomic save). Module 4 covered the general design trade-off; Module 10 covered the EF-Core-specific mechanics. Here, the question is architectural placement: repositories are an outbound port (previous topics), owned by the domain/application layer, implemented by infrastructure.

```
Domain/Application layer:  defines IOrderRepository, IUnitOfWork (the PORTS)
Infrastructure layer:      implements EfOrderRepository, EfUnitOfWork (the ADAPTERS)
```

## Alternatives & Trade-offs

Letting application services call `DbContext`/`SaveChanges()` directly (Module 10's territory) is simpler and avoids an extra abstraction, appropriate when the application is small enough that swapping persistence technology or isolating from EF Core for testing isn't a real concern. Introducing repository and unit-of-work ports at the architecture level buys a genuine technology-independence boundary — the application/domain layers never reference EF Core at all — which matters more as a system grows, has multiple bounded modules (previous modular-monolith topic), or genuinely needs infrastructure-independent testing.

## How It Works

### The unit of work as an explicit port, when EF Core's own `DbContext` isn't allowed past the boundary

```csharp
// Domain/application layer's port — no mention of EF Core anywhere
public interface IUnitOfWork
{
    IOrderRepository Orders { get; }
    IInventoryRepository Inventory { get; }
    Task<int> SaveChangesAsync();
}

// Infrastructure's adapter — EF Core specifics live here, and ONLY here
public class EfUnitOfWork : IUnitOfWork
{
    private readonly AppDbContext _context;
    public IOrderRepository Orders { get; }
    public IInventoryRepository Inventory { get; }
    public Task<int> SaveChangesAsync() => _context.SaveChangesAsync();
}
```

```csharp
public class PlaceOrderApplicationService
{
    private readonly IUnitOfWork _unitOfWork; // never sees DbContext, ever

    public async Task ExecuteAsync(PlaceOrderRequest request)
    {
        await _unitOfWork.Inventory.ReserveStockAsync(request.Sku, request.Quantity);
        await _unitOfWork.Orders.AddAsync(Order.Create(request));
        await _unitOfWork.SaveChangesAsync(); // ONE atomic commit across both repositories
    }
}
```

This is architecturally identical to what `DbContext` already gives you "for free" per Module 10 — the explicit `IUnitOfWork` port exists specifically to keep that unit-of-work capability available to the application layer *without* exposing `DbContext` itself past the domain/infrastructure boundary.

### When this extra abstraction genuinely isn't worth it

```
For a small application with one bounded context, no plan to swap persistence technology,
and integration tests already covering the database layer directly (Module 10/11's testing
content), introducing IUnitOfWork on top of DbContext may just be ceremony — DbContext's own
unit-of-work behavior can be used directly by the application layer, accepting that dependency.
```

### How this connects the three repository discussions in this handbook into one coherent view

```
Module 4:  should you have a repository at all, and what's the general design trade-off?
Module 10: given a repository, how does it interact with EF Core's change tracking and lifetime?
Module 14 (here): where does the repository/unit-of-work PORT live architecturally, and when does
                    that extra boundary (versus using DbContext directly) actually pay for itself?
```

## Application

Introduce explicit repository and unit-of-work ports at the architecture level specifically when the domain/application layers must stay technology-independent (a modular monolith with a possible future service split, a system requiring infrastructure-free domain testing) — not as a default for every application, echoing this module's later topic on avoiding unnecessary architecture.

## Common Mistakes

- Introducing `IUnitOfWork`/`IRepository` ports without a real reason, when the application is small enough that `DbContext` used directly by the application layer would be simpler and equally testable (Module 10's testing strategies still apply either way).
- Defining the repository/unit-of-work interfaces in the infrastructure layer rather than the domain/application layer, inverting the ownership this whole module's dependency-direction discussion is about.
- Treating this as an entirely new discussion disconnected from Module 4 and Module 10, rather than the architectural-placement angle on the same recurring decision.
- Leaking `DbContext`-specific types (like `IQueryable<T>` results with EF Core-specific behavior) through the supposedly technology-independent `IUnitOfWork` interface.

## Common Interview Questions

### Basic
- What is a unit of work, and how does it relate to a repository?
- Where should repository and unit-of-work interfaces be defined in a layered architecture?

### Intermediate
- What does introducing an explicit `IUnitOfWork` port provide that using `DbContext` directly in the application layer doesn't?
- When would this extra abstraction not be worth its cost?

### Advanced
- How would you design `IUnitOfWork` and repository ports to genuinely keep the domain/application layers free of any EF Core-specific type, including in method return types?
- How do this module's three separate discussions of the repository pattern (Module 4, 10, 14) fit together as one coherent design decision viewed from different angles?

### Follow-up Questions
- Does using `IUnitOfWork` prevent the captive-dependency and lifetime issues covered in Module 10?
- Can a system use `DbContext` directly for some modules and an explicit `IUnitOfWork` for others?

### Code Prediction
Given `IUnitOfWork.SaveChangesAsync()` implemented by wrapping `DbContext.SaveChangesAsync()`, if `PlaceOrderApplicationService` calls both `_unitOfWork.Orders.AddAsync(...)` and `_unitOfWork.Inventory.ReserveStockAsync(...)` before calling `SaveChangesAsync()` once, are both changes committed atomically together? Why does that match `DbContext`'s own natural behavior from Module 10?

## Practical Tasks

- Design `IUnitOfWork` and repository ports for a small application, ensuring no EF Core-specific type crosses the domain/application boundary.
- Compare an application service using `IUnitOfWork` against one using `DbContext` directly, and articulate when each is the more appropriate choice.
- Audit a hypothetical `IUnitOfWork` interface for EF Core-specific leakage (e.g., a method returning `IQueryable<T>`) and fix it.

## Readiness Criteria

Place repository and unit-of-work abstractions correctly within a layered architecture, judge when the extra abstraction over direct `DbContext` usage is or isn't justified, and connect this discussion to the general (Module 4) and EF-Core-specific (Module 10) treatments already covered.

## References

### Other

- [Repository pattern (Module 4)](../m04-oop-design/repository.md)
- [Repository pattern in EF Core (Module 10)](../m10-entity-framework-core/repository-pattern-in-ef-core.md)
