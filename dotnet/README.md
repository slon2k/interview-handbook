# .NET Interview Handbook

This roadmap covers the language, runtime, backend, engineering, and interview skills needed for practical .NET development and interview preparation.

**Short on time?** See the [Fast Track](fast-track.md) for a curated shortlist of the highest-yield topics instead of the full roadmap below.

## Scope

The core path is designed for junior-to-mid-level roles. Advanced sections provide optional depth for stronger interviews and follow-up questions; they do not imply a senior-level target.

## Target Baseline

Examples target modern C# and supported .NET LTS releases. Version-specific behavior is called out where relevant.

## Learning Path

- **Part I:** C# and .NET foundations
- **Part II:** Backend application development
- **Part III:** Software engineering context
- **Part IV:** Interview execution

Module links will be added as their documentation is created. Modules 2–15 (all of Parts I, II, III, and now Part IV's workflow foundation) are now complete: [C# Language and Type System](m02-csharp-language/README.md), [Collections, LINQ, and Basic Algorithms](m03-collections-linq/README.md), [Object-Oriented Design and Maintainable Code](m04-oop-design/README.md), [Exceptions, Resources, and Memory Management](m05-exceptions-resources-memory/README.md), [Asynchronous Programming and Concurrency](m06-async-concurrency/README.md), [HTTP, REST, and API Design](m07-http-rest-api-design/README.md), [ASP.NET Core Fundamentals](m08-aspnet-core-fundamentals/README.md), [Relational Databases and SQL](m09-relational-databases-and-sql/README.md), [Entity Framework Core](m10-entity-framework-core/README.md), [Testing and Testability](m11-testing-and-testability/README.md), [Application Security](m12-application-security/README.md), [Performance, Diagnostics, and Observability](m13-performance-diagnostics-observability/README.md), [Architecture and System Design Fundamentals](m14-architecture-and-system-design/README.md), and [Development Workflow and Delivery Fundamentals](m15-development-workflow-and-delivery/README.md). Module 1 (.NET Platform and Development Model) remains an outline only — priority Medium, deliberately last in the queue.

## Module Format

Each module should document:

- Scope and prerequisites
- Core topics
- Expected outcomes
- Practical exercises
- Common interview questions
- Direct references

## Part I — C# and .NET Foundations

### Module 1 — The .NET Platform and Development Model

This module establishes what .NET actually is and how an application is built and executed.
High-level topics:

- .NET Framework, .NET Core and modern unified .NET
- Supported and LTS versions
- SDK, runtime, and `dotnet` CLI
- Runtime and SDK selection
- CLR and Base Class Library
- Project and solution structure
- `.csproj` files and target frameworks
- NuGet packages and dependency basics
- `dotnet new`, `restore`, `build`, `run`, and `publish`
- Compilation from C# source to IL and native code
- JIT compilation and basic AOT awareness
- Assemblies, executables, and class libraries
- Framework-dependent and self-contained deployment
- Debug and Release configurations
- Application execution lifecycle
- Cross-platform execution
- Runtime compatibility and target framework selection
- Basic legacy .NET Framework awareness

**Expected outcome:** Explain the .NET platform model, create and build a simple application with the CLI, describe how it executes, and distinguish SDK, runtime, target framework, assembly, and deployment concepts.

**Priority:** Medium. Understand the current platform model; historical version-by-version memorisation is unnecessary.

### Module 2 — C# Language and Type System

This is the central language module and one of the most important interview areas.
High-level topics:

See the completed [C# language and type system module](m02-csharp-language/README.md).

**Status:** Complete  
**Priority:** Critical.

### Module 3 — Collections, LINQ, and Basic Algorithms

See the [Collections, LINQ, and Basic Algorithms module](m03-collections-linq/README.md).

**Status:** Complete  
**Priority:** Critical.


### Module 4 — Object-Oriented Design and Maintainable Code

See the [Object-Oriented Design and Maintainable Code module](m04-oop-design/README.md).

**Status:** Complete  
**Priority:** High.

This is less about naming definitions and more about making and defending design decisions.

### Module 5 — Exceptions, Resources, and Memory Management

See the [Exceptions, Resources, and Memory Management module](m05-exceptions-resources-memory/README.md).

**Status:** Complete  
**Priority:** High.

This combines normal error handling with the relevant parts of the runtime memory model.

### Module 6 — Asynchronous Programming and Concurrency

See the [Asynchronous Programming and Concurrency module](m06-async-concurrency/README.md).

**Status:** Complete  
**Priority:** Critical.

This deserves an independent module because it is a frequent differentiator between junior and mid-level candidates.

## Part II — Backend Application Development

### Module 7 — HTTP, REST, and API Design

See the [HTTP, REST, and API Design module](m07-http-rest-api-design/README.md).

**Status:** Complete  
**Priority:** Critical.

### Module 8 — ASP.NET Core Fundamentals

See the [ASP.NET Core Fundamentals module](m08-aspnet-core-fundamentals/README.md).

**Status:** Complete  
**Priority:** Critical.

### Module 9 — Relational Databases and SQL

See the [Relational Databases and SQL module](m09-relational-databases-and-sql/README.md).

**Status:** Complete  
**Priority:** Critical.

### Module 10 — Entity Framework Core

See the [Entity Framework Core module](m10-entity-framework-core/README.md).

**Status:** Complete  
**Priority:** Critical.

### Module 11 — Testing and Testability

See the [Testing and Testability module](m11-testing-and-testability/README.md).

**Status:** Complete  
**Priority:** Critical.

### Module 12 — Application Security

See the [Application Security module](m12-application-security/README.md).

**Status:** Complete  
**Priority:** High.

### Module 13 — Performance, Diagnostics, and Observability

See the [Performance, Diagnostics, and Observability module](m13-performance-diagnostics-observability/README.md).

**Status:** Complete  
**Priority:** Medium to high.

## Part III — Software Engineering Context

### Module 14 — Architecture and System Design Fundamentals

See the [Architecture and System Design Fundamentals module](m14-architecture-and-system-design/README.md).

**Status:** Complete  
**Priority:** Medium. Practical design reasoning is more important than pattern vocabulary.

### Module 15 — Development Workflow and Delivery Fundamentals

See the [Development Workflow and Delivery Fundamentals module](m15-development-workflow-and-delivery/README.md).

**Status:** Complete  
**Priority:** High for practical readiness, although usually lighter in theoretical interviews.

## Part IV — Interview Execution

### Module 16 — Practical Interview Exercises

This module converts knowledge into interview performance.
Exercise categories:

- Short coding tasks
- LINQ tasks
- Refactoring tasks
- Debugging broken code
- Reviewing a pull request
- Explaining a code snippet
- Writing unit tests
- Designing an endpoint
- Designing a small service
- SQL query exercises
- Finding EF Core performance issues
- Async/concurrency problem analysis
- Small architecture discussion
- “What would you improve?” questions
**Priority:** Critical.

### Module 17 — Experience and Behavioural Evidence

Even though this is not purely technical, it should be connected to the technical modules.
Prepare examples for:

- A difficult bug
- A production or test failure
- A disagreement during code review
- Receiving critical feedback
- Learning a new technology
- Refactoring difficult code
- Improving performance
- Working with unclear requirements
- Missing a deadline or making a mistake
- Helping another developer
- Making a design trade-off
- Handling technical debt
Use the STAR structure, but keep answers natural rather than memorised.
**Priority:** High.

## Recommended Priority Tiers

The modules do not deserve equal preparation time.

### Tier A — Must Be Strong

1. C# language and type system
2. Collections, LINQ, and basic algorithms
3. Async and concurrency
4. HTTP, REST, and API design
5. ASP.NET Core
6. Relational databases and SQL
7. Entity Framework Core
8. Testing

These are likely to determine whether you meet the technical bar.

### Tier B — Must Be Competent

1. OOP and maintainable design
2. Exceptions, resources and memory
3. Security
4. Development workflow and tooling
5. Practical interview exercises

### Tier C — Working Awareness Is Initially Sufficient

1. .NET platform internals
2. Performance and observability
3. Architecture and distributed systems
4. Advanced algorithms
