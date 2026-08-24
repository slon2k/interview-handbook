# Separation of Concerns

## Definition

Separation of Concerns (SoC) means dividing a system so each part addresses one distinct concern — one aspect of behavior, such as persistence, business rules, or presentation — with minimal overlap between parts. It's the architectural-scale version of the idea behind SRP.

```csharp
// Concerns tangled together
public class OrderController
{
    public IActionResult Create(OrderDto dto)
    {
        if (dto.Quantity <= 0) return BadRequest(); // validation concern
        var total = dto.Quantity * dto.UnitPrice * 0.9m; // business logic concern
        using var conn = new SqlConnection("..."); // persistence concern
        return Ok();
    }
}
```

## Alternatives & Trade-offs

Separating concerns into layers (presentation, application/business logic, persistence) makes each part independently testable and replaceable, at the cost of more files and more indirection to trace a single request end-to-end. For a genuinely trivial script or prototype, collapsing concerns into one file is a reasonable, temporary trade-off — the risk is that "temporary" prototypes become permanent, tangled production code.

## How It Works

```csharp
public class CreateOrderValidator
{
    public bool IsValid(OrderDto dto) => dto.Quantity > 0;
}

public class OrderPricingService
{
    public decimal CalculateTotal(OrderDto dto) => dto.Quantity * dto.UnitPrice * 0.9m;
}

public interface IOrderRepository { Task SaveAsync(Order order); }

public class OrderController
{
    private readonly CreateOrderValidator _validator;
    private readonly OrderPricingService _pricing;
    private readonly IOrderRepository _repository;

    public OrderController(CreateOrderValidator validator, OrderPricingService pricing, IOrderRepository repository)
        => (_validator, _pricing, _repository) = (validator, pricing, repository);

    public async Task<IActionResult> Create(OrderDto dto)
    {
        if (!_validator.IsValid(dto)) return BadRequest();
        var total = _pricing.CalculateTotal(dto);
        await _repository.SaveAsync(new Order(dto, total));
        return Ok();
    }
}
```

The controller now only orchestrates; validation, pricing, and persistence each live where they can be tested and changed independently.

## Application

Apply SoC at every scale: within a method (don't mix validation, calculation, and I/O in one block), within a class (SRP), and across a codebase (layered/clean architecture separating domain, application, and infrastructure).

## Common Mistakes

- Putting business logic directly inside controllers or minimal API endpoint handlers, making it untestable without spinning up the whole HTTP pipeline.
- Letting persistence concerns (SQL, EF Core specifics) leak into domain or business logic classes.
- Over-separating trivial logic into many tiny classes/layers where the concerns genuinely don't need independent variation, adding navigation overhead for no real benefit.

## Common Interview Questions

### Basic
- What is Separation of Concerns, and how does it relate to SRP?
- Why shouldn't business logic live directly in a controller?

### Intermediate
- What concerns are typically separated in a layered ASP.NET Core application?
- How does SoC improve testability?

### Advanced
- How does SoC show up differently at the method level versus the architectural level?
- How would you decide the right granularity of separation for a given feature, avoiding both a tangled God-controller and needless over-layering?

### Follow-up Questions
- Is putting validation logic in the controller ever acceptable?
- How does SoC relate to the layers in Clean/Hexagonal Architecture?

### Code Prediction
Given the tangled `OrderController` example, if the pricing discount rate needs to change from 10% to 15%, how many places must be edited before and after the refactor shown in "How It Works"?

## Practical Tasks

- Refactor a controller mixing validation, business logic, and persistence into separated, independently testable classes.
- Write unit tests for the extracted `OrderPricingService` without touching HTTP or the database.
- Identify a case of over-separation (too many trivial layers) in a hypothetical codebase and simplify it.

## Readiness Criteria

Identify tangled concerns in a controller or service, refactor them into independently testable units, and judge the right granularity of separation for a given feature's actual complexity.

## References

### Microsoft Learn

- [Common architectural principles](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/architectural-principles)
- [Common web application architectures](https://learn.microsoft.com/dotnet/architecture/modern-web-apps-azure/common-web-application-architectures)
