# Layered Architecture, Separation of Domain/Infrastructure, and Dependency Direction

## Definition

Layered architecture organizes a codebase into horizontal slices — typically presentation, application/business logic, and infrastructure/data access — each depending only on the layer(s) below it. **Separation of domain and infrastructure** means business rules shouldn't know or care how they're persisted, transported, or displayed. **Dependency direction** is the specific rule that makes this real: dependencies should point *inward*, toward business logic, never the reverse — infrastructure depends on the domain's abstractions, not the other way around.

```
Presentation  ->  Application  ->  Domain
                                     ^
Infrastructure ----------------------┘   (infrastructure depends ON the domain's interfaces, not vice versa)
```

## Alternatives & Trade-offs

A naive layering where the domain directly references infrastructure types (a business rule calling `SqlConnection` directly) is simpler to write initially, but couples business logic to a specific technology choice and makes it hard to test without a real database. Inverting the dependency — the domain defines an interface, infrastructure implements it — costs an extra abstraction but keeps business rules technology-agnostic and independently testable, which is the whole practical payoff Module 4's dependency-inversion content was building toward; this topic is that same idea applied at the architecture-wide scale rather than a single class.

## How It Works

### The wrong direction — domain depends on infrastructure directly

```csharp
public class OrderService // "business logic," but tightly coupled to a specific persistence technology
{
    public void PlaceOrder(Order order)
    {
        using var connection = new SqlConnection("..."); // domain logic now knows about SQL Server specifically
        // ...
    }
}
```

### The correct direction — infrastructure depends on an abstraction the domain owns

```csharp
// Domain layer defines what it needs, with no knowledge of how it's implemented
public interface IOrderRepository { Task SaveAsync(Order order); }

public class OrderService // depends only on the abstraction
{
    private readonly IOrderRepository _repository;
    public OrderService(IOrderRepository repository) => _repository = repository;
}

// Infrastructure layer implements the domain's interface — infrastructure depends on domain, not the reverse
public class SqlOrderRepository : IOrderRepository
{
    public Task SaveAsync(Order order) { /* SQL Server specifics live here, isolated */ }
}
```

The interface `IOrderRepository` is *owned* by the domain/application layer even though it's *implemented* by infrastructure — this inversion is what "dependency direction" actually refers to, and it's the architecture-scale version of Module 4's Dependency Inversion Principle.

### Why this matters beyond just testability

```
If SqlOrderRepository is replaced by MongoOrderRepository (a completely different persistence
technology), OrderService and every other domain/application class needs ZERO changes —
they only ever depended on IOrderRepository, which hasn't changed at all.
```

This is the practical payoff: the domain layer, which contains the most valuable and most expensive-to-get-wrong logic, stays stable and portable even as infrastructure choices change around it.

### Presentation layer's place in the direction

```
Controllers/API layer -> depends on -> Application services -> depends on -> Domain
```

The presentation layer also depends inward — it calls application services, which orchestrate domain logic; it never contains business rules itself (this connects to Module 8's controller/middleware content, which is deliberately about request handling, not business logic).

## Application

Structure new backend services with the domain/business-logic layer defining the interfaces it needs, and infrastructure implementing those interfaces — never the domain directly referencing a specific database, message queue, or external API client type. Use this as the default lens for reviewing where a new piece of logic belongs: does it represent a business rule (domain), an orchestration of business rules for one use case (application), or a technical detail of how something is stored/transmitted (infrastructure)?

## Common Mistakes

- Letting domain/business logic directly reference infrastructure types (a specific ORM's `DbContext`, a specific message queue client), coupling business rules to a technology choice.
- Defining the repository interface in the infrastructure layer instead of the domain/application layer, inverting the ownership even though the dependency direction looks superficially similar.
- Treating "layered architecture" as just a folder-naming convention without actually enforcing the dependency direction at compile time (e.g., via project references that physically can't point the wrong way).
- Putting business logic in the presentation layer (a controller action) because it was the most convenient place to write it in the moment.

## Common Interview Questions

### Basic
- What are the typical layers in a layered architecture?
- What does "dependency direction should point inward" mean in practice?

### Intermediate
- Why should a repository interface be owned by the domain/application layer rather than the infrastructure layer?
- What's the practical benefit of separating domain logic from infrastructure, beyond just "clean code"?

### Advanced
- How would you enforce dependency direction at the project/compilation level, not just as a convention developers might violate?
- How does this pattern relate to Module 4's Dependency Inversion Principle, applied at the whole-application scale rather than a single class?

### Follow-up Questions
- Can the application layer depend on the infrastructure layer directly, or must it always go through a domain-owned abstraction?
- Does layered architecture require physically separate projects/assemblies, or can it be achieved within a single project via convention?

### Code Prediction
Given `SqlOrderRepository : IOrderRepository` where `IOrderRepository` is defined in the domain project and `SqlOrderRepository` is defined in an infrastructure project referencing the domain project, what would happen if a developer tried to make the domain project reference the infrastructure project too? Why is that a design smell?

## Practical Tasks

- Refactor a service directly referencing a specific database client into one depending on a domain-owned repository interface.
- Design project/assembly boundaries for a layered application that physically prevent the domain layer from referencing infrastructure.
- Identify business logic accidentally living in a controller and relocate it to the appropriate application/domain layer.

## Readiness Criteria

Explain layered architecture and dependency direction precisely, structure new code so infrastructure depends on domain-owned abstractions rather than the reverse, and recognize violations of this direction in existing code.

## References

### Other

- [Microsoft: Common web application architectures](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures)
