# "What Would You Improve?" Questions

## What This Assesses

Given an existing, working piece of code or design, can you critically evaluate it and propose genuine improvements — not nitpicks — while being fair about what's actually fine as-is? This tests judgment and calibration as much as knowledge: knowing when *not* to suggest a change is as much a signal as knowing when to suggest one.

## Format and Time Expectations

Usually presented as "here's some real-ish code/design, what would you improve?" with no single correct answer expected — the interviewer is evaluating your reasoning and prioritization, not checking off a list.

## Exercise 1: A Working But Rough Service Class

**Problem:** "What would you improve about this class?"

```csharp
public class OrderService
{
    private readonly AppDbContext _context;
    public OrderService(AppDbContext context) => _context = context;

    public async Task<bool> PlaceOrder(int customerId, List<(string sku, int qty)> items)
    {
        var order = new Order { CustomerId = customerId };
        foreach (var (sku, qty) in items)
        {
            var product = _context.Products.FirstOrDefault(p => p.Sku == sku);
            order.Items.Add(new OrderItem { ProductId = product.Id, Quantity = qty });
        }
        _context.Orders.Add(order);
        await _context.SaveChangesAsync();
        return true;
    }
}
```

**What a strong answer demonstrates:** Prioritizing real issues over nitpicks: `FirstOrDefault` isn't async (blocking inside an otherwise-async method, Module 6) and isn't null-checked before `product.Id` is accessed (crash risk if a SKU doesn't exist); tuple parameters (`(string sku, int qty)`) are less discoverable/self-documenting than a proper request DTO (Module 14); the method always returns `true` regardless of outcome, giving the caller no way to distinguish success from a silent partial failure; no validation that `items` is non-empty. A strong answer explicitly *prioritizes* these (the null-check crash risk first, the always-`true` return second) rather than listing them with equal weight.

**Common mistakes:** Focusing only on naming/formatting nitpicks while missing the actual crash risk, or listing every possible change with no sense of relative importance.

## Exercise 2: A Reasonable Design That's Mostly Fine

**Problem:** "Here's our current order-processing design: a modular monolith, one shared database, synchronous calls between modules, no message queue anywhere. What would you improve?"

**What a strong answer demonstrates:** Recognizing that this description alone doesn't obviously need improving (Module 14's avoiding-unnecessary-architecture theme) — a modular monolith with synchronous calls is a perfectly reasonable default, not something to fix by default. A strong answer says so explicitly ("this sounds like a reasonable starting point; I'd want to know what specific pain point prompted this question before proposing changes") rather than inventing problems to sound thorough.

**Common mistakes:** Treating "what would you improve" as an instruction to find something wrong no matter what, proposing an unjustified architectural change (splitting into microservices, adding a message queue) to a design that's actually fine for its stated context.

## Exercise 3: A Test Suite With a Real Gap

**Problem:** "Our test suite for `OrderService` has 95% code coverage. What would you improve?"

**What a strong answer demonstrates:** Not being satisfied by the coverage number alone (Module 11's code-coverage content) — asking what the tests actually *assert*, since high coverage with weak assertions (`Assert.NotNull` everywhere, no specific expected-value checks) provides far less confidence than the number suggests. Proposing specific missing categories: exception/edge-case paths, concurrency-sensitive code if any exists, and whether integration tests (Module 11) exist alongside the unit tests, since 95% *unit* coverage says nothing about whether the pieces work together correctly.

**Common mistakes:** Treating "95% coverage" as sufficient evidence the test suite is good, without asking any follow-up question about what's actually being verified.

## Readiness Criteria

Prioritize genuine issues over surface-level nitpicks, explicitly rank concerns by severity when several exist, and recognize — and say so directly — when something presented for critique is actually reasonable as-is rather than manufacturing a problem to seem thorough.

## References

- [Guard clauses (Module 4)](../m04-oop-design/guard-clauses.md)
- [API and class design (Module 4)](../m04-oop-design/api-and-class-design.md)
- [Avoiding unnecessary architecture (Module 14)](../m14-architecture-and-system-design/avoiding-unnecessary-architecture.md)
- [Code coverage and its limitations (Module 11)](../m11-testing-and-testability/code-coverage.md)
