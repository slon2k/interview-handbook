# Observer Pattern

## Definition

The Observer pattern lets an object (the subject) notify a list of dependents (observers) automatically when its state changes, without the subject needing to know concrete details about them. C# has first-class support for this via `event` and delegates.

```csharp
public sealed class OrderProcessor
{
    public event EventHandler<OrderShippedEventArgs>? OrderShipped;

    public void ShipOrder(Order order)
    {
        // ... shipping logic ...
        OrderShipped?.Invoke(this, new OrderShippedEventArgs(order));
    }
}
```

## Alternatives & Trade-offs

Observer decouples the subject from its dependents — the subject doesn't need to know who's listening or why. This is ideal for in-process notifications (UI updates, domain event handling) but doesn't scale to distributed notification across services; that's the job of a message broker (Kafka, RabbitMQ, Azure Service Bus), which is Observer's conceptual cousin at a different scale. `IObservable<T>`/`IObserver<T>` (Reactive Extensions) formalize the same idea for streams of values.

## How It Works

### Classic event-based observer

```csharp
public sealed class OrderShippedEventArgs : EventArgs
{
    public Order Order { get; }
    public OrderShippedEventArgs(Order order) => Order = order;
}

public sealed class EmailNotifier
{
    public void Subscribe(OrderProcessor processor) =>
        processor.OrderShipped += (sender, e) => Console.WriteLine($"Email: order {e.Order.Id} shipped");
}

var processor = new OrderProcessor();
new EmailNotifier().Subscribe(processor);
processor.ShipOrder(order); // triggers the subscribed handler
```

`OrderProcessor` has no reference to `EmailNotifier` — it only raises an event that any number of subscribers can listen to.

### Multiple independent observers

```csharp
processor.OrderShipped += (s, e) => auditLog.Record(e.Order);
processor.OrderShipped += (s, e) => analytics.Track("order_shipped", e.Order);
```

Both handlers run when `OrderShipped` is raised, and neither knows about the other.

### A common pitfall: memory leaks from forgotten unsubscription

```csharp
processor.OrderShipped += handler.OnOrderShipped;
// if `handler` goes out of scope but never does processor.OrderShipped -= handler.OnOrderShipped,
// `processor` keeps `handler` alive for as long as `processor` itself lives
```

## Application

Use Observer for in-process, one-to-many notifications: domain events within an application, UI state changes, logging/auditing hooks that shouldn't be coupled to the core operation that triggered them.

## Common Mistakes

- Forgetting to unsubscribe long-lived event handlers, causing the subject to keep observers alive longer than intended (a common source of memory leaks).
- Using `event` for cross-service or cross-process notification instead of a message broker, when the observers actually need to run in a different process.
- Letting event handlers throw unhandled exceptions, which by default stops later subscribers in the invocation list from running.
- Putting heavy synchronous work directly in an event handler, blocking the subject's thread.

## Common Interview Questions

### Basic
- What is the Observer pattern, and how does C# support it natively?
- What's the difference between `event` and a plain delegate field?

### Intermediate
- How can forgetting to unsubscribe from an event cause a memory leak?
- What happens if one subscriber throws an exception during event invocation?

### Advanced
- How does the Observer pattern relate to message brokers and pub/sub systems at a distributed-systems scale?
- How would you make an observer-based notification system asynchronous without blocking the subject?

### Follow-up Questions
- What is the difference between `IObservable<T>`/`IObserver<T>` and plain C# events?
- Why is `event` preferred over a public delegate field for this pattern?

### Code Prediction
```csharp
public class Publisher { public event Action? Notify; }
var pub = new Publisher();
pub.Notify += () => Console.WriteLine("A");
pub.Notify += () => Console.WriteLine("B");
pub.Notify?.Invoke();
```
What is printed, and in what order? What happens if the "A" handler throws?

## Practical Tasks

- Implement an `OrderProcessor` that raises an `OrderShipped` event, with two independent subscribers (email, audit log).
- Reproduce a subscriber memory leak scenario and fix it by properly unsubscribing.
- Convert a synchronous event handler doing slow I/O into an asynchronous notification flow.

## Readiness Criteria

Explain events as an implementation of Observer, identify and fix subscription memory leaks, and reason about exception and ordering behavior across multiple subscribers.

## References

### Microsoft Learn

- [Events (C# programming guide)](https://learn.microsoft.com/dotnet/csharp/programming-guide/events/)
- [Observer design pattern](https://learn.microsoft.com/dotnet/standard/events/observer-design-pattern)
