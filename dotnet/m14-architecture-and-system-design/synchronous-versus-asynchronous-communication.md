# Synchronous vs. Asynchronous Communication (System Level)

## Definition

This is the system-design-scale version of the sync/async distinction from Module 6 — not "does this method use `await`," but "when service A needs something from service B, does A wait for an immediate response (synchronous — REST, gRPC), or does A hand off a message and continue without waiting (asynchronous — a queue or event, next topic)?" The choice shapes coupling, failure behavior, and latency characteristics across the whole system.

```
Synchronous:  Order Service --(HTTP call, waits)--> Inventory Service --(response)--> Order Service continues
Asynchronous: Order Service --(publishes "OrderPlaced" event)--> [returns immediately]
              Inventory Service (whenever it gets to it) --(consumes the event)--> reserves stock
```

## Alternatives & Trade-offs

Synchronous communication is simpler to reason about (a clear request/response, easy to trace, immediate error feedback) but couples the caller's availability to the callee's — if Inventory Service is down or slow, Order Service's request blocks or fails too, and the coupling compounds across chains of synchronous calls. Asynchronous communication decouples availability (Order Service can proceed even if Inventory Service is temporarily down, since the message just waits in a queue) at the cost of giving up the immediate response and requiring the whole system to reason about eventual consistency (a later topic) instead of an immediate, consistent result.

## How It Works

### Synchronous — REST/gRPC (Module 7's territory, at system scale)

```csharp
public async Task PlaceOrderAsync(Order order)
{
    var reserved = await _inventoryClient.ReserveStockAsync(order.Sku, order.Quantity); // blocks until Inventory responds
    if (!reserved) throw new InsufficientStockException();
    // ...
}
```

If `InventoryService` is down, this call fails immediately and visibly — which is good for a use case genuinely needing an immediate answer ("can I actually buy this right now"), but means `OrderService`'s own availability is now coupled to `InventoryService`'s.

### Asynchronous — publish and move on (the next topic's territory)

```csharp
public async Task PlaceOrderAsync(Order order)
{
    await _eventPublisher.PublishAsync(new OrderPlacedEvent(order.Id, order.Sku, order.Quantity));
    // returns immediately — Order Service doesn't wait for Inventory Service to actually process this at all
}
```

`OrderService` stays available and responsive even if `InventoryService` is completely down — the event just accumulates in a queue until `InventoryService` recovers and catches up. The trade-off: there's no immediate confirmation that stock was actually reserved successfully.

### Chained synchronous calls compound the coupling problem

```
Client -> OrderService -(sync)-> InventoryService -(sync)-> WarehouseService
```

If any one service in this chain is slow or down, the failure/latency propagates all the way back to the client — a chain of synchronous calls is only as reliable and as fast as its weakest link, multiplied across every hop.

### Choosing based on the actual requirement, not habit

```
Needs an immediate answer to proceed (checking real-time stock before showing "add to cart")?
  -> synchronous is usually the right, simpler choice.

Can tolerate "this will be processed soon" (sending a confirmation email, updating an analytics
dashboard, reserving stock as a background effect of order placement)?
  -> asynchronous decouples availability and is often the better choice.
```

## Application

Choose synchronous communication when a caller genuinely needs an immediate result to proceed, and accept the resulting availability coupling as the price of that requirement. Choose asynchronous communication when the receiving side's immediate availability isn't actually required for the caller to make progress, accepting eventual consistency in exchange for decoupled availability and resilience to the receiver being temporarily down.

## Common Mistakes

- Defaulting to synchronous calls for every inter-service interaction out of habit, creating long chains of coupled availability that make the whole system only as reliable as its least reliable synchronous dependency.
- Choosing asynchronous communication for a use case that genuinely needs an immediate, consistent answer, then having to bolt on awkward polling or callback logic to simulate synchronicity anyway.
- Not considering that a synchronous call chain's total latency is the sum of every hop's latency, compounding across a long chain.
- Assuming "asynchronous" always means better resilience without accounting for the eventual-consistency and message-handling complexity it introduces (covered in later topics).

## Common Interview Questions

### Basic
- What's the difference between synchronous and asynchronous communication at the system-design level, as opposed to the async/await level from Module 6?
- Why does synchronous communication couple two services' availability together?

### Intermediate
- Why does a chain of synchronous calls compound both latency and availability risk?
- What use case characteristics suggest synchronous communication is the right choice despite its coupling cost?

### Advanced
- How would you decide, for a specific cross-service interaction, whether the immediate-consistency benefit of synchronous communication is worth its availability coupling?
- How does REST/gRPC (Module 7) relate to this system-level synchronous choice, versus messaging/events (next topic) relating to the asynchronous choice?

### Follow-up Questions
- Can a system use synchronous communication for some interactions and asynchronous for others?
- Does asynchronous communication eliminate the need to handle failure at all, or just change its shape?

### Code Prediction
In the chained synchronous example (`Client -> OrderService -> InventoryService -> WarehouseService`), if `WarehouseService` takes 3 seconds to respond due to its own internal load, what's the minimum possible total latency the client experiences, even if `OrderService` and `InventoryService` add zero overhead of their own?

## Practical Tasks

- Design a system with at least one genuinely synchronous interaction (needs immediate confirmation) and one genuinely asynchronous interaction (can tolerate eventual processing), justifying each choice.
- Trace the compounding latency and availability risk of a three-hop synchronous call chain, and redesign one hop to be asynchronous where appropriate.
- Identify a use case in a hypothetical system currently using synchronous calls that would be a better fit for asynchronous communication, and justify the change.

## Readiness Criteria

Choose between synchronous and asynchronous communication based on the actual requirement (immediate consistency vs. tolerable eventual processing) rather than habit, and reason about the compounding effect of chained synchronous calls.

## References

### Other

- [Microsoft: Asynchronous message-based communication](https://learn.microsoft.com/dotnet/architecture/microservices/architect-microservice-container-applications/asynchronous-message-based-communication)
