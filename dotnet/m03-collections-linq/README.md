# Module 3 - Collections, LINQ, and Basic Algorithms

**Status:** Complete  
**Priority:** Critical  
**Prerequisites:** [C# Language and Type System](../m02-csharp-language/README.md)

## Scope

This module covers everyday collection choices, LINQ-based data processing, iteration, complexity, and basic algorithms used in .NET applications and interviews.

The module focuses on practical reasoning: selecting an appropriate data structure, understanding query execution, measuring complexity, and writing clear solutions.

## Learning Outcomes

By the end of this module, you should be able to:

- Choose an appropriate collection for a given access and mutation pattern.
- Explain the common time and space costs of collection operations.
- Use equality comparers correctly with sets and dictionaries.
- Write readable LINQ queries for projection, filtering, grouping, joining, and aggregation.
- Explain deferred execution, materialization, and repeated enumeration.
- Implement and analyze basic search, sorting, recursion, and tree traversal solutions.
- Identify common performance, correctness, and readability problems in collection code.

## Topics

### 1. Core Collections

- [Arrays and multidimensional or jagged arrays](arrays.md)
- [`List<T>`](lists.md)
- [`Dictionary<TKey, TValue>`](dictionaries.md)
- [`HashSet<T>`](sets.md)
- [`Queue<T>` and `Stack<T>`](queues-and-stacks.md)
- [`LinkedList<T>` and when it is useful](linked-lists.md)
- [Immutable and concurrent collections](immutable-and-concurrent-collections.md)

Concurrent collections are covered at an awareness level here. Detailed synchronization and coordination belong in [Module 6 - Asynchronous Programming and Concurrency](../m06-async-concurrency/README.md).

### 2. Collection Design

- [Selecting the correct collection](selecting-collections.md)
- [Read-only and mutable collection interfaces](collection-interfaces.md)
- [Equality comparers and dictionary keys](equality-comparers.md)
- [Hashing and dictionary keys](hashing-and-dictionary-keys.md)
- [Time and space complexity](complexity-analysis.md)
- [Capacity, resizing, and allocation awareness](capacity-resizing-and-allocation.md)

### 3. LINQ and Iteration

- [`IEnumerable<T>` and iterators](ienumerable-and-iterators.md)
- [LINQ query operators](linq-query-operators.md)
- [Deferred execution and materialization](deferred-execution-and-materialization.md)
- [`IEnumerable<T>` versus `IQueryable<T>`](ienumerable-vs-iqueryable.md)

`IQueryable<T>` is introduced here as a contrast with in-memory LINQ. Query translation, database execution, loading, and provider-specific behavior belong primarily in [Module 10 - Entity Framework Core](../m10-entity-framework-core/README.md).

### 4. Basic Algorithms

- [Searching and sorting](searching-and-sorting.md)
- [Binary search](binary-search.md)
- [Recursion](recursion.md)
- [Trees, BFS, and DFS](tree-traversal.md)
- [Collection and string interview problems](collection-interview-problems.md)

Advanced graph algorithms are optional for the target roles covered by this handbook.

## Scope Boundaries

- C# generics, delegates, iterators, and value semantics are covered in the C# module.
- Thread safety and synchronization belong in [Module 6 - Asynchronous Programming and Concurrency](../m06-async-concurrency/README.md).
- Detailed SQL and query-provider behavior belong in Modules 8 and 9.
- Advanced algorithms and specialized data structures are optional unless a target role requires them.

## Suggested Learning Sequence

1. Arrays, lists, sets, dictionaries, queues, and stacks.
2. Collection interfaces and equality comparers.
3. Complexity and collection performance.
4. `IEnumerable<T>`, iterators, and deferred execution.
5. Core LINQ operators, streaming, and materialization.
6. Query composition, grouping, joining, and repeated enumeration.
7. Searching, sorting, recursion, and tree traversal.
8. Mixed interview problems and performance reviews.

## Practical Deliverables

- Create a collection selection guide for common access patterns.
- Implement a small in-memory search and filtering service.
- Demonstrate deferred execution and repeated enumeration with logging.
- Compare `List<T>`, `HashSet<T>`, and `Dictionary<TKey, TValue>` for representative operations.
- Implement binary search and explain its complexity.
- Solve one tree traversal problem with both breadth-first and depth-first approaches.
- Review a LINQ-heavy method for correctness, allocations, and readability.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and API familiarity.
- Intermediate questions involving implementation and trade-offs.
- Advanced questions involving complexity, execution behavior, and performance.
- Follow-up questions that test practical judgment.
- Code-prediction questions involving deferred execution, mutation, and enumeration.

## References

### Microsoft Learn

- [Collections and data structures](https://learn.microsoft.com/dotnet/standard/collections/)
- [Selecting a collection class](https://learn.microsoft.com/dotnet/standard/collections/selecting-a-collection-class)
- [Generic collections in .NET](https://learn.microsoft.com/dotnet/standard/generics/collections)
- [Language Integrated Query (LINQ)](https://learn.microsoft.com/dotnet/csharp/linq/)
- [Standard query operators overview](https://learn.microsoft.com/dotnet/csharp/linq/standard-query-operators/)
- [Deferred execution and lazy evaluation](https://learn.microsoft.com/dotnet/standard/linq/deferred-execution-lazy-evaluation)
- [Iterators](https://learn.microsoft.com/dotnet/csharp/iterators)
