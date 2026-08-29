# Distributed Transactions, Eventual Consistency, and Idempotent Message Handling

## Definition

Once data spans multiple services (each with its own database, per the monolith-vs-microservices topic), a single ACID transaction (Module 9) spanning all of them isn't available — there's no single database to wrap a `BEGIN TRANSACTION` around. **Eventual consistency** accepts that different parts of a distributed system will be briefly inconsistent after a change, converging to a consistent state shortly after, typically via the messaging patterns from earlier in this module. **Idempotent message handling** — designing a message consumer so processing the same message twice has the same effect as processing it once — is what makes this safe under the at-least-once delivery guarantee most message brokers provide.

```
Traditional ACID transaction (one database): atomic, immediately consistent, all-or-nothing.
Distributed, eventually-consistent flow (multiple services): each step commits locally and
publishes an event; the overall operation becomes consistent gradually, across several steps,
each independently retriable.
```

## Alternatives & Trade-offs

A traditional distributed transaction protocol (two-phase commit) technically exists and provides immediate, ACID-like consistency across multiple databases, but at a steep cost: it requires all participants to be available and responsive simultaneously, holds locks across the whole operation, and scales poorly — this is why it's rarely used in practice for modern microservices. Eventual consistency via choreographed events or an orchestrated saga (below) sacrifices immediate consistency for availability and scalability, requiring the system (and the people debugging it) to reason about "this will become consistent shortly" rather than "this is consistent right now."

## How It Works

### The saga pattern — coordinating a multi-step distributed operation

```
1. Order Service creates an Order in "Pending" state, publishes "OrderCreated"
2. Inventory Service reserves stock, publishes "StockReserved" (or "StockReservationFailed")
3. Payment Service charges the customer, publishes "PaymentCompleted" (or "PaymentFailed")
4. Order Service, having heard both success events, marks the Order "Confirmed"

If any step fails, a COMPENSATING action undoes the previous steps
(e.g., "PaymentFailed" -> Inventory Service releases the reserved stock) —
there's no single rollback; each prior step must be explicitly, individually undone.
```

This compensating-action requirement is the real complexity distributed transactions introduce compared to a single database's automatic rollback — every step needs its own explicitly-designed "undo."

### Idempotent message handling — required because of at-least-once delivery (an earlier topic)

```csharp
public class ReserveStockHandler : IConsumer<OrderCreatedEvent>
{
    public async Task Consume(OrderCreatedEvent message)
    {
        if (await _processedMessages.ContainsAsync(message.MessageId)) return; // already handled — skip
        await _inventory.ReserveStockAsync(message.Sku, message.Quantity);
        await _processedMessages.MarkProcessedAsync(message.MessageId);
    }
}
```

Without this idempotency check, a message redelivered after a consumer crash (but after it had already succeeded, just before acknowledging) would reserve stock *twice* for the same order — exactly the kind of duplicate-side-effect risk Module 7's idempotency-key discussion addressed at the HTTP level, now applied to message consumption.

### Reading data during the inconsistency window — designing for it, not around it

```
Between step 2 and step 4 above, querying the Order's status might show "Pending" even though
stock has already been successfully reserved — this is the eventual-consistency window. UI/UX
and other consumers need to be designed with this window in mind (e.g., showing "Processing...")
rather than assuming every read reflects a fully settled, final state.
```

## Application

Design multi-step operations spanning multiple services as sagas — a sequence of local transactions, each publishing an event, with an explicit compensating action for every step that might need to be undone. Make every message consumer idempotent by design, tracking processed message IDs, given that at-least-once delivery is the default assumption. Design UI and downstream consumers to tolerate the eventual-consistency window rather than assuming immediate, always-settled state.

## Common Mistakes

- Attempting to use a traditional two-phase-commit distributed transaction across microservices instead of designing a saga with compensating actions, running into the availability and scaling problems that pattern is known for.
- Building message consumers that aren't idempotent, causing duplicate side effects (double charges, double stock reservations) under the at-least-once redelivery that most brokers guarantee.
- Not designing explicit compensating actions for every step of a saga, leaving a partially-completed operation with no way to cleanly undo the steps that already succeeded.
- Assuming a read immediately after a distributed operation reflects the fully consistent final state, when it might still be in the eventual-consistency window.

## Common Interview Questions

### Basic
- Why isn't a single ACID transaction available across multiple services' databases?
- What is eventual consistency?

### Intermediate
- What is the saga pattern, and what role do compensating actions play in it?
- Why is idempotent message handling necessary given at-least-once delivery?

### Advanced
- How would you design a saga for a multi-step operation (order + payment + inventory) including compensating actions for each possible failure point?
- How would you design a UI or downstream consumer to tolerate the eventual-consistency window without confusing or misleading the user?

### Follow-up Questions
- Does the saga pattern guarantee the same "all or nothing" atomicity as a single-database transaction?
- Can a saga's compensating action itself fail, and if so, what happens?

### Code Prediction
In the saga example above, if `PaymentService` charges the customer successfully but the "PaymentCompleted" event is lost due to a network issue before `OrderService` receives it, what state does the `Order` remain in, and what would need to happen for the system to eventually reach consistency?

## Practical Tasks

- Design a saga for a multi-step distributed operation, including the compensating action for each step's possible failure.
- Implement an idempotent message consumer using a processed-message-ID tracking mechanism.
- Design how a UI should represent the eventual-consistency window for a specific multi-step operation, rather than assuming immediate settled state.

## Readiness Criteria

Design sagas with proper compensating actions for multi-step distributed operations, implement idempotent message handling as a default discipline, and design for the eventual-consistency window rather than assuming immediate consistency.

## References

### Other

- [Microsoft: Saga distributed transactions pattern](https://learn.microsoft.com/azure/architecture/reference-architectures/saga/saga)
