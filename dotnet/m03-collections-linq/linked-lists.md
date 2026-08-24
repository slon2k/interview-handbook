# `LinkedList<T>`

## Definition

`LinkedList<T>` stores elements in nodes connected by links. Each node can reference its previous and next node.

```csharp
var values = new LinkedList<int>([1, 3]);
LinkedListNode<int> node = values.First!;
values.AddAfter(node, 2);
```

## Alternatives & Trade-offs

Linked lists provide constant-time insertion or removal when the node is already known, but locating an element is linear and node allocation reduces locality. Prefer `List<T>` for most ordered data.

## How It Works

A linked list stores node objects rather than a contiguous buffer. `AddAfter`, `AddBefore`, and removal are $O(1)$ with a node reference; search and indexed traversal are $O(n)$.

## Application

Use linked lists for specialized algorithms that already hold node references or require frequent insertion/removal around known nodes. They are uncommon as a general-purpose default.

## Common Mistakes

- Choosing a linked list for fast indexed access.
- Ignoring per-node allocation and cache locality.
- Losing the node reference before removal.
- Assuming insertion by value is constant time.
- Treating it as automatically thread-safe.

## Common Interview Questions

### Basic

- What is a linked list?
- How does it differ from an array-backed list?
- What is a node?

### Intermediate

- What is the complexity of search and node insertion?
- Why can `List<T>` outperform a linked list?
- What is the role of `LinkedListNode<T>`?

### Advanced

- How do cache locality and allocation affect linked-list performance?
- When does a linked list make algorithmic sense?
- How would you implement an intrusive list?
- What ownership and lifetime issues exist around retained nodes?
- How would you benchmark it against a list fairly?

### Follow-up Questions

- Does `LinkedList<T>` support indexing efficiently?
- Can a node belong to two lists?
- What happens when a node is removed?

### Code Prediction

What is printed?

```csharp
var values = new LinkedList<int>([1, 3]);
values.AddAfter(values.First!, 2);
Console.WriteLine(string.Join(",", values));
```

## Practical Tasks

- Implement an LRU list component using node references.
- Compare middle insertion in a linked list and a list.
- Measure allocation and traversal differences.

## Readiness Criteria

Explain node-based storage, operation complexity, locality costs, and why linked lists are a specialized rather than default collection.

## References

### Microsoft Learn

- [LinkedList<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.linkedlist-1)
- [LinkedListNode<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.linkedlistnode-1)
