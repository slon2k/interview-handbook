# Repository Pattern

## Definition

The Repository pattern provides a collection-like interface for accessing domain objects, hiding the underlying data-access technology (SQL, EF Core, an external API) behind an abstraction the rest of the application depends on.

```csharp
public interface IOrderRepository
{
    Task<Order?> GetByIdAsync(int id);
    Task AddAsync(Order order);
    Task<IReadOnlyList<Order>> GetByCustomerAsync(int customerId);
}
```

## Alternatives & Trade-offs

EF Core's `DbSet<T>` and `DbContext` already implement the Unit of Work and, arguably, a repository-like abstraction over the database. Adding a custom repository on top gives you:

- Pros: a narrower, intention-revealing API (`GetActiveCustomers()` instead of raw LINQ scattered across services); a seam for unit testing without a real database; the ability to swap the storage technology later.
- Cons: for straightforward CRUD apps, a repository can become a thin, mechanical wrapper that just forwards to `DbContext` ("repository over repository"), adding a layer with little payoff. In simple services, injecting `DbContext` (or `IQueryable`) directly and using integration tests against a real/test database is often more pragmatic.

The right choice depends on team conventions, testing strategy, and how much the persistence technology is expected to change.

## How It Works

```csharp
public sealed class EfOrderRepository : IOrderRepository
{
    private readonly AppDbContext _context;

    public EfOrderRepository(AppDbContext context) => _context = context;

    public Task<Order?> GetByIdAsync(int id) =>
        _context.Orders.FirstOrDefaultAsync(o => o.Id == id);

    public Task AddAsync(Order order)
    {
        _context.Orders.Add(order);
        return _context.SaveChangesAsync();
    }

    public Task<IReadOnlyList<Order>> GetByCustomerAsync(int customerId) =>
        _context.Orders
            .Where(o => o.CustomerId == customerId)
            .ToListAsync()
            .ContinueWith(t => (IReadOnlyList<Order>)t.Result);
}
```

Business logic depends on `IOrderRepository`, never on `AppDbContext` or SQL directly. For tests, a fake or in-memory implementation replaces `EfOrderRepository` without touching a real database.

```csharp
public sealed class FakeOrderRepository : IOrderRepository
{
    private readonly List<Order> _orders = new();
    public Task<Order?> GetByIdAsync(int id) => Task.FromResult(_orders.FirstOrDefault(o => o.Id == id));
    public Task AddAsync(Order order) { _orders.Add(order); return Task.CompletedTask; }
    public Task<IReadOnlyList<Order>> GetByCustomerAsync(int customerId) =>
        Task.FromResult((IReadOnlyList<Order>)_orders.Where(o => o.CustomerId == customerId).ToList());
}
```

## Application

Use a repository when domain logic needs to be tested without a database, when the persistence technology might change, or when raw query logic would otherwise leak into multiple services. Skip it for small CRUD-only services where `DbContext` injection plus integration tests against a real test database is simpler and equally testable.

## Common Mistakes

- Making the repository interface leak EF Core types (`IQueryable<T>`, `DbSet<T>`) into calling code, defeating the point of the abstraction.
- Writing a generic `IRepository<T>` with `GetAll()`, `Add()`, `Update()`, `Delete()` for every entity, which tends to encourage anemic, query-less domain services and still forces callers to compose filtering logic outside the repository.
- Adding a repository layer purely out of habit in a small service where it provides no real seam or benefit.
- Confusing repository with Unit of Work: a repository manages access to one aggregate/entity type; Unit of Work coordinates saving changes across several.

## Common Interview Questions

### Basic
- What problem does the Repository pattern solve?
- Is `DbContext` already a form of repository?

### Intermediate
- When would you skip a repository layer entirely in a .NET backend?
- What's wrong with a generic `IRepository<T>` used everywhere?
- How does a repository make unit testing easier?

### Advanced
- How would you design repository interfaces for aggregate roots in a DDD-influenced design?
- How does the repository pattern interact with EF Core's own change tracking and Unit of Work behavior?
- What are the trade-offs of returning `IQueryable<T>` from a repository versus a materialized list?

### Follow-up Questions
- Should a repository return domain entities or DTOs?
- How would you handle pagination inside a repository interface?

### Code Prediction
```csharp
public interface IOrderRepository
{
    IQueryable<Order> GetOrders();
}
```
What testability and abstraction problem does returning `IQueryable<Order>` here introduce, compared to returning `Task<IReadOnlyList<Order>>`?

## Practical Tasks

- Design a repository interface for an `Order` aggregate with exactly the operations the application actually needs (not a generic CRUD interface).
- Implement both an EF Core–backed and an in-memory fake implementation of the same interface.
- Write a unit test for a service that depends on `IOrderRepository`, using the fake implementation.
- Refactor a service that queries `DbContext` directly in five places into one that depends on a narrow repository interface.

## Readiness Criteria

Explain what a repository abstracts and why, argue both for and against introducing one in a given scenario, and design a repository interface driven by actual use cases rather than generic CRUD.

## References

### Microsoft Learn

- [Infrastructure persistence layer design](https://learn.microsoft.com/dotnet/architecture/microservices/microservice-ddd-cqrs-patterns/infrastructure-persistence-layer-design)
- [Testing without your database](https://learn.microsoft.com/ef/core/testing/)
