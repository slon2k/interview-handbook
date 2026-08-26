# Module 4 - Object-Oriented Design and Maintainable Code

**Status:** Complete  
**Priority:** High  
**Prerequisites:** [C# Language and Type System](../m02-csharp-language/README.md)

## Scope

This module covers object-oriented design, maintainability, refactoring, and common design patterns for practical .NET development and interviews.

The focus is on making and defending design decisions rather than memorizing pattern definitions. Examples should explain trade-offs, ownership, extensibility, testability, and operational consequences.

## Learning Outcomes

By the end of this module, you should be able to:

- Explain and apply encapsulation, abstraction, inheritance, and polymorphism.
- Choose composition or inheritance based on the relationship and change points.
- Design clear interfaces, abstract classes, and public APIs.
- Recognize coupling, cohesion, and violations of separation of concerns.
- Apply SOLID, DRY, KISS, and YAGNI pragmatically.
- Use guard clauses and immutability to make code easier to reason about.
- Apply dependency inversion and dependency injection as design techniques.
- Identify code smells and propose focused refactorings.
- Select, implement, and explain common design patterns without overengineering.

## Topics

### 1. OOP Foundations

- [Encapsulation](encapsulation.md)
- [Abstraction](abstraction.md)
- [Inheritance](inheritance.md)
- [Polymorphism](polymorphism.md)
- [Composition versus inheritance](composition-versus-inheritance.md)
- [Interfaces versus abstract classes](interfaces-versus-abstract-classes.md)
- [Method overloading versus overriding](overloading-versus-overriding.md)
- [API and class design](api-and-class-design.md)

### 2. Maintainable Design

- [Coupling and cohesion](coupling-and-cohesion.md)
- [Separation of concerns](separation-of-concerns.md)
- [SOLID principles](solid-principles.md)
- [DRY, KISS, and YAGNI](dry-kiss-yagni.md)
- [Guard clauses](guard-clauses.md)
- [Immutability in object design](immutability-in-object-design.md)
- [Dependency inversion and dependency injection](dependency-inversion-and-injection.md)

Dependency injection is covered here as a design technique. ASP.NET Core container configuration and service lifetimes belong in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md).

### 3. Code Quality and Refactoring

- [Code smells](code-smells.md)
- [Refactoring techniques](refactoring-techniques.md)
- [Pattern overuse and overengineering](pattern-overuse-and-overengineering.md)

### 4. Common Design Patterns

- [Strategy](strategy.md)
- [Factory](factory.md)
- [Adapter](adapter.md)
- [Decorator](decorator.md)
- [Observer](observer.md)
- [Builder](builder.md)
- [Repository](repository.md)

Patterns should be introduced as responses to recurring design problems. Each topic should explain when a simpler design is preferable.

### Pattern Selection Guide

| Need | Pattern |
| --- | --- |
| Interchangeable algorithm or policy | Strategy |
| Controlled or variable object creation | Factory |
| Convert an incompatible interface | Adapter |
| Add behavior while preserving a contract | Decorator |
| Notify multiple in-process subscribers | Observer |
| Construct a complex object step by step | Builder |
| Hide a meaningful persistence boundary | Repository |

## Scope Boundaries

- C# syntax and type semantics belong in [Module 2 - C# Language and Type System](../m02-csharp-language/README.md).
- ASP.NET Core dependency-injection registration, service lifetimes, middleware, and framework configuration belong in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md).
- Unit, integration, and end-to-end testing belong in Module 11 - Testing and Testability, although testability is a design concern here.
- Large-scale architecture and distributed-system trade-offs belong in Module 14 - Architecture and System Design Fundamentals.

## Suggested Learning Sequence

1. Encapsulation, abstraction, interfaces, and polymorphism.
2. Composition versus inheritance and API design.
3. Coupling, cohesion, and separation of concerns.
4. SOLID, DRY, KISS, YAGNI, and guard clauses.
5. Immutability and dependency inversion.
6. Code smells and focused refactoring.
7. Strategy, Factory, Adapter, Decorator, Observer, Builder, and Repository.
8. Pattern selection and overengineering review.

## Practical Deliverables

- Design a small service using interfaces and composition.
- Refactor a tightly coupled class into dependency-inverted components.
- Review a class for SOLID violations and propose the smallest useful change.
- Identify code smells in a legacy method and apply two focused refactorings.
- Compare a pattern-based design with a simpler direct implementation.
- Explain the trade-offs of Repository over EF Core in a data-access scenario.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and principles.
- Intermediate questions involving implementation and design trade-offs.
- Advanced questions involving extensibility, testability, coupling, and maintenance cost.
- Follow-up questions that challenge pattern selection and practical judgment.
- Code-prediction or code-review questions where useful.

## References

### Microsoft Learn

- [Object-oriented programming](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/)
- [Interfaces](https://learn.microsoft.com/dotnet/csharp/fundamentals/types/interfaces)
- [Inheritance](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/inheritance)
- [Polymorphism](https://learn.microsoft.com/dotnet/csharp/fundamentals/object-oriented/polymorphism)
- [Dependency injection in .NET](https://learn.microsoft.com/dotnet/core/extensions/dependency-injection)
