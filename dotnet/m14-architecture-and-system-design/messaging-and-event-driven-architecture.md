# Messaging Fundamentals and Event-Driven Architecture

## Definition

**Messaging** is the mechanism underlying asynchronous communication (previous topic): a sender publishes a message to a broker (Kafka, RabbitMQ, Azure Service Bus), and one or more consumers process it independently, decoupled in time. **Event-driven architecture** is a specific style built on messaging where services react to *events* — facts about something that already happened ("OrderPlaced") — rather than being directly commanded to do something, letting new consumers react to existing events without the publisher knowing or caring who's listening.

```
Publisher: "OrderPlaced { OrderId: 42, CustomerId: 7 }" -> [message broker]
Consumer A (Inventory): reserves stock
Consumer B (Email):     sends confirmation email
Consumer C (Analytics): records the event
-- none of these consumers existing or not existing affects the publisher at all --
```

## Alternatives & Trade-offs

Direct synchronous calls (previous topic) are simpler when there's exactly one thing that needs to happen in response and it needs to happen immediately. Event-driven messaging shines when *multiple* things need to happen in response to one occurrence, and new reactions need to be addable later without touching the original publisher's code at all — at the cost of the eventual-consistency and message-handling complexity covered in this and later topics.

## How It Works

### Commands vs. events — a useful vocabulary distinction

```
Command: "ReserveStock" — an instruction, sent to a specific, known recipient, expecting it to act.
Event:   "OrderPlaced"  — a fact, broadcast to whoever happens to be listening, with no expectation
                           about who (or how many consumers) will react, or whether anyone will at all.
```

A command implies the sender knows and cares who handles it; an event doesn't — this distinction is why event-driven systems can add new consumers without changing the publisher, while command-based systems inherently couple the sender to a specific, known receiver.

### Queue vs. topic/pub-sub — one consumer group vs. many

```
Queue (point-to-point):  one message is consumed by exactly ONE consumer (even with multiple
                          workers competing, each message still goes to only one of them) —
                          good for distributing WORK across workers.
Topic (pub-sub):          one message is delivered to EVERY subscriber independently — good for
                          broadcasting an EVENT to multiple, independent reactions.
```

Choosing the wrong one for the use case either accidentally drops a message that multiple consumers needed (using a queue when pub-sub was needed) or duplicates work unnecessarily (using pub-sub for something that should only be handled once).

### At-least-once delivery — the default reality most brokers operate under

```
Most message brokers guarantee AT LEAST ONCE delivery, not EXACTLY once — a message can be
delivered more than once (after a consumer crash before acknowledging, for example). This is
exactly why idempotent message handling (a dedicated later topic) isn't optional — it's the
direct consequence of this delivery guarantee.
```

### A minimal publish/consume example

```csharp
// Publisher — has no idea who, if anyone, is listening
await _messageBus.PublishAsync(new OrderPlacedEvent(order.Id, order.CustomerId));

// Consumer — added independently, without ever touching OrderService's code
public class SendConfirmationEmailHandler : IConsumer<OrderPlacedEvent>
{
    public async Task Consume(OrderPlacedEvent message) => await _emailSender.SendAsync(message.CustomerId, "Order confirmed");
}
```

## Application

Use event-driven messaging when one occurrence should trigger multiple independent reactions, especially ones that might be added or changed over time without wanting to modify the original publisher. Use direct commands (synchronous or asynchronous) when there's a single, specific, known recipient expected to act. Choose queue vs. pub-sub delivery semantics based on whether a message represents distributable work or a broadcast fact.

## Common Mistakes

- Confusing commands and events, designing an "event" that's actually only ever meant for one specific consumer to act on (which is really a command in disguise).
- Choosing a point-to-point queue for something that multiple independent consumers each need to see, causing only one of them to ever receive a given message.
- Assuming exactly-once delivery, when most brokers guarantee at-least-once — building message handlers that aren't idempotent and break under a duplicate delivery.
- Adding event-driven messaging for a simple, single-recipient interaction where a direct call (synchronous or asynchronous) would be simpler and sufficient.

## Common Interview Questions

### Basic
- What's the difference between a command and an event?
- What's the difference between a message queue and a pub-sub topic?

### Intermediate
- Why does event-driven architecture make it easy to add new consumers without modifying the publisher?
- What does "at-least-once delivery" mean, and why does it matter for how consumers are designed?

### Advanced
- How would you decide, for a specific inter-service interaction, whether it should be modeled as a command or an event?
- How would you design a system where multiple independent consumers need to react to the same event, while a separate piece of work needs to be distributed across a pool of workers for load-balancing?

### Follow-up Questions
- Does event-driven architecture guarantee message ordering?
- Can a single system use both commands and events for different interactions?

### Code Prediction
An `OrderPlaced` event is published to a point-to-point queue, but both the `InventoryReservation` consumer and the `EmailConfirmation` consumer are competing to read from the same queue. What happens to the second consumer's chance of ever processing a given message, and what change (queue vs. topic) would fix this?

## Practical Tasks

- Design an event-driven flow where one event triggers at least two independent consumer reactions, without either consumer knowing about the other.
- Distinguish, for a set of hypothetical interactions, which should be modeled as commands and which as events.
- Design an idempotent consumer for an at-least-once-delivered message, anticipating the next topic's concern.

## Readiness Criteria

Distinguish commands from events and queues from topics precisely, design event-driven flows that let new consumers be added independently, and account for at-least-once delivery semantics in consumer design.

## References

### Other

- [Microsoft: Asynchronous message-based communication](https://learn.microsoft.com/dotnet/architecture/microservices/architect-microservice-container-applications/asynchronous-message-based-communication)
- [Martin Fowler: What do you mean by "Event-Driven"?](https://martinfowler.com/articles/201701-event-driven.html)
