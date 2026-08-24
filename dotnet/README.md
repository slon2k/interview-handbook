# .NET Interview Handbook

This roadmap covers the language, runtime, backend, engineering, and interview skills needed for practical .NET development and interview preparation.

## Scope

The core path is designed for junior-to-mid-level roles. Advanced sections provide optional depth for stronger interviews and follow-up questions; they do not imply a senior-level target.

## Target Baseline

Examples target modern C# and supported .NET LTS releases. Version-specific behavior is called out where relevant.

## Learning Path

- **Part I:** C# and .NET foundations
- **Part II:** Backend application development
- **Part III:** Software engineering context
- **Part IV:** Interview execution

Module links will be added as their documentation is created. The [C# module](m02-csharp-language/README.md) is currently available.

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

This deserves an independent module because it is a frequent differentiator between junior and mid-level candidates.
High-level topics:

- Synchronous versus asynchronous execution
- Concurrency versus parallelism
- I/O-bound versus CPU-bound work
- Task and Task<T>
- How async and await work conceptually
- Why async does not necessarily create a thread
- Task.WhenAll and Task.WhenAny
- Sequential versus concurrent async execution
- Exception propagation
- async void
- Cancellation tokens
- Timeouts
- Task.Run
- Thread and ThreadPool
- Race conditions
- Deadlocks
- Shared mutable state
- lock, Monitor and Interlocked
- SemaphoreSlim
- Concurrent collections
- IAsyncEnumerable<T>
- Async disposal
- Synchronization context and ConfigureAwait
- Bounded concurrency
**Priority:** Critical.

## Part II — Backend Application Development

### Module 7 — HTTP, REST, and ASP.NET Core

This is the main framework module for a backend or full-stack .NET role.
High-level topics:

- HTTP request/response model
- HTTP methods
- Status codes
- Headers and content types
- Idempotency
- REST principles and practical trade-offs
- ASP.NET Core hosting model
- Kestrel and reverse proxies
- Request pipeline
- Middleware and middleware ordering
- Middleware versus filters
- Routing
- Controllers versus minimal APIs
- Model binding
- Validation
- Dependency-injection container
- Service lifetimes: transient, scoped, and singleton
- Configuration and options pattern
- Logging
- Global exception handling
- ProblemDetails
- OpenAPI/Swagger
- Pagination, filtering and sorting
- File handling
- Background services
- HttpClientFactory
- Cancellation of HTTP requests
- CORS
- API versioning
- Health checks
- Basic caching
**Priority:** Critical.

### Module 8 — Relational SQL

SQL fundamentals come first because strong data-access decisions require understanding the queries and database behavior underneath the ORM.
High-level topics:

- Tables, rows and schemas
- Primary and foreign keys
- Normalisation
- Relationships
- Constraints
- Inner and outer joins
- Grouping and aggregation
- Subqueries and CTEs
- Views and stored procedures
- Transactions and ACID
- Isolation levels at a basic level
- Indexes
- Query execution plans
- Pagination
- SQL injection prevention

**Expected outcome:** Design basic relational schemas, write and analyze common SQL queries, and identify indexing, pagination, transaction, and injection risks.

**Priority:** Critical.

### Module 9 — Entity Framework Core

EF Core maps .NET objects to relational data, translates LINQ into SQL, and manages persistence concerns. It should be learned after the relational SQL fundamentals above.
High-level topics:

- DbContext and DbSet
- DbContext lifetime and thread safety
- Entity configuration
- Relationships
- Code-first migrations
- Change tracking
- AsNoTracking
- Query translation
- IQueryable<T>
- Projection
- Eager, explicit and lazy loading
- N+1 queries
- Split versus single queries
- Transactions
- Optimistic concurrency
- Raw SQL
- Bulk operations awareness
- Testing EF-based code
- Repository pattern over EF Core: benefits and possible redundancy

**Expected outcome:** Build a correctly scoped `DbContext`, write efficient queries, manage changes and migrations, and diagnose common translation, loading, and concurrency problems.

**Priority:** Critical.

### Module 10 — Testing and Testability

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

### Module 11 — Application Security

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

### Module 12 — Performance, Diagnostics, and Observability

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

### Module 13 — Architecture and System Design Fundamentals

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

### Module 14 — Development Workflow and Delivery Fundamentals

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

### Module 15 — Practical Interview Exercises

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

### Module 16 — Experience and Behavioural Evidence

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
4. ASP.NET Core and HTTP
5. SQL
6. Entity Framework Core
7. Testing

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
