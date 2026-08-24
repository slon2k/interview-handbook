# Module 3 - Collections, LINQ, and Basic Algorithms

**Status:** In progress  
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

- Arrays and multidimensional or jagged arrays
- `List<T>`
- `Dictionary<TKey, TValue>`
- `HashSet<T>`
- `Queue<T>` and `Stack<T>`
- `LinkedList<T>` and when it is useful
- Immutable collections
- Concurrent collections at an awareness level

### 2. Collection Design

- Selecting the correct collection
- Read-only and mutable collection interfaces
- Equality comparers and dictionary keys
- Hashing requirements for sets and dictionaries
- Time and space complexity
- Capacity, resizing, and allocation awareness

### 3. LINQ and Iteration

- `IEnumerable<T>`
- Iterators and `yield return`
- Deferred execution
- Projection and filtering
- Aggregation
- Grouping and joining
- `Select` versus `SelectMany`
- `First`, `Single`, `Any`, and `All`
- Ordering and distinct results
- Materialization with `ToList` and `ToArray`
- Repeated enumeration
- `IEnumerable<T>` versus `IQueryable<T>`

`IQueryable<T>` is introduced here as a contrast with in-memory LINQ. Query translation, database execution, loading, and provider-specific behavior belong primarily in [Module 9 - Entity Framework Core](../README.md#module-9--entity-framework-core).

### 4. Basic Algorithms

- Searching and sorting
- Binary search
- Recursion
- Basic tree representation
- Breadth-first search and depth-first search awareness
- Common interview problems involving strings and collections

Advanced graph algorithms are optional for the target roles covered by this handbook.

## Scope Boundaries

- C# generics, delegates, iterators, and value semantics are covered in the C# module.
- Thread safety and synchronization belong in Module 6 - Asynchronous Programming and Concurrency.
- Detailed SQL and query-provider behavior belong in Modules 8 and 9.
- Advanced algorithms and specialized data structures are optional unless a target role requires them.

## Suggested Learning Sequence

1. Arrays, lists, sets, dictionaries, queues, and stacks.
2. Collection interfaces and equality comparers.
3. Complexity and collection performance.
4. `IEnumerable<T>`, iterators, and deferred execution.
5. Core LINQ operators and materialization.
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
- [Complexity analysis](https://learn.microsoft.com/dotnet/standard/collections/)
