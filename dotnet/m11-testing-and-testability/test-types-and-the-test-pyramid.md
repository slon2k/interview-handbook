# Test Types and the Test Pyramid

## Definition

**Unit tests** verify a single unit of logic (a method, a class) in isolation, typically with dependencies faked. **Integration tests** verify multiple real components working together (a service plus a real database). **Component tests** verify one deployable component's behavior end-to-end but still in isolation from other services. **End-to-end (E2E) tests** exercise the whole system through its real interfaces, as close to production as practical. The **test pyramid** is a shorthand for the recommended balance: many fast unit tests, fewer integration tests, fewer still E2E tests.

```
        /\
       /E2E\        <- few, slow, expensive, high confidence
      /------\
     /Integr. \     <- some, moderate speed/cost
    /----------\
   /   Unit     \   <- many, fast, cheap, isolated
  /--------------\
```

## Alternatives & Trade-offs

Unit tests run in milliseconds and pinpoint failures precisely, but can't catch problems that only appear when real components interact (a wrong SQL mapping, a misconfigured DI registration). E2E tests catch those integration problems and give the highest confidence the system actually works, but are slow, flaky, and expensive to maintain, and a failure often doesn't point directly at the cause. The pyramid shape is a deliberate cost/confidence trade-off: get most of your confidence cheaply from unit tests, and reserve the expensive, slow tests for what only they can catch.

## How It Works

### The same behavior, tested at different levels

```csharp
// Unit test: OrderService in isolation, IOrderRepository faked
[Fact]
public void CalculateTotal_AppliesDiscount()
{
    var service = new OrderService(new FakeOrderRepository());
    var total = service.CalculateTotal(order);
    Assert.Equal(90m, total);
}

// Integration test: OrderService with a REAL database-backed repository
[Fact]
public async Task SaveOrder_PersistsToRealDatabase()
{
    using var context = CreateRealTestDbContext();
    var repository = new EfOrderRepository(context);
    await repository.AddAsync(order);
    Assert.NotNull(await context.Orders.FindAsync(order.Id));
}

// End-to-end test: the full API, over real HTTP, against a running instance
[Fact]
public async Task PostOrder_ReturnsCreatedOrder()
{
    var response = await _httpClient.PostAsJsonAsync("/orders", newOrderRequest);
    response.EnsureSuccessStatusCode();
}
```

### "Practical variations" on the pyramid

The strict pyramid shape is a starting heuristic, not a law — some teams (especially those with a strong, well-designed API surface and less complex UI logic) invert part of it toward more integration/component tests and fewer, more targeted unit tests ("the testing trophy" is one commonly cited variation). The right shape depends on where your system's actual risk lives.

## Application

Write the bulk of tests as fast unit tests for business logic and edge cases. Add integration tests specifically for the seams where real components could disagree with your assumptions (ORM mappings, serialization, external API contracts). Reserve E2E tests for the critical user-facing flows that justify their cost and flakiness risk, not for exhaustively re-testing logic already covered at the unit level.

## Common Mistakes

- Treating the pyramid shape as a rigid rule rather than a starting heuristic, when a specific system's risk profile might justify a different balance.
- Writing E2E tests for logic that's already well-covered by fast unit tests, adding slow, flaky duplication instead of new confidence.
- Skipping integration tests entirely and relying only on unit tests with fakes, missing real bugs at the actual integration seams (a fake repository can't catch a real SQL mapping mistake).
- Confusing "integration test" with "end-to-end test" — they test different scopes, and conflating them leads to unclear expectations about what a given test suite actually verifies.

## Common Interview Questions

### Basic
- What are the main types of automated tests, and how do they differ in scope?
- What does the test pyramid recommend, and why?

### Intermediate
- Why do E2E tests provide the highest confidence but also the highest cost?
- What's the practical difference between an integration test and a component test?

### Advanced
- When would a team deliberately deviate from the classic pyramid shape, and what would justify it?
- How would you decide, for a specific piece of business logic, whether it needs unit, integration, or E2E coverage — or several?

### Follow-up Questions
- Can a single feature reasonably need tests at all four levels?
- Does having a large E2E test suite reduce the need for unit tests?

### Code Prediction
Given the three tests above for essentially the same order-creation behavior, if a bug is introduced in `OrderService.CalculateTotal`, which of the three tests fails first and fastest, and which (if any) might still pass despite the bug?

## Practical Tasks

- Write the same piece of business logic tested at both the unit level (fake dependencies) and the integration level (real database).
- Classify a list of existing tests in a hypothetical codebase by their actual level (unit/integration/component/E2E), correcting any mislabeled ones.
- Propose a test-level balance for a specific feature, justifying the choice against the classic pyramid shape.

## Readiness Criteria

Distinguish the test levels precisely, explain the cost/confidence trade-off the pyramid represents, and justify a test-level balance for a given feature rather than defaulting to one level for everything.

## References

### Microsoft Learn

- [Unit testing best practices](https://learn.microsoft.com/dotnet/core/testing/unit-testing-best-practices)

### Other

- [Martin Fowler: TestPyramid](https://martinfowler.com/bliki/TestPyramid.html)
