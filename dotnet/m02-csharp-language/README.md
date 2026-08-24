# Module 2 - C# Language and Type System

**Status:** Complete
**Priority:** Critical
**Prerequisites:** None (this is the entry point into the core learning path).

## Scope

This module covers the C# language and type system: syntax fundamentals, value versus reference semantics, the type-declaration constructs (classes, structs, records), nullability, equality, generics, functional-style constructs, and the cross-cutting language features (events, attributes, reflection, exceptions) that support everything built in later modules.

The focus is on precise, correct mental models — how the language actually behaves, not just what its syntax looks like — since this is the module interviewers probe most aggressively for depth.

## Learning Outcomes

By the end of this module, you should be able to:

- Explain value versus reference semantics and predict copy/aliasing behavior correctly.
- Choose between class, struct, record, and record struct for a given design.
- Reason precisely about nullability, both for value types and reference types.
- Implement correct equality and hashing for a custom type.
- Use pattern matching, generics, and delegates/lambdas idiomatically.
- Explain how events, attributes, reflection, and exceptions work under the hood, not just how to use them.
- Predict the output of non-obvious C# code snippets involving boxing, closures, and dispatch.

## Topics

### 1. Language Fundamentals

- [Variables, expressions and control flow](variables-expressions-and-control-flow.md)
- [Built-in numeric types and conversions](built-in-numeric-types.md)
- [Value and reference semantics](value-and-reference-semantics.md)
- [Parameters: `ref`, `out`, `in`, `params`](parameters-ref-out-in-params.md)

### 2. Types and Members

- [Classes, structs, records and record structs](classes-structs-records-and-record-structs.md)
- [Fields, properties and constructors](fields-properties-constructors.md)
- [Access modifiers](access-modifiers.md)
- [Static members and static classes](static-members-and-static-classes.md)

### 3. Nullability and Equality

- [Nullable value types](nullable-value-types.md)
- [Nullable reference types](nullable-reference-types.md)
- [Equality and hashing](equality-and-hashing.md)
- [Strings and immutability](strings-and-immutability.md)

### 4. Functional and Modern Constructs

- [Tuples and deconstruction](tuples-and-deconstruction.md)
- [Pattern matching](pattern-matching.md)
- [Generics and constraints](generics-and-constraints.md)
- [Covariance and contravariance](covariance-and-contravariance.md)
- [Delegates, lambdas and closures](delegates-lambdas-and-closures.md)
- [Modern C# syntax (init properties, using declarations, required members, collection expressions, span, etc.)](modern-csharp-syntax.md)

### 5. Cross-Cutting Language Features

- [Events](events.md)
- [Attributes](attributes.md)
- [Reflection basics](reflection-basics.md)
- [Exception handling](exception-handling.md)

## Cross-Folder References

Topics related to but documented elsewhere:

- **Interfaces & abstract classes** → [OOP Design module](../m04-oop-design/interfaces-versus-abstract-classes.md)
- **Method overloading/overriding** → [OOP Design module](../m04-oop-design/overloading-versus-overriding.md)
- **Collections & LINQ** → [Collections, LINQ, and Basic Algorithms module](../m03-collections-linq/README.md)
- **Async/await & parallelism** → [Asynchronous Programming and Concurrency module](../m06-async-concurrency/README.md)

## Scope Boundaries

- Collection types and LINQ operators belong in [Module 3 - Collections, LINQ, and Basic Algorithms](../m03-collections-linq/README.md).
- OOP design principles, SOLID, and design patterns belong in [Module 4 - Object-Oriented Design and Maintainable Code](../m04-oop-design/README.md); this module covers only the language mechanics (e.g., `virtual`/`override` syntax), not design judgment.
- Deep exception *design* (boundaries, custom exception hierarchies, when to throw vs. return a result) belongs in [Module 5 - Exceptions, Resources, and Memory Management](../m05-exceptions-resources-memory/README.md); this module covers exception-handling syntax and mechanics.
- `async`/`await`, `Task`, and concurrency primitives belong in [Module 6 - Asynchronous Programming and Concurrency](../m06-async-concurrency/README.md).

## Suggested Learning Sequence

1. Variables, expressions, control flow, numeric types, and value/reference semantics.
2. Type-declaration constructs: classes, structs, records, and their members.
3. Nullability and equality.
4. Tuples, pattern matching, generics, variance, and delegates/lambdas.
5. Events, attributes, reflection, and exception-handling mechanics.

## Practical Deliverables

- Predict and explain the output of a set of value/reference-semantics and boxing code snippets without running them.
- Implement correct `Equals`/`GetHashCode` (or use a record) for a value-object type and justify the choice.
- Refactor a type-checking `if`/`else` chain into pattern matching.
- Write a generic method with appropriate constraints for a stated requirement.
- Demonstrate a closure-capture bug (e.g., capturing a loop variable) and fix it.
- Build a small publisher/subscriber example using `event` and explain its lifetime implications.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and syntax.
- Intermediate questions involving common usage and trade-offs.
- Advanced questions involving runtime behavior, performance, and edge cases.
- Follow-up questions that test precise understanding rather than memorized definitions.
- Code-prediction questions, since this module is the most common source of "what does this print" interview questions.

## References

### Microsoft Learn

- [C# documentation](https://learn.microsoft.com/dotnet/csharp/)
- [C# language reference](https://learn.microsoft.com/dotnet/csharp/language-reference/)
- [.NET API browser](https://learn.microsoft.com/dotnet/api/)

### Other

- [C# language design repository](https://github.com/dotnet/csharplang)
