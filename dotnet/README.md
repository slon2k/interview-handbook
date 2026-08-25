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

Module links will be added as their documentation is created. Modules 2–9 (Part I's language, collections, design, memory, and concurrency content, plus Part II's HTTP/API, framework, and database foundations) are now complete: [C# Language and Type System](m02-csharp-language/README.md), [Collections, LINQ, and Basic Algorithms](m03-collections-linq/README.md), [Object-Oriented Design and Maintainable Code](m04-oop-design/README.md), [Exceptions, Resources, and Memory Management](m05-exceptions-resources-memory/README.md), [Asynchronous Programming and Concurrency](m06-async-concurrency/README.md), [HTTP, REST, and API Design](m07-http-rest-api-design/README.md), [ASP.NET Core Fundamentals](m08-aspnet-core-fundamentals/README.md), and [Relational Databases and SQL](m09-relational-databases-and-sql/README.md). Module 1 (.NET Platform and Development Model) remains an outline only — priority Medium, deliberately last in the queue.

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

EF Core maps .NET objects to relational data, translates LINQ into SQL, and manages persistence concerns. It should be learned after the relational database fundamentals above.
High-level topics:

- DbContext and DbSet
- DbContext lifetime and thread safety
- DbContext pooling
- Entity configuration
- Relationships
- Code-first migrations
- Change tracking
- AsNoTracking
- Query translation
- `IQueryable<T>` and EF Core's LINQ provider translation
- Projection
- Eager, explicit and lazy loading
- N+1 queries
- Split versus single queries
- Global query filters (e.g., soft delete)
- Transactions
- Optimistic concurrency
- Raw SQL
- Bulk operations (`ExecuteUpdate`/`ExecuteDelete`, bulk-operation libraries)
- Testing EF-based code
- Repository pattern in EF Core: interaction with change tracking and DbContext lifetime

**Expected outcome:** Build a correctly scoped `DbContext`, write efficient queries, manage changes and migrations, and diagnose common translation, loading, and concurrency problems.

**Priority:** Critical.

### Module 11 — Testing and Testability

This module should cover both concepts and actual test implementation.
High-level topics:

- Unit, integration, component and end-to-end tests
- Test pyramid and practical variations
- Arrange–Act–Assert
- Test naming
- Testing behaviour rather than implementation
- xUnit or NUnit
- Assertions
- Parameterised tests
- Test fixtures
- Test data builders
- Mocks, stubs and fakes
- Moq or NSubstitute
- What should and should not be mocked
- Testing asynchronous code
- Testing exceptions
- Testing HttpClient
- ASP.NET Core integration testing
- WebApplicationFactory
- Database integration tests
- Testcontainers
- Code coverage and its limitations
- Reliable and deterministic tests
- Tests in CI
**Priority:** Critical.

### Module 12 — Application Security

For your level, focus on secure application-development fundamentals rather than cryptographic implementation details.
High-level topics:

- Authentication versus authorisation
- Claims, roles and policies
- Cookies versus bearer tokens
- JWT structure and validation
- Access and refresh tokens
- Password storage
- Secret management
- HTTPS
- CORS versus CSRF
- XSS
- SQL injection
- Input validation
- Output encoding
- Mass assignment/over-posting
- File-upload security
- Rate limiting
- Principle of least privilege
- Secure logging
- OWASP Top 10
- OWASP API Security Top 10
- GDPR awareness
**Priority:** High.

### Module 13 — Performance, Diagnostics, and Observability

The goal is not advanced optimisation. It is demonstrating that you can investigate performance systematically.
High-level topics:

- Measuring before optimising
- CPU-bound versus I/O-bound bottlenecks
- Allocations and GC pressure
- Common collection and LINQ performance issues
- Database-query performance
- N+1 problems
- Caching strategies
- In-memory versus distributed caching
- Logging and structured logging
- Metrics
- Tracing
- Correlation IDs
- OpenTelemetry awareness
- Application Insights awareness
- Health checks
- Profilers and diagnostic tools
- Load and stress testing
- Basic resilience: timeouts, retries, and circuit breakers
**Priority:** Medium to high.

## Part III — Software Engineering Context

### Module 14 — Architecture and System Design Fundamentals

For junior+/mid-level positions, this should focus on explaining reasonable designs rather than designing global-scale systems.
High-level topics:

- Layered architecture
- Modular monolith
- Clean/hexagonal architecture at a conceptual level
- Separation of domain and infrastructure
- Dependency direction
- DTOs versus domain entities
- Application services
- Repositories and units of work
- Synchronous versus asynchronous communication
- Messaging fundamentals
- Event-driven architecture
- Caching
- Relational versus NoSQL: when each fits (awareness)
- Stateless application design
- Horizontal versus vertical scaling
- Availability and resilience
- Monolith versus microservices
- Microservice trade-offs
- Distributed transactions and eventual consistency
- Idempotent message handling
- Basic CQRS awareness
- Basic domain-driven design vocabulary
- Avoiding unnecessary architecture
**Priority:** Medium. Practical design reasoning is more important than pattern vocabulary.

### Module 15 — Development Workflow and Delivery Fundamentals

This covers the normal daily workflow expected from a professional developer.
High-level topics:

- Git fundamentals
- Branching and pull requests
- Commit quality
- Merge conflicts
- Code review
- Debugging in Visual Studio or Rider
- Breakpoints and conditional breakpoints
- NuGet dependency management
- Build and publish commands
- Project configuration
- Environment-specific settings
- Static analysis and analyzers
- Formatting
- Docker fundamentals
- CI/CD concepts
- Running tests in a pipeline
- Database migrations during deployment
- Feature flags
- Basic release and rollback thinking
Azure-specific delivery details can be moved to the later cloud plan.
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
