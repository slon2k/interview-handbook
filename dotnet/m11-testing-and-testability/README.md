# Module 11 - Testing and Testability

**Status:** Complete  
**Priority:** Critical  
**Prerequisites:** [C# Language and Type System](../m02-csharp-language/README.md), [Object-Oriented Design and Maintainable Code](../m04-oop-design/README.md)

## Scope

This module covers both testing concepts (what makes a good test, the test pyramid, behavior vs. implementation) and actual implementation (xUnit/NUnit, mocking libraries, `WebApplicationFactory`, Testcontainers). It builds directly on Module 4's design principles — testable code and good tests are the same discipline seen from two sides — and Module 6/7/8/10's technical foundations, since testing async code, HTTP clients, and EF Core-backed repositories each has its own specific pattern.

## Learning Outcomes

By the end of this module, you should be able to:

- Explain the test pyramid and justify a test-level balance for a given feature.
- Write clear, well-structured, behavior-focused tests using consistent naming and AAA structure.
- Use parameterized tests, fixtures, and test data builders to keep test suites maintainable.
- Choose correctly between mocks, stubs, and fakes, and judge what should and shouldn't be mocked.
- Test asynchronous code, exceptions, and `HttpClient`-dependent code correctly.
- Write ASP.NET Core integration tests with `WebApplicationFactory` and database integration tests with Testcontainers.
- Use code coverage as a gap-finder rather than a target, and eliminate common sources of test flakiness.
- Configure a CI pipeline that actually enforces test results as a merge-blocking gate.

## Topics

### 1. Testing Concepts

- [Test types and the test pyramid](test-types-and-the-test-pyramid.md)
- [Arrange-Act-Assert and test naming](arrange-act-assert-and-test-naming.md)
- [Testing behavior, not implementation](testing-behavior-not-implementation.md)

### 2. Writing Tests

- [Test frameworks and assertions (xUnit / NUnit)](test-frameworks-and-assertions.md)
- [Parameterized tests](parameterized-tests.md)
- [Test fixtures and shared context](test-fixtures-and-shared-context.md)
- [Test data builders](test-data-builders.md)

### 3. Test Doubles

- [Mocks, stubs, and fakes](mocks-stubs-and-fakes.md)
- [What should and should not be mocked](what-to-mock.md)

### 4. Testing Specific Scenarios

- [Testing asynchronous code](testing-async-code.md)
- [Testing exceptions](testing-exceptions.md)
- [Testing HttpClient](testing-httpclient.md)

### 5. Integration Testing

- [ASP.NET Core integration testing and WebApplicationFactory](aspnet-core-integration-testing.md)
- [Database integration tests and Testcontainers](database-integration-tests.md)

### 6. Test Suite Health

- [Code coverage and its limitations](code-coverage.md)
- [Reliable and deterministic tests](reliable-and-deterministic-tests.md)
- [Tests in CI](tests-in-ci.md)

## Scope Boundaries

- General OOP design principles that make code testable in the first place (SOLID, dependency inversion, the repository pattern) belong in [Module 4 - Object-Oriented Design and Maintainable Code](../m04-oop-design/README.md); this module assumes that foundation and focuses on writing the tests themselves.
- Async/await mechanics in general belong in [Module 6 - Asynchronous Programming and Concurrency](../m06-async-concurrency/README.md); this module covers only the test-specific patterns for exercising async code.
- `HttpClientFactory` and typed HTTP clients in general belong in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md); this module covers only how to test code that depends on them.
- EF Core testing fidelity trade-offs (in-memory provider vs. SQLite vs. Testcontainers) are introduced in [Module 10 - Entity Framework Core](../m10-entity-framework-core/README.md); this module's database-integration-tests topic builds directly on that.
- CI/CD pipeline mechanics beyond running and gating tests belong in [Module 15 - Development Workflow and Delivery Fundamentals](../m15-development-workflow-and-delivery/README.md).

## Suggested Learning Sequence

1. Test types, the test pyramid, AAA, naming, and behavior-vs-implementation.
2. Test frameworks, parameterized tests, fixtures, and test data builders.
3. Mocks, stubs, fakes, and what should and shouldn't be mocked.
4. Testing async code, exceptions, and `HttpClient`.
5. ASP.NET Core and database integration testing.
6. Code coverage, test reliability, and CI enforcement.

## Practical Deliverables

- Write a small unit test suite for a business-logic class using AAA structure, clear naming, parameterized tests, and a test data builder.
- Refactor a mock-verification-heavy test into one asserting on outcomes, and identify a case where a fake would be more appropriate than a mock.
- Write tests for async, exception-throwing, and `HttpClient`-dependent code using the correct pattern for each.
- Set up a `WebApplicationFactory`-based integration test substituting the database with a fake repository.
- Set up a Testcontainers-based database integration test running real migrations.
- Diagnose and fix a deliberately introduced flaky test (time-dependent, order-dependent, or timing-assumption-based).
- Configure a CI pipeline step running the test suite as a required, merge-blocking check.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and framework familiarity.
- Intermediate questions involving common usage and design trade-offs.
- Advanced questions involving test-suite design decisions and diagnosing subtle failures.
- Follow-up questions that test precise understanding rather than memorized definitions.
- Code-prediction questions grounded in concrete test examples, since this module produces some of the most practical "what would this test actually catch" interview questions.

## References

### Microsoft Learn

- [Testing overview](https://learn.microsoft.com/dotnet/core/testing/)
- [Unit testing best practices](https://learn.microsoft.com/dotnet/core/testing/unit-testing-best-practices)
- [Integration tests in ASP.NET Core](https://learn.microsoft.com/aspnet/core/test/integration-tests)

### Other

- [xUnit.net documentation](https://xunit.net/)
- [Testcontainers for .NET](https://dotnet.testcontainers.org/)
