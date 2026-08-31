# Fast Track — .NET Interview Prep

A curated shortlist for when you don't have time to read every module in full. This is **not** a replacement for the modules — it's a triage tool: read these first, and go deeper on the full module topic list wherever time allows.

Unlike the "Recommended Priority Tiers" list in the [main README](README.md), which ranks whole *modules*, this page picks the highest-yield *individual topics* within those modules — not every file in a Critical-priority module is equally load-bearing, and this list is meant to reflect that.

This page will grow as new modules are completed. Until then, it only covers what exists today.

## Core — Read These First

The topics most likely to come up, directly or as follow-ups, in a real interview loop.

### C# Language ([Module 2](m02-csharp-language/README.md))

- [Value and reference semantics](m02-csharp-language/value-and-reference-semantics.md)
- [Classes, structs, records and record structs](m02-csharp-language/classes-structs-records-and-record-structs.md)
- [Nullable reference types](m02-csharp-language/nullable-reference-types.md)
- [Equality and hashing](m02-csharp-language/equality-and-hashing.md)
- [Pattern matching](m02-csharp-language/pattern-matching.md)
- [Generics and constraints](m02-csharp-language/generics-and-constraints.md)
- [Delegates, lambdas and closures](m02-csharp-language/delegates-lambdas-and-closures.md)
- [Exception handling](m02-csharp-language/exception-handling.md)

### Collections, LINQ, and Basic Algorithms ([Module 3](m03-collections-linq/README.md))

- [Selecting the correct collection](m03-collections-linq/selecting-collections.md)
- [`Dictionary<TKey, TValue>`](m03-collections-linq/dictionaries.md)
- [Time and space complexity](m03-collections-linq/complexity-analysis.md)
- [Deferred execution and materialization](m03-collections-linq/deferred-execution-and-materialization.md)
- [LINQ query operators](m03-collections-linq/linq-query-operators.md)
- [`IEnumerable<T>` versus `IQueryable<T>`](m03-collections-linq/ienumerable-vs-iqueryable.md)
- [Collection and string interview problems](m03-collections-linq/collection-interview-problems.md)

### Asynchronous Programming and Concurrency ([Module 6](m06-async-concurrency/README.md))

- [How `async`/`await` work conceptually](m06-async-concurrency/async-await-mechanics.md)
- [`Task.WhenAll`, `Task.WhenAny`, and sequential vs. concurrent execution](m06-async-concurrency/combinators-and-execution-shape.md)
- [Exception propagation and `async void`](m06-async-concurrency/exception-handling-in-async-code.md)
- [Race conditions, deadlocks, and shared mutable state](m06-async-concurrency/race-conditions-and-deadlocks.md)
- [Synchronization context and `ConfigureAwait`](m06-async-concurrency/synchronization-context-and-configureawait.md)
- [Cancellation tokens and timeouts](m06-async-concurrency/cancellation-and-timeouts.md)

### HTTP, REST, and API Design ([Module 7](m07-http-rest-api-design/README.md))

- [HTTP methods and idempotency](m07-http-rest-api-design/http-methods-and-idempotency.md)
- [HTTP status codes](m07-http-rest-api-design/status-codes.md)
- [REST principles and practical trade-offs](m07-http-rest-api-design/rest-principles-and-trade-offs.md)
- [Pagination, filtering, and sorting](m07-http-rest-api-design/pagination-filtering-and-sorting.md)
- [API versioning](m07-http-rest-api-design/api-versioning.md)

### ASP.NET Core Fundamentals ([Module 8](m08-aspnet-core-fundamentals/README.md))

- [The request pipeline and middleware](m08-aspnet-core-fundamentals/request-pipeline-and-middleware.md)
- [Service lifetimes: transient, scoped, and singleton](m08-aspnet-core-fundamentals/service-lifetimes.md)
- [Global exception handling and ProblemDetails](m08-aspnet-core-fundamentals/global-exception-handling-and-problemdetails.md)
- [HttpClientFactory](m08-aspnet-core-fundamentals/httpclientfactory.md)
- [Controllers vs. minimal APIs](m08-aspnet-core-fundamentals/controllers-versus-minimal-apis.md)

### Relational Databases and SQL ([Module 9](m09-relational-databases-and-sql/README.md))

- [Inner and outer joins](m09-relational-databases-and-sql/joins.md)
- [Grouping and aggregation](m09-relational-databases-and-sql/grouping-and-aggregation.md)
- [Window functions](m09-relational-databases-and-sql/window-functions.md)
- [NULL handling and three-valued logic](m09-relational-databases-and-sql/null-handling.md)
- [Transactions and ACID](m09-relational-databases-and-sql/transactions-and-acid.md)
- [Indexes](m09-relational-databases-and-sql/indexes.md)
- [SQL injection prevention](m09-relational-databases-and-sql/sql-injection-prevention.md)

### Entity Framework Core ([Module 10](m10-entity-framework-core/README.md))

- [DbContext lifetime, thread safety, and pooling](m10-entity-framework-core/dbcontext-lifetime-and-pooling.md)
- [N+1 queries](m10-entity-framework-core/n-plus-one-queries.md)
- [Eager, explicit, and lazy loading](m10-entity-framework-core/loading-strategies.md)
- [AsNoTracking](m10-entity-framework-core/asnotracking.md)
- [Optimistic concurrency](m10-entity-framework-core/optimistic-concurrency.md)
- [Split vs. single queries](m10-entity-framework-core/split-vs-single-queries.md)

### Testing and Testability ([Module 11](m11-testing-and-testability/README.md))

- [Testing behavior, not implementation](m11-testing-and-testability/testing-behavior-not-implementation.md)
- [Mocks, stubs, and fakes](m11-testing-and-testability/mocks-stubs-and-fakes.md)
- [What should and should not be mocked](m11-testing-and-testability/what-to-mock.md)
- [ASP.NET Core integration testing and WebApplicationFactory](m11-testing-and-testability/aspnet-core-integration-testing.md)
- [Reliable and deterministic tests](m11-testing-and-testability/reliable-and-deterministic-tests.md)

### Practical Interview Exercises ([Module 16](m16-practical-interview-exercises/README.md))

- [Debugging broken code](m16-practical-interview-exercises/debugging-broken-code.md)
- [Explaining a code snippet](m16-practical-interview-exercises/explaining-a-code-snippet.md)
- [Reviewing a pull request](m16-practical-interview-exercises/reviewing-a-pull-request.md)
- [Finding EF Core performance issues](m16-practical-interview-exercises/finding-ef-core-performance-issues.md)
- [Async/concurrency problem analysis](m16-practical-interview-exercises/async-concurrency-problem-analysis.md)

## Strengthen — If You Have More Time

Load-bearing topics from High-priority modules — not the full module, just the parts most likely to anchor a design or memory-management discussion.

### Object-Oriented Design and Maintainable Code ([Module 4](m04-oop-design/README.md))

- [SOLID principles](m04-oop-design/solid-principles.md)
- [Dependency inversion and dependency injection](m04-oop-design/dependency-inversion-and-injection.md)
- [Repository pattern](m04-oop-design/repository.md)
- [Composition versus inheritance](m04-oop-design/composition-versus-inheritance.md)
- [Interfaces versus abstract classes](m04-oop-design/interfaces-versus-abstract-classes.md)
- [Coupling and cohesion](m04-oop-design/coupling-and-cohesion.md)

### Exceptions, Resources, and Memory Management ([Module 5](m05-exceptions-resources-memory/README.md))

- [Exception handling and design](m05-exceptions-resources-memory/exception-handling-and-design.md)
- [Resource management and disposal](m05-exceptions-resources-memory/resource-management-and-disposal.md)
- [Garbage collection and object lifetime](m05-exceptions-resources-memory/gc-and-object-lifetime.md)
- [Boxing and allocation pressure](m05-exceptions-resources-memory/boxing-and-allocation-pressure.md)

### Application Security ([Module 12](m12-application-security/README.md))

- [Authentication vs. authorization](m12-application-security/authentication-vs-authorization.md)
- [JWT structure and validation](m12-application-security/jwt-structure-and-validation.md)
- [Password storage](m12-application-security/password-storage.md)
- [CORS vs. CSRF](m12-application-security/cors-vs-csrf.md)
- [XSS and output encoding](m12-application-security/xss-and-output-encoding.md)
- [Mass assignment / over-posting](m12-application-security/mass-assignment-and-over-posting.md)
- [OWASP Top 10 and OWASP API Security Top 10](m12-application-security/owasp-top-10-and-api-security-top-10.md)

### Performance, Diagnostics, and Observability ([Module 13](m13-performance-diagnostics-observability/README.md))

- [Measuring before optimizing](m13-performance-diagnostics-observability/measure-before-optimizing.md)
- [Database query performance and N+1 problems](m13-performance-diagnostics-observability/database-query-performance-and-n-plus-one.md)
- [Caching strategies: in-memory vs. distributed](m13-performance-diagnostics-observability/caching-strategies.md)
- [Metrics and tracing](m13-performance-diagnostics-observability/metrics-and-tracing.md)
- [Basic resilience: timeouts, retries, and circuit breakers](m13-performance-diagnostics-observability/resilience-timeouts-retries-circuit-breakers.md)

### Architecture and System Design Fundamentals ([Module 14](m14-architecture-and-system-design/README.md))

- [Layered architecture, separation of domain/infrastructure, and dependency direction](m14-architecture-and-system-design/layered-architecture-and-boundaries.md)
- [Synchronous vs. asynchronous communication (system level)](m14-architecture-and-system-design/synchronous-versus-asynchronous-communication.md)
- [Monolith vs. microservices, and microservice trade-offs](m14-architecture-and-system-design/monolith-versus-microservices.md)
- [Distributed transactions, eventual consistency, and idempotent message handling](m14-architecture-and-system-design/distributed-transactions-and-eventual-consistency.md)
- [Avoiding unnecessary architecture](m14-architecture-and-system-design/avoiding-unnecessary-architecture.md)

### Development Workflow and Delivery Fundamentals ([Module 15](m15-development-workflow-and-delivery/README.md))

- [Branching and pull requests](m15-development-workflow-and-delivery/branching-and-pull-requests.md)
- [Code review](m15-development-workflow-and-delivery/code-review.md)
- [CI/CD concepts, and running tests in a pipeline](m15-development-workflow-and-delivery/ci-cd-concepts-and-pipeline-testing.md)
- [Database migrations during deployment](m15-development-workflow-and-delivery/database-migrations-during-deployment.md)
- [Basic release and rollback thinking](m15-development-workflow-and-delivery/release-and-rollback-thinking.md)

### Experience and Behavioural Evidence ([Module 17](m17-experience-and-behavioural-evidence/README.md))

- [The STAR method, and why technical grounding is what makes it work](m17-experience-and-behavioural-evidence/star-method-and-technical-grounding.md)
- [A difficult bug](m17-experience-and-behavioural-evidence/a-difficult-bug.md)
- [A production or test failure](m17-experience-and-behavioural-evidence/a-production-or-test-failure.md)
- [Missing a deadline or making a mistake](m17-experience-and-behavioural-evidence/missing-a-deadline-or-making-a-mistake.md)
- [Making a design trade-off](m17-experience-and-behavioural-evidence/making-a-design-trade-off.md)

## Pending — Not Yet Written

Every module through Module 17 is now complete. Only Module 1 (.NET Platform and Development Model) remains an outline-only entry in the main roadmap — deliberately last in the queue, priority Medium, and not expected to need a full file treatment the way the other modules did.

## If You Truly Have One Day

Read the **Core** section only, and skim the "Common Mistakes" and "Code Prediction" section of each — that's where most interview follow-ups actually come from.
