# Observer Pattern

## Definition

Observer allows a subject to notify multiple dependents when its state or an event changes. .NET events are a common implementation; see [Events](../m02-csharp-language/events.md) for the language-level event model.

## Alternatives & Trade-offs

Observer decouples publishers from subscribers but creates hidden control flow and lifetime risks. A direct callback, message broker, or observable stream may better fit other requirements.

## How It Works

Observers subscribe to a subject and receive notifications. The publisher defines ordering, threading, error, and unsubscribe semantics.

```csharp
public sealed class OrderSubject
{
	public event EventHandler? Completed;

	public void Complete()
	{
		Completed?.Invoke(this, EventArgs.Empty);
	}
}
```

## Application

Use for in-process notifications, UI changes, domain events, and lifecycle callbacks.

## Common Mistakes

- Forgetting to unsubscribe.
- Assuming notification is asynchronous.
- Allowing one observer exception to break all notifications without a policy.

## Common Interview Questions

### Basic
- What is Observer?
- How do .NET events implement it?

### Intermediate
- How can subscriptions cause memory leaks?
- What should event arguments contain?

### Advanced
- How do reentrancy, ordering, and backpressure affect observers?
- When should in-process Observer become messaging?
- How can weak subscriptions trade complexity for lifetime safety?

### Follow-up Questions
- Who controls notification?
- What happens if an observer throws?

### Code Prediction
How many subscribers receive a notification after subscribing twice?

## Practical Tasks

- Implement an event-based notification with safe unsubscribe behavior.
- Define error and ordering behavior for multiple observers.

## Readiness Criteria

Explain publisher/subscriber ownership, event lifetime, notification semantics, and when to choose events, observables, or messaging.

## References

### Microsoft Learn

- [Events](https://learn.microsoft.com/dotnet/csharp/events-overview)
- [Event pattern](https://learn.microsoft.com/dotnet/csharp/event-pattern)
