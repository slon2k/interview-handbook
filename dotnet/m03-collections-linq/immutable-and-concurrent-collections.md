# Immutable and Concurrent Collections

## Definition

Immutable collections cannot be changed after creation. A modification returns a new collection or a structurally shared version. Concurrent collections provide thread-safe operations for shared mutable access.

```csharp
ImmutableArray<int> values = [1, 2, 3];
ImmutableArray<int> updated = values.Add(4);

var queue = new ConcurrentQueue<int>();
queue.Enqueue(1);
queue.TryDequeue(out int item);
```

## Main Collection Types

### Concurrent Collections

| Type | Typical use |
| --- | --- |
| `ConcurrentDictionary<TKey, TValue>` | Thread-safe key/value lookup and updates |
| `ConcurrentQueue<T>` | FIFO work items shared by producers and consumers |
| `ConcurrentStack<T>` | LIFO work items shared by multiple threads |
| `ConcurrentBag<T>` | Unordered items when threads often add and remove their own work; not a FIFO queue |

These types provide thread-safe individual operations. They do not make a sequence of operations an atomic transaction.

### Blocking and Asynchronous Coordination

| Type | Typical use |
| --- | --- |
| `BlockingCollection<T>` | Bounded or blocking producer-consumer workflows over an `IProducerConsumerCollection<T>` |
| `Channel<T>` | Asynchronous producer-consumer pipelines, completion, and backpressure |

`BlockingCollection<T>` and `Channel<T>` are coordination primitives rather than general-purpose collections. Detailed cancellation, backpressure, and bounded-concurrency design belongs in Module 6.

### Immutable Collections

| Type | Typical use |
| --- | --- |
| `ImmutableArray<T>` | Value-type wrapper around a compact immutable array for read-heavy access |
| `ImmutableList<T>` | Reference-type persistent immutable list with structural sharing |
| `ImmutableDictionary<TKey, TValue>` | Immutable key/value snapshots |
| `ImmutableHashSet<T>` | Immutable unique-value sets |
| `ImmutableQueue<T>` and `ImmutableStack<T>` | Immutable FIFO and LIFO workflows |

An immutable collection can contain mutable objects, so immutability of the collection does not guarantee deep immutability of its elements.

### Frozen Collections

| Type | Typical use |
| --- | --- |
| `FrozenDictionary<TKey, TValue>` | .NET 8+, build-once, read-heavy key/value lookup |
| `FrozenSet<T>` | .NET 8+, build-once, read-heavy membership checks |

Frozen collections are optimized after construction for repeated reads. They are useful for static lookup data, not for frequently changing state.

`ImmutableArray<T>` is a struct, while `ImmutableList<T>` is a class. Adding or removing an element from an immutable array generally copies the underlying array and is $O(n)$. Immutable list updates can reuse parts of the existing structure, which is useful when producing many related versions.

## Alternatives & Trade-offs

Use immutable collections to make ownership and sharing easier to reason about. Use concurrent collections when multiple threads truly share a changing data structure. Ordinary collections protected by a clear lock can be simpler when operations must be coordinated across multiple steps.

Immutable collections trade mutation cost and sometimes allocation for safety. Concurrent collections provide atomic collection operations but do not automatically make a whole workflow atomic.

## How It Works

Immutable collections preserve previous versions and may share internal structure. Concurrent collections use synchronization and lock-free or fine-grained techniques depending on the type. Methods such as `TryDequeue` combine checking and mutation atomically.

| Requirement | Suitable starting point |
| --- | --- |
| Shared key/value state | `ConcurrentDictionary<TKey, TValue>` |
| FIFO synchronous work | `ConcurrentQueue<T>` or `BlockingCollection<T>` |
| Asynchronous work pipeline | `Channel<T>` |
| Read-only shared snapshot | An immutable collection |
| Static read-heavy lookup | `FrozenDictionary<TKey, TValue>` or `FrozenSet<T>` |

## Application

- Immutable collections: configuration snapshots, shared state, functional transformations.
- Concurrent collections: producer-consumer queues, shared indexes, work coordination.
- Use `Channel<T>` when the requirement is asynchronous producer-consumer flow with backpressure.

## Common Mistakes

- Assuming immutable means deeply immutable.
- Calling multiple concurrent operations and treating them as one atomic transaction.
- Using concurrent collections without bounding growth.
- Sharing ordinary collections across threads without synchronization.
- Choosing concurrency primitives before defining ownership and workload.

## Common Interview Questions

### Basic

- What is an immutable collection?
- What problem does `ConcurrentQueue<T>` solve?
- Are ordinary collections thread-safe?
- Name the main concurrent collection types.
- What is the difference between immutable and frozen collections?

### Intermediate

- What is structural sharing?
- Why are `Try` methods common in concurrent collections?
- When is a lock simpler than a concurrent collection?
- When would you use `ConcurrentDictionary<TKey, TValue>` instead of a locked dictionary?
- What is the difference between `BlockingCollection<T>` and `Channel<T>`?

### Advanced

- How do immutable collections balance copying and sharing?
- What guarantees do concurrent collection operations provide?
- Why does thread-safe collection access not make a multi-step workflow atomic?
- How would you select between a concurrent queue and a channel?
- How can contention, false sharing, and unbounded growth affect performance?
- How do frozen collections optimize read-heavy workloads?
- What are the allocation and throughput trade-offs between immutable and concurrent collections?
- How would you preserve a consistent immutable snapshot while writers update the next version?
- How do `ImmutableArray<T>` and `ImmutableList<T>` differ in representation and update cost?

### Follow-up Questions

- Can an immutable collection contain mutable objects?
- Does `ConcurrentDictionary` make a factory workflow atomic?
- How should cancellation and completion be modeled for producers and consumers?

### Code Prediction

What is printed?

```csharp
ImmutableArray<int> original = [1, 2];
ImmutableArray<int> updated = original.Add(3);
Console.WriteLine(original.Length);
Console.WriteLine(updated.Length);
```

## Practical Tasks

- Replace shared mutable configuration with immutable snapshots.
- Build a producer-consumer example with a bounded channel.
- Identify which operations in a concurrent workflow must be atomic together.

## Readiness Criteria

Explain immutability, structural sharing, concurrent operation guarantees, workflow atomicity, and when a lock or channel is a better choice.

## References

### Microsoft Learn

- [Immutable collections](https://learn.microsoft.com/dotnet/standard/collections/immutable)
- [Concurrent collections](https://learn.microsoft.com/dotnet/standard/collections/thread-safe/)
- [ConcurrentDictionary<TKey, TValue>](https://learn.microsoft.com/dotnet/api/system.collections.concurrent.concurrentdictionary-2)
- [BlockingCollection<T>](https://learn.microsoft.com/dotnet/api/system.collections.concurrent.blockingcollection-1)
- [Channel<T> class](https://learn.microsoft.com/dotnet/api/system.threading.channels.channel-1)
- [Frozen collections](https://learn.microsoft.com/dotnet/api/system.collections.frozen)
