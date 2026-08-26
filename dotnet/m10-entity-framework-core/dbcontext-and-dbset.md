# DbContext and DbSet

## Definition

`DbContext` represents a session with the database — it tracks entities, translates LINQ queries into SQL, and coordinates saving changes. `DbSet<T>` represents a queryable, updatable collection of entities of type `T`, mapped to a table (or view), exposed as a property on a `DbContext` subclass.

```csharp
public class AppDbContext : DbContext
{
    public DbSet<Order> Orders => Set<Order>();
    public DbSet<Customer> Customers => Set<Customer>();

    protected override void OnConfiguring(DbContextOptionsBuilder options) =>
        options.UseSqlServer("connection string");
}
```

## Alternatives & Trade-offs

EF Core's `DbContext`/`DbSet` model gives you a single, coherent unit of work — query and modify multiple entity types, then persist everything together with one `SaveChanges()` call, with automatic SQL generation from LINQ. Compared to a lighter-weight tool like Dapper (hand-written SQL, results mapped to objects, no change tracking), EF Core trades some control and raw performance for significantly less boilerplate, automatic change tracking, and migrations — the right choice depends on whether the app needs rich object-graph persistence or mostly needs fast, simple query-and-map operations.

## How It Works

### `DbContext` as a unit of work

```csharp
using var context = new AppDbContext();
var order = await context.Orders.FindAsync(42);
order.Status = "Shipped";
context.Customers.Add(new Customer { Name = "New Customer" });
await context.SaveChangesAsync(); // both changes persisted together, in one transaction by default
```

A single `SaveChangesAsync()` call persists every tracked change across every `DbSet` on the context — this is what makes `DbContext` a genuine unit of work, not just a query tool.

### Registering `DbContext` with dependency injection

```csharp
builder.Services.AddDbContext<AppDbContext>(options =>
    options.UseSqlServer(builder.Configuration.GetConnectionString("Default")));
```

`AddDbContext` registers the context as Scoped by default — one instance per request, shared across everything that needs it within that request (see `dbcontext-lifetime-and-pooling.md`).

### `DbSet<T>` is both a query root and a write target

```csharp
// Querying — DbSet<T> implements IQueryable<T>
var pendingOrders = await context.Orders.Where(o => o.Status == "Pending").ToListAsync();

// Writing — Add/Remove/Update stage changes; nothing hits the database until SaveChanges
context.Orders.Add(new Order { CustomerId = 7 });
context.Orders.Remove(order);
```

### `OnModelCreating` — where entity configuration lives

```csharp
protected override void OnModelCreating(ModelBuilder modelBuilder)
{
    modelBuilder.Entity<Order>().HasIndex(o => o.CustomerId);
    // full entity configuration is covered in entity-configuration.md
}
```

## Application

Design one `DbContext` per logical unit of work (usually per bounded area of the application, not necessarily one per entire application), inject it via DI with the default Scoped lifetime, and use it to compose multi-entity changes that must be saved together atomically via a single `SaveChanges()` call.

## Common Mistakes

- Treating `DbContext` as a long-lived, shared object instead of a short-lived unit of work scoped to one request/operation, leading to stale tracked data and the captive-dependency issues covered in Module 8.
- Calling `SaveChanges()` after every single small change instead of batching related changes into one call, losing the atomicity benefit of a single unit of work.
- Querying through `DbSet<T>` without realizing it's `IQueryable<T>` — building up a query with LINQ operators before it's actually executed (see `query-translation-and-iqueryable.md`).
- Instantiating `DbContext` directly with `new AppDbContext()` in application code instead of resolving it via DI, losing the configured connection string, lifetime management, and pooling.

## Common Interview Questions

### Basic
- What is `DbContext`, and what is `DbSet<T>`?
- What does calling `SaveChanges()` actually do?

### Intermediate
- Why is `DbContext` described as a "unit of work"?
- What's the default lifetime of a `DbContext` registered via `AddDbContext`, and why?

### Advanced
- How would you design `DbContext` boundaries for an application with several distinct business domains — one big context, or several smaller ones?
- What happens internally between calling `context.Orders.Add(...)` and calling `SaveChangesAsync()`?

### Follow-up Questions
- Does querying a `DbSet<T>` immediately hit the database?
- Can a single `SaveChanges()` call span changes to multiple different `DbSet<T>` properties?

### Code Prediction
```csharp
context.Orders.Add(new Order { CustomerId = 7 });
context.Customers.Add(new Customer { Name = "Test" });
// no SaveChanges() call yet
```
At this point, has anything been written to the database? What would happen if the application crashed right now?

## Practical Tasks

- Design a `DbContext` with two related `DbSet<T>` properties and perform a multi-entity change persisted in one `SaveChanges()` call.
- Register a `DbContext` via DI and observe its default Scoped lifetime across two requests.
- Explain, for a given application with distinct business domains, whether one `DbContext` or several would be more appropriate.

## Readiness Criteria

Explain `DbContext` as a unit of work, use `DbSet<T>` for both querying and staging changes, and register `DbContext` correctly via DI rather than manual instantiation.

## References

### Microsoft Learn

- [DbContext class](https://learn.microsoft.com/dotnet/api/microsoft.entityframeworkcore.dbcontext)
- [Configuring a DbContext](https://learn.microsoft.com/ef/core/dbcontext-configuration/)
