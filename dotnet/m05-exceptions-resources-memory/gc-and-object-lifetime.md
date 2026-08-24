# Garbage Collection and Object Lifetime

## Definition

The GC reclaims managed objects that are unreachable from roots. Roots include live local references, static fields, active threads, runtime handles, and references from other reachable objects.

## Alternatives & Trade-offs

Let the GC manage ordinary objects. Investigate allocation and retention before calling collection APIs or changing GC settings. High memory does not automatically mean a leak.

## How It Works

Objects that survive collections may be promoted through generations. Short-lived objects are commonly collected in younger generations. Large allocations use the Large Object Heap and have different compaction behavior.

An object remains alive while a root can reach it, even if the application no longer intends to use it. Common retention paths include static collections, unbounded caches, timers, closures, and event subscriptions.

## Application

- Analyze reachability when memory grows.
- Bound caches and remove expired entries.
- Unsubscribe from long-lived publishers.
- Release references to large buffers when their work is complete.
- Use heap snapshots or profilers to identify retention paths.

### Event Retention Example

If a long-lived publisher stores a subscriber's event handler, the publisher keeps the subscriber reachable. The subscriber should unsubscribe or dispose its subscription when its lifetime ends.

## Common Mistakes

- Calling `GC.Collect()` as a normal optimization.
- Assuming every retained object is a leak.
- Forgetting static fields and timers as roots.
- Retaining large objects through closures or caches.
- Ignoring LOH and fragmentation when analyzing large buffers.

## Common Interview Questions

### Basic

- What is a GC root?
- What are GC generations?
- What is the Large Object Heap?
- What is a managed memory leak?

### Intermediate

- How can an event handler retain an object?
- Why are short-lived allocations often acceptable?
- How do you investigate a growing heap?

### Advanced

- How do promotion and LOH fragmentation affect latency?
- How do you distinguish cache growth, fragmentation, and a true leak?
- How do weak references trade lifetime safety for complexity?
- How would you analyze a retention path from a heap snapshot?
- How does object lifetime affect generation promotion?

### Follow-up Questions

- Can an object be collected while a local variable exists?
- Does setting a variable to null always fix a leak?
- Can an immutable collection contain mutable objects?

### Code Prediction

Can the object be collected while this field still references it?

```csharp
private static readonly List<object> Cache = [];
```

## Practical Tasks

- Draw the root-to-object path for a retained subscriber.
- Demonstrate and fix an event-handler retention issue.
- Compare heap behavior for short-lived and retained allocations.
- Use a memory profiler to identify a growing cache.

## Readiness Criteria

You should be able to explain reachability, generations, LOH awareness, event retention, bounded ownership, and the evidence needed to diagnose memory growth.

## References

### Microsoft Learn

- [Fundamentals of garbage collection](https://learn.microsoft.com/dotnet/standard/garbage-collection/fundamentals)
- [Large Object Heap](https://learn.microsoft.com/dotnet/standard/garbage-collection/large-object-heap)
- [WeakReference class](https://learn.microsoft.com/dotnet/api/system.weakreference)
