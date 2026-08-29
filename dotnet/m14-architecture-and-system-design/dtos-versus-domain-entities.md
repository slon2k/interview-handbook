# DTOs vs. Domain Entities

## Definition

A domain entity models business concepts and enforces business invariants (Module 4's encapsulation content). A DTO (Data Transfer Object) is a plain, often anemic data holder shaped specifically for crossing a boundary — an API request/response, a message queue payload — with no behavior and no invariants of its own. Keeping them distinct, rather than using one type for both purposes, is a recurring architectural decision this handbook has touched on from several angles already.

```csharp
// Domain entity: enforces invariants, has behavior
public class Order
{
    public IReadOnlyList<OrderItem> Items => _items;
    public void AddItem(OrderItem item) { /* validates business rules before allowing this */ }
}

// DTO: a flat, purpose-shaped data holder for one specific boundary-crossing use
public class CreateOrderRequest
{
    public int CustomerId { get; set; }
    public List<CreateOrderItemRequest> Items { get; set; } = new();
}
```

## Alternatives & Trade-offs

Using domain entities directly at every boundary (binding requests to them, returning them from APIs) saves the effort of defining and mapping to separate DTOs, but reopens exactly the mass-assignment risk from Module 12 and the over-fetching problem from Module 7 — the entity's full shape, including fields never meant to be client-visible or client-settable, becomes part of the API's implicit contract. Separate DTOs cost mapping code, but make each boundary's actual contract explicit and let the domain model evolve independently of any specific API shape.

## How It Works

### Why one shared type for both purposes causes problems

```csharp
public class Order
{
    public int Id { get; set; }
    public decimal InternalCostBasis { get; set; } // internal only — never meant to be exposed externally
    public bool IsFraudFlagged { get; set; }        // internal only
}

[HttpGet("{id}")]
public async Task<Order> Get(int id) => await _repository.GetByIdAsync(id); // leaks BOTH internal fields to any API consumer
```

Returning the domain entity directly makes every one of its fields part of the API's contract by accident, not by design — exactly the excessive-data-exposure risk named in Module 12's OWASP topic.

### The DTO boundary, done correctly

```csharp
public class OrderResponse // shaped deliberately for what external consumers should see
{
    public int Id { get; set; }
    public decimal Total { get; set; }
    public string Status { get; set; } = "";
}

[HttpGet("{id}")]
public async Task<OrderResponse> Get(int id)
{
    var order = await _repository.GetByIdAsync(id);
    return new OrderResponse { Id = order.Id, Total = order.Total, Status = order.Status.ToString() };
}
```

### The domain model stays free to evolve independently

```
If the domain's internal representation of Order changes (a new internal field, a refactored
invariant), OrderResponse's shape doesn't have to change at all, as long as the mapping logic
is updated — external API consumers are insulated from internal refactoring, which is exactly
the kind of stability Module 7's API-versioning discussion depends on being able to provide.
```

### Mapping — a real but manageable cost

```csharp
// Hand-written mapping, or a mapping library (AutoMapper, Mapster) for larger models
public static OrderResponse ToResponse(Order order) => new()
{
    Id = order.Id, Total = order.Total, Status = order.Status.ToString()
};
```

## Application

Define dedicated DTOs for every external boundary — API requests/responses, message queue payloads — distinct from domain entities, even when the shapes look similar today. Let the mapping layer absorb the cost of keeping the two in sync, so the domain model can evolve and the external contract can be deliberately versioned, independently of each other.

## Common Mistakes

- Binding API requests directly onto domain entities, reopening the mass-assignment vulnerability from Module 12.
- Returning domain entities directly from API responses, leaking internal-only fields and coupling the API's contract to the domain model's exact current shape.
- Assuming DTOs are unnecessary "boilerplate" for a small application, without considering how quickly that assumption breaks down once the domain model needs to change independently of the API contract.
- Letting a DTO accumulate business logic or validation that actually belongs in the domain layer, blurring the distinction this topic is about maintaining.

## Common Interview Questions

### Basic
- What's the difference between a DTO and a domain entity?
- Why shouldn't a domain entity typically be returned directly from an API endpoint?

### Intermediate
- How does using dedicated DTOs help API versioning and internal domain-model refactoring stay independent of each other?
- What risk does binding a request directly to a domain entity reopen, that this handbook covered in a different module?

### Advanced
- How would you design the mapping layer between domain entities and DTOs for a large model to keep it maintainable as both evolve?
- When, if ever, might using the same type for both purposes be an acceptable trade-off?

### Follow-up Questions
- Should DTOs ever contain business logic?
- Does using DTOs eliminate the need for request validation (Module 8)?

### Code Prediction
Given the `Order` entity with `InternalCostBasis` and `IsFraudFlagged` fields, if a `GET /orders/{id}` endpoint returns the entity directly (no DTO), what would an external API consumer be able to see that was never intended to be exposed?

## Practical Tasks

- Design dedicated request/response DTOs for a small API and implement the mapping to/from the domain entity.
- Identify an endpoint returning a domain entity directly and refactor it to use a DTO instead.
- Evolve a domain entity's internal structure while keeping an existing DTO's external shape unchanged, verifying only the mapping layer needs to change.

## Readiness Criteria

Explain why DTOs and domain entities should stay distinct, design appropriate DTOs for API boundaries, and connect this decision to the mass-assignment and data-exposure risks covered elsewhere in this handbook.

## References

### Other

- [Repository pattern (Module 4)](../m04-oop-design/repository.md)
- [Mass assignment / over-posting (Module 12)](../m12-application-security/mass-assignment-and-over-posting.md)
