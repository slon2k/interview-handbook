# Application Services

## Definition

An application service orchestrates one specific use case — coordinating domain objects, repositories, and external calls to accomplish it — without containing business rules itself. Business rules live in the domain layer (entities, value objects); the application service's job is sequencing and coordination: load this, call that, save the result, in the right order.

```csharp
public class PlaceOrderApplicationService
{
    private readonly IOrderRepository _orders;
    private readonly IInventoryService _inventory;

    public async Task ExecuteAsync(PlaceOrderRequest request)
    {
        var order = Order.Create(request.CustomerId); // business rules live INSIDE Order.Create, not here
        foreach (var item in request.Items)
        {
            await _inventory.ReserveStockAsync(item.Sku, item.Quantity); // coordination, not a business rule itself
            order.AddItem(item.Sku, item.Quantity); // Order enforces its own invariants
        }
        await _orders.SaveAsync(order);
    }
}
```

## Alternatives & Trade-offs

Putting orchestration logic directly in a controller action is less code upfront for a simple case, but mixes HTTP-specific concerns (Module 8) with use-case coordination, making the logic harder to reuse (from a message-queue consumer, say) and harder to test without spinning up the web framework. A dedicated application service separates "what does this use case need to do" from "how is it triggered," which is also exactly what makes the ports-and-adapters framing from an earlier topic possible — the application service is often the thing sitting behind the port.

## How It Works

### Where business rules go, versus where orchestration goes

```csharp
public class Order // domain entity — owns the actual business rule
{
    public void AddItem(string sku, int quantity)
    {
        if (quantity <= 0) throw new ArgumentException("Quantity must be positive"); // a business rule
        _items.Add(new OrderItem(sku, quantity));
    }
}

public class PlaceOrderApplicationService // orchestrates, doesn't decide business rules itself
{
    public async Task ExecuteAsync(PlaceOrderRequest request)
    {
        var order = Order.Create(request.CustomerId);
        foreach (var item in request.Items) order.AddItem(item.Sku, item.Quantity); // delegates the rule check
        await _orders.SaveAsync(order); // just sequencing: create, populate, save
    }
}
```

If the application service itself starts doing `if (quantity <= 0) throw ...`, that's a business rule leaking into the orchestration layer — a common and easy-to-miss architectural drift.

### One application service per use case, not one per entity

```
PlaceOrderApplicationService     — one specific use case
CancelOrderApplicationService    — a different use case, even though it touches the same Order entity
```

This differs from a generic "OrderService" that accumulates every operation related to `Order` regardless of use case — the use-case-per-service granularity keeps each one focused and keeps unrelated operations from becoming entangled in one large class (echoing Module 4's cohesion discussion).

### Application services as the natural home for cross-cutting orchestration concerns

```csharp
public async Task ExecuteAsync(PlaceOrderRequest request)
{
    using var transaction = await _unitOfWork.BeginTransactionAsync(); // transaction boundary lives here
    try
    {
        // ... orchestration ...
        await transaction.CommitAsync();
    }
    catch { await transaction.RollbackAsync(); throw; }
}
```

Transaction boundaries (Module 9/10), and often authorization checks that depend on the specific use case rather than a general role, naturally belong at the application-service level — they're about *this specific operation*, not a universal domain rule.

## Application

Design one application service (or one method with a clearly single-purpose responsibility) per use case, containing only orchestration — loading data, calling domain methods, coordinating with infrastructure, managing transaction boundaries — while business rules themselves live in domain entities and value objects.

## Common Mistakes

- Letting business rules leak into the application service (a validation check that belongs on the domain entity), blurring the boundary this layer exists to maintain.
- Building one large, generic service class per entity type instead of one focused service per use case, recreating the low-cohesion "manager class" problem from Module 4.
- Putting orchestration logic directly in a controller action, coupling use-case coordination to a specific way of being triggered.
- Not giving the application service a clear transaction boundary, leaving multi-step operations vulnerable to the partial-completion risk covered in Module 9/10.

## Common Interview Questions

### Basic
- What is an application service, and what's its responsibility compared to a domain entity?
- Why shouldn't business rules live in an application service?

### Intermediate
- Why is one application service per use case often preferred over one generic service per entity type?
- Where should a transaction boundary typically live in this layering?

### Advanced
- How would you refactor a controller action containing significant orchestration logic into a proper application service, and what would move to the domain layer versus stay in the application service?
- How does the application service relate to the "port" concept from the ports-and-adapters topic?

### Follow-up Questions
- Can an application service call another application service directly?
- Should authorization checks live in the application service or the domain layer?

### Code Prediction
Given `PlaceOrderApplicationService.ExecuteAsync` above, if a developer adds `if (request.Items.Count > 100) throw new InvalidOperationException("Too many items")` directly inside the application service instead of inside `Order.AddItem` or a domain-level check, what architectural principle does this violate, and why might it matter later?

## Practical Tasks

- Design an application service for a specific use case, ensuring all business rules live in domain entities rather than the service itself.
- Refactor a controller action containing orchestration logic into a dedicated application service.
- Identify a "God service" containing many unrelated operations for one entity type and split it into use-case-focused application services.

## Readiness Criteria

Design application services that orchestrate without containing business rules, place transaction boundaries appropriately, and structure services around use cases rather than entities.

## References

### Other

- [Domain-driven design vocabulary (this module)](domain-driven-design-vocabulary.md)
