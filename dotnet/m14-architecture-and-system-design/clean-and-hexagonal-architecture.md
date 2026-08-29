# Clean and Hexagonal Architecture (Conceptual Level)

## Definition

Clean Architecture and Hexagonal Architecture (also called Ports and Adapters) are two closely related, commonly-cited ways of visualizing the same dependency-direction idea from the previous topic: the domain sits at the center, knowing nothing about the outside world; everything external (databases, web frameworks, message queues, UI) connects to it through **ports** (interfaces the domain defines) implemented by **adapters** (infrastructure code).

```
         ┌─────────────────────┐
         │   Adapters (outer)   │  <- web framework, database, message queue clients
         │  ┌───────────────┐   │
         │  │  Ports (interfaces)│
         │  │ ┌───────────┐  │   │
         │  │ │  Domain    │  │   │  <- knows nothing about the outer rings
         │  │ └───────────┘  │   │
         │  └───────────────┘   │
         └─────────────────────┘
```

## Alternatives & Trade-offs

A simpler, conventional layered architecture (previous topic) achieves largely the same dependency-direction goal with less conceptual ceremony — "ports and adapters" and "layers" are describing overlapping ideas from different angles. Clean/Hexagonal architecture's specific value is making the *symmetry* of the domain's isolation more visible: the domain doesn't just depend downward on infrastructure, it's genuinely agnostic about *which* of several possible adapters (a REST API vs. a message queue trigger vs. a CLI) is driving it — a distinction that matters more for systems with several different "front doors" than for a single conventional web API.

## How It Works

### Ports — interfaces the domain defines for what it needs, or how it can be driven

```csharp
// An "outbound" port — something the domain needs FROM the outside world
public interface IOrderRepository { Task SaveAsync(Order order); }

// An "inbound" port — a use case the domain EXPOSES to the outside world, regardless of how it's triggered
public interface IPlaceOrderUseCase { Task ExecuteAsync(PlaceOrderRequest request); }
```

### Adapters — infrastructure implementations plugging into those ports

```csharp
// Outbound adapter: implements what the domain needs
public class SqlOrderRepository : IOrderRepository { /* ... */ }

// Multiple INBOUND adapters can drive the same use case, without the use case knowing or caring which one is active
public class OrdersController : ControllerBase // HTTP adapter
{
    public async Task<IActionResult> Post(PlaceOrderRequest request) => Ok(await _useCase.ExecuteAsync(request));
}

public class OrderQueueConsumer // message-queue adapter, driving the SAME use case
{
    public async Task HandleAsync(OrderMessage message) => await _useCase.ExecuteAsync(message.ToRequest());
}
```

This is the concrete payoff: `IPlaceOrderUseCase`'s implementation is completely unaware of whether it's being called from an HTTP controller, a message queue consumer, or a test — all three "adapters" plug into the same port.

### The "hexagon" is just a visualization choice, not a different idea

```
The hexagon shape itself has no special significance — it was chosen historically just to allow
drawing several ports/adapters around a central domain without implying a specific number or
a left-to-right "layers" reading. Clean Architecture's concentric-circles diagram expresses the
same core idea (dependencies point inward) with a different visual metaphor.
```

## Application

Reach for the explicit ports-and-adapters framing specifically when a system genuinely has multiple different ways of being driven (HTTP API, message consumer, scheduled job, CLI) and needs that symmetry made clear — it earns its conceptual overhead there. For a straightforward single-API service, a conventional layered architecture (previous topic) expresses the same dependency-direction discipline with less unfamiliar vocabulary, and is often the more practical choice.

## Common Mistakes

- Treating Clean/Hexagonal Architecture as a fundamentally different discipline from layered architecture, rather than recognizing they're expressing the same dependency-inversion idea through different visual metaphors.
- Introducing the full ports-and-adapters vocabulary and ceremony for a simple, single-entry-point web API where it adds conceptual overhead without a corresponding benefit.
- Defining a port so specifically tied to one particular adapter's needs that a second adapter (a message consumer instead of an HTTP controller) can't actually reuse it without modification.
- Confusing "port" (an interface) with "adapter" (an implementation) in discussion, when precise vocabulary is exactly the point of using this framing at all.

## Common Interview Questions

### Basic
- What do "ports" and "adapters" mean in Hexagonal Architecture?
- How does Clean/Hexagonal Architecture relate to the dependency-direction idea from layered architecture?

### Intermediate
- What's the practical benefit of designing a use case as a port that multiple different adapters (HTTP, message queue) can drive?
- Why is the "hexagon" shape itself not meaningful beyond being a diagram convention?

### Advanced
- When would the explicit ports-and-adapters framing provide real value over a simpler layered architecture, and when would it just be unnecessary ceremony?
- How would you design a use case's port so it's genuinely reusable across an HTTP adapter and a message-queue adapter without modification?

### Follow-up Questions
- Is Clean Architecture a completely different approach from Hexagonal Architecture, or largely the same idea?
- Does every application need multiple adapters to benefit from this framing?

### Code Prediction
Given `IPlaceOrderUseCase` driven by both `OrdersController` (HTTP) and `OrderQueueConsumer` (message queue) in the example above, if the use case's implementation changes internally (e.g., adds a new validation step), do either of the two adapters need to change?

## Practical Tasks

- Design a use case as an inbound port and implement it with two different adapters (e.g., an HTTP controller and a background job trigger).
- Identify a domain-owned outbound port and implement two different adapters for it (e.g., a SQL-backed and an in-memory implementation) for production and testing respectively.
- Compare a conventional layered-architecture description and a ports-and-adapters description of the same system, identifying where they express the same idea differently.

## Readiness Criteria

Explain ports and adapters precisely, design use cases that can be driven by multiple adapters without modification, and judge when this framing's conceptual overhead is or isn't worth adopting for a given system.

## References

### Other

- [Alistair Cockburn: Hexagonal Architecture (original description)](https://alistair.cockburn.us/hexagonal-architecture/)
- [Clean Architecture (Robert C. Martin's original blog post)](https://blog.cleancoder.com/uncle-bob/2012/08/13/the-clean-architecture.html)
