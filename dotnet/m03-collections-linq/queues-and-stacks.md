# `Queue<T>` and `Stack<T>`

## Definition

`Queue<T>` models first-in, first-out (FIFO) processing. `Stack<T>` models last-in, first-out (LIFO) processing.

```csharp
var queue = new Queue<string>();
queue.Enqueue("first");
string item = queue.Dequeue();

var stack = new Stack<string>();
stack.Push("top");
string value = stack.Pop();
```

## Alternatives & Trade-offs

Use a queue for work scheduling and breadth-first traversal. Use a stack for undo operations, parsing, and depth-first traversal. A concurrent queue or channel is more appropriate when multiple threads produce and consume work.

## How It Works

Both are resizable collections. `Enqueue`/`Dequeue` and `Push`/`Pop` are generally amortized $O(1)$. `Peek` observes the next element without removing it.

## Application

- Queues: work items, breadth-first search, request buffering.
- Stacks: depth-first search, undo history, delimiter matching, call-like workflows.

## Common Mistakes

- Calling `Dequeue` or `Pop` when empty.
- Using the wrong ordering model.
- Assuming collections provide thread-safe coordination.
- Removing items while enumerating.
- Using an unbounded queue without a backpressure policy.

## Common Interview Questions

### Basic

- What is FIFO?
- What is LIFO?
- What is the difference between `Peek` and `Dequeue` or `Pop`?

### Intermediate

- What are the operation complexities?
- How can a queue implement breadth-first search?
- How can a stack implement depth-first search?

### Advanced

- How does queue ring-buffer behavior reduce shifting?
- How would you bound a work queue?
- What is the difference between `Queue<T>` and `ConcurrentQueue<T>`?
- How do queue and stack allocations grow?
- How would you preserve fairness in a producer-consumer system?

### Follow-up Questions

- What exception is thrown when removing from an empty collection?
- Can these collections be enumerated without mutation?
- When should a channel replace a concurrent queue?

### Code Prediction

What is printed?

```csharp
var values = new Queue<int>([1, 2]);
values.Enqueue(3);
Console.WriteLine(values.Dequeue());

var stack = new Stack<int>([1, 2]);
Console.WriteLine(stack.Pop());
```

## Practical Tasks

- Implement breadth-first and depth-first traversal using these types.
- Build a bounded work queue design.
- Implement delimiter matching with a stack.

## Readiness Criteria

Choose FIFO or LIFO correctly, explain operation complexity and empty behavior, and distinguish ordinary collections from concurrent coordination primitives.

## References

### Microsoft Learn

- [Queue<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.queue-1)
- [Stack<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.stack-1)
