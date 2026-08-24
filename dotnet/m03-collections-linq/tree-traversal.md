# Trees, BFS, and DFS

## Definition

A tree is a hierarchical structure of nodes. Breadth-first search (BFS) explores level by level, while depth-first search (DFS) follows a path before backtracking.

```csharp
public sealed class Node(string value)
{
	public string Value { get; } = value;
	public List<Node> Children { get; } = [];
}

var root = new Node("Root");
var left = new Node("A");
var right = new Node("B");
left.Children.Add(new Node("C"));
root.Children.Add(left);
root.Children.Add(right);
```

## Alternatives & Trade-offs

BFS is useful for shortest unweighted paths and level-order processing, but its queue can grow with the width of the tree. DFS uses space proportional to depth and is useful for exhaustive exploration, subtree processing, and backtracking.

Use recursion for naturally recursive trees when depth is controlled; use an explicit stack for untrusted or very deep input.

## How It Works

BFS dequeues a node, processes it, and enqueues each unvisited child. DFS pushes each child onto a stack and processes nodes until the stack is empty. With a visited set, both run in $O(V + E)$ on a graph; for a tree with $n$ nodes, traversal is $O(n)$.

```csharp
var visited = new HashSet<Node>();
var queue = new Queue<Node>();
queue.Enqueue(root);
visited.Add(root);

while (queue.Count > 0)
{
	Node current = queue.Dequeue();
	Console.WriteLine(current.Value);

	foreach (Node child in current.Children)
	{
		if (visited.Add(child))
		{
			queue.Enqueue(child);
		}
	}
}
```

The equivalent iterative DFS uses a stack. Because a stack is last-in, first-out, children are pushed in reverse order when left-to-right visit order is required.

```csharp
var visited = new HashSet<Node>();
var stack = new Stack<Node>();
stack.Push(root);

while (stack.Count > 0)
{
	Node current = stack.Pop();

	if (!visited.Add(current))
	{
		continue;
	}

	Console.WriteLine(current.Value);

	for (int index = current.Children.Count - 1; index >= 0; index--)
	{
		stack.Push(current.Children[index]);
	}
}
```

A tree has no cycles, but using a visited set is necessary when the structure may actually be a graph or may contain shared nodes.

## Application

- Find the shortest number of edges in an unweighted graph with BFS.
- Process organizational hierarchies with DFS.
- Perform level-order tree output with BFS.
- Search nested folders or dependency graphs while preventing revisits.

## Common Mistakes

- Forgetting cycle detection when traversing graph-like data.
- Marking nodes visited too late and enqueuing duplicates.
- Using BFS when only depth-based processing is required.
- Recursing into unbounded input.
- Confusing tree traversal complexity with an operation that searches every node repeatedly.

## Common Interview Questions

### Basic

- What is the difference between BFS and DFS?
- Which collection supports BFS?
- Which collection supports iterative DFS?

### Intermediate

- What are the time and space complexities?
- When does BFS find a shortest path?
- Why is a visited set needed for graphs?

### Advanced

- How does tree width versus depth affect BFS and DFS memory use?
- How would you reconstruct a shortest path with BFS?
- How do iterative and recursive DFS differ in failure modes?
- How would you traverse a graph with millions of nodes?
- How do adjacency representation choices affect traversal cost?

### Follow-up Questions

- Does DFS always find the shortest path?
- Can a tree contain a cycle?
- What happens when the root is null?

### Code Prediction

What order is printed by BFS?

```csharp
var queue = new Queue<Node>();
queue.Enqueue(root);

while (queue.Count > 0)
{
	Node current = queue.Dequeue();
	Console.WriteLine(current.Value);

	foreach (Node child in current.Children)
	{
		queue.Enqueue(child);
	}
}
```

For the tree created above, the output is:

```text
Root
A
B
C
```

## Practical Tasks

- Implement BFS level-order traversal.
- Implement iterative and recursive DFS.
- Add cycle detection and parent tracking to a graph traversal.

## Readiness Criteria

Choose BFS or DFS based on the problem, explain time and space complexity, use queues or stacks correctly, and prevent cycles and excessive recursion depth.

## References

### Microsoft Learn

- [Queue<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.queue-1)
- [Stack<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.stack-1)
- [HashSet<T> class](https://learn.microsoft.com/dotnet/api/system.collections.generic.hashset-1)
