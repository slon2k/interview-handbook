# Events

## Definition

An event is a publisher-controlled notification mechanism built on delegates. Subscribers can add and remove handlers, but only the declaring type can raise the event.

```csharp
public event EventHandler? Updated;

protected virtual void OnUpdated() => Updated?.Invoke(this, EventArgs.Empty);
```

## Alternatives & Trade-offs

Events are useful for one-to-many notifications and decouple a publisher from subscribers. Direct callbacks are simpler for one consumer. An observable stream or message broker is better for composition, buffering, replay, or cross-process delivery.

Events can create hidden control flow and lifetime problems. Document threading, ordering, and error behavior for important events.

## How It Works

The compiler exposes add/remove accessors backed by a delegate. `+=` subscribes and `-=` unsubscribes. The publisher invokes a snapshot of the delegate to reduce races during notification.

```csharp
var handlers = Updated;
handlers?.Invoke(this, EventArgs.Empty);
```

A subscriber holds a reference from the publisher, so failing to unsubscribe can keep the subscriber alive.

## Application

- Notify UI or domain observers of state changes.
- Implement framework lifecycle callbacks.
- Publish completed, failed, or changed notifications.
- Use `EventHandler<TEventArgs>` for conventional .NET events.

## Common Mistakes

- Exposing a public delegate field instead of an event.
- Forgetting to unsubscribe long-lived subscriptions.
- Raising events from outside the declaring type.
- Assuming handlers execute asynchronously.
- Allowing one handler exception to silently stop notification without a policy.
- Performing expensive work synchronously in an event handler.

## Common Interview Questions

### Basic

- What is an event?
- How is an event different from a delegate?
- Who can raise an event?
- How do you subscribe and unsubscribe?

### Intermediate

- Why use `EventHandler<TEventArgs>`?
- How can events cause memory leaks?
- Should event invocation be thread-safe?
- What is the standard sender and event-arguments pattern?

### Advanced

- How do custom event accessors work?
- How would you make event subscription safe under concurrency?
- What happens if one event handler throws?
- How can event storms affect throughput and latency?
- When should an event be replaced by an observable or message queue?
- How do weak-event patterns trade complexity for lifetime safety?
- How should event ordering and reentrancy be specified?
- What are the versioning implications of event argument types?
- How would you test subscription, unsubscription, and handler failures?
- How do static events create process-wide lifetime risks?

### Follow-up Questions

- Can a derived class raise a base-class event?
- Why is `event` safer than a public delegate?
- What happens when the same handler is added twice?
- Are event handlers invoked synchronously by default?
- Why should event publishers avoid exposing mutable event arguments?

### Code Prediction

How many lines are printed?

```csharp
publisher.Changed += Handler;
publisher.Changed += Handler;
publisher.RaiseChanged();
```

## Practical Tasks

### Publisher

Implement a publisher with a typed event and a protected virtual `OnChanged` method.

### Lifetime Review

Create a long-lived publisher and short-lived subscriber, then demonstrate and fix the retention problem.

### Error Policy

Define how a publisher handles exceptions from multiple subscribers and test the chosen policy.

## Readiness Criteria

You should be able to implement conventional events, explain publisher/subscriber responsibilities, reason about lifetimes and threading, and choose an alternative when event semantics become insufficient.

## References

### Microsoft Learn

- [Events](https://learn.microsoft.com/dotnet/csharp/events-overview)
- [Event pattern](https://learn.microsoft.com/dotnet/csharp/event-pattern)
- [How to publish events](https://learn.microsoft.com/dotnet/csharp/programming-guide/events/how-to-publish-events)
- [How to subscribe to and unsubscribe from events](https://learn.microsoft.com/dotnet/csharp/programming-guide/events/how-to-subscribe-to-and-unsubscribe-from-events)
