# Basic CQRS Awareness

## Definition

CQRS (Command Query Responsibility Segregation) separates the model used to write data (commands) from the model used to read data (queries) — instead of one model serving both. At its simplest, this can just mean different DTOs/methods for reads versus writes on the same database; at its most involved, it means genuinely separate read and write data stores, kept in sync asynchronously. This is awareness-level knowledge: knowing what the pattern is and its trade-off, not defaulting to implementing the most elaborate version of it.

```
Without CQRS: one model (often the domain entity) used for both reading and updating data.
With CQRS (simple form): a rich domain model for writes/commands, and simpler, denormalized
                          read DTOs optimized for specific queries — same database.
With CQRS (full form):   entirely separate read and write databases, synchronized via events
                          (this module's messaging topic) — the read side can be optimized
                          completely independently (denormalized, differently indexed, even a
                          different database technology) from the write side's needs.
```

## Alternatives & Trade-offs

Using one model for both reads and writes is simpler — one set of entities, one database, no synchronization to manage — and is entirely sufficient for most applications, where read and write patterns aren't dramatically different. CQRS earns its cost specifically when read and write patterns genuinely diverge — reads need heavy denormalization/aggregation for dashboard-style queries while writes need strict normalization and validation — letting each side be optimized independently, at the cost of a synchronization mechanism (often eventual consistency, from the previous topic) between them.

## How It Works

### The simple form — different DTOs, same database, no real architectural split

```csharp
// Write side: the rich domain model enforcing invariants (Module 4)
public class Order { public void AddItem(...) { /* validates business rules */ } }

// Read side: a simple, denormalized query returning exactly what a specific view needs,
// bypassing the domain model's write-side ceremony entirely for a read that doesn't need it
public class OrderListQuery
{
    public async Task<List<OrderSummaryDto>> ExecuteAsync() =>
        await _context.Orders.Select(o => new OrderSummaryDto { Id = o.Id, Total = o.Total }).ToListAsync();
}
```

This lightweight form of CQRS — just not forcing every read through the same rich domain model used for writes — is common and low-cost, and doesn't require separate databases or eventual consistency at all.

### The full form — genuinely separate read and write stores

```
Write side:  a normalized relational database (Module 9), optimized for correctness and
             transactional integrity.
Read side:   a denormalized, heavily-indexed (or even a different technology, like a document
             store per the relational-vs-NoSQL topic) store, optimized purely for the specific
             queries the application's views actually need — kept in sync via events published
             whenever the write side changes.
```

```csharp
// Write side publishes an event on every change
public async Task PlaceOrderAsync(Order order)
{
    await _writeRepository.SaveAsync(order);
    await _eventPublisher.PublishAsync(new OrderPlacedEvent(order));
}

// A separate process consumes the event and updates the READ side's denormalized store
public class OrderReadModelUpdater : IConsumer<OrderPlacedEvent>
{
    public async Task Consume(OrderPlacedEvent e) => await _readStore.UpsertOrderSummaryAsync(e.Order);
}
```

This full form genuinely introduces eventual consistency between the write and read sides — a query immediately after a write might not yet reflect it, the same trade-off from the previous topic, now applied specifically to this pattern.

## Application

Reach for the simple form of CQRS (different read/write models, same database) freely — it's low-cost and often just good practice once read and write shapes diverge even slightly. Reach for the full form (separate read/write stores with event-based synchronization) specifically when read and write patterns diverge dramatically and the resulting eventual-consistency cost is genuinely acceptable for the use case — not by default, echoing this module's later topic on avoiding unnecessary architecture.

## Common Mistakes

- Jumping straight to the full CQRS form (separate databases, event synchronization) for an application whose read and write patterns don't actually diverge enough to justify it.
- Treating CQRS as requiring separate databases, when the simple form (different DTOs on the same database) is a legitimate, much lower-cost application of the same underlying idea.
- Not accounting for the eventual-consistency window the full form introduces, causing confusion when a read immediately after a write doesn't yet reflect it.
- Confusing CQRS with event sourcing — they're often used together but are genuinely separate concepts; CQRS is about separating read/write models, not about how write-side state is stored (as a sequence of events vs. current state).

## Common Interview Questions

### Basic
- What does CQRS stand for, and what does it separate?
- What's the difference between the "simple" and "full" forms of CQRS?

### Intermediate
- When would the simple form of CQRS (different read/write DTOs on one database) be sufficient, without needing separate databases?
- What cost does the full form of CQRS introduce that the simple form doesn't?

### Advanced
- How would you decide whether a specific application's read and write patterns diverge enough to justify the full CQRS form's eventual-consistency cost?
- How does CQRS relate to (and differ from) event sourcing?

### Follow-up Questions
- Does CQRS require using different database technologies for reads and writes?
- Can the simple form of CQRS be introduced without any architectural risk, given it doesn't involve eventual consistency?

### Code Prediction
Given the full-form CQRS example above, if a client immediately queries the read-side `OrderSummaryDto` right after successfully placing an order, before `OrderReadModelUpdater` has processed the corresponding event, what would that query likely return?

## Practical Tasks

- Implement the simple form of CQRS for an existing feature — separate read DTOs from the write-side domain model, on the same database.
- Design the full form of CQRS for a specific use case where read and write patterns genuinely diverge, including the event-based synchronization mechanism.
- Explain, for a given application, why the full CQRS form would or wouldn't be justified compared to the simple form.

## Readiness Criteria

Explain the difference between CQRS's simple and full forms, apply the simple form freely where it fits, and justify the full form's eventual-consistency cost only when read/write divergence genuinely warrants it.

## References

### Other

- [Microsoft: CQRS pattern](https://learn.microsoft.com/azure/architecture/patterns/cqrs)
- [Martin Fowler: CQRS](https://martinfowler.com/bliki/CQRS.html)
