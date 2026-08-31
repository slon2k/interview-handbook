# Reviewing a Pull Request

## What This Assesses

Can you evaluate someone else's code critically and constructively — spotting real correctness/design issues, not just style nits — the way Module 15's code-review content described? This exercise also tests communication: how you *phrase* feedback matters as much as whether you find the issue.

## Format and Time Expectations

Usually a diff or a small set of changed files, with a prompt to "review this as if it were a real PR." Talk through what you'd comment on, and how you'd phrase each comment.

## Exercise 1: A Repository Method with Several Issues

**Problem:** Review this PR diff.

```csharp
public class OrderRepository
{
    private AppDbContext _context = new AppDbContext(); // registered as Scoped elsewhere, but instantiated directly here

    public Order GetOrder(int id)
    {
        return _context.Orders.Where(o => o.Id == id).FirstOrDefault();
    }

    public void UpdateStatus(int id, string status)
    {
        var order = GetOrder(id);
        order.Status = status;
        _context.SaveChanges();
    }
}
```

**What a strong answer demonstrates:** Flagging, with specific, actionable phrasing: (1) `new AppDbContext()` bypasses DI entirely, losing the configured connection string and lifetime management (Module 10) — should be constructor-injected; (2) `GetOrder` followed by `SaveChanges` isn't async, blocking a thread unnecessarily for I/O (Module 6); (3) no null check after `GetOrder` before setting `.Status`, risking `NullReferenceException`; (4) `status` as a raw `string` instead of an enum, risking typos with no compile-time safety.

**Common mistakes:** Only commenting on the `new AppDbContext()` issue (the most obvious one) and missing the async and null-safety issues, or commenting on all of them but phrasing it as "this is wrong" without explaining the specific risk each represents.

## Exercise 2: An Endpoint with a Security Gap

**Problem:** Review this PR diff.

```csharp
[Authorize]
[HttpGet("orders/{id}")]
public async Task<IActionResult> GetOrder(int id)
{
    var order = await _repository.GetByIdAsync(id);
    return Ok(order);
}
```

**What a strong answer demonstrates:** Recognizing Broken Object-Level Authorization (Module 12) — `[Authorize]` confirms the caller is *authenticated*, but nothing checks whether *this specific caller* should see *this specific order*. A strong review comment would name the specific risk (any logged-in user can view any order by ID) and suggest a concrete fix (check `order.CustomerId == currentUserId`, or `Forbid()`/`NotFound()` if not).

**Common mistakes:** Approving the PR because `[Authorize]` is present, without checking whether authorization is actually scoped to the right resource — exactly the gap this exercise is designed to surface.

## Exercise 3: A Test That Doesn't Actually Test Anything

**Problem:** Review this PR diff, which adds a new test alongside a bug fix.

```csharp
[Fact]
public void CalculateDiscount_Works()
{
    var order = new Order(subtotal: 100m, discountRate: 0.1m);
    var result = order.CalculateTotal();
    Assert.True(result >= 0); // the only assertion
}
```

**What a strong answer demonstrates:** Recognizing this test would pass even if the discount calculation were completely wrong (Module 11's `code-coverage.md` — high coverage, low actual verification) and asking for a specific expected-value assertion (`Assert.Equal(90m, result)`) instead of a vague, always-true-ish check.

**Common mistakes:** Approving the PR because "there's a test now," without checking whether the test actually verifies the behavior it claims to.

## Readiness Criteria

Identify correctness, security, and design issues (not just style), phrase feedback specifically and actionably rather than vaguely, and avoid rubber-stamping a PR based on surface signals (a test exists, an `[Authorize]` attribute is present) without verifying they actually do their job.

## References

- [Code review (Module 15)](../m15-development-workflow-and-delivery/code-review.md)
- [OWASP Top 10 and API Security Top 10 (Module 12)](../m12-application-security/owasp-top-10-and-api-security-top-10.md)
- [Code coverage and its limitations (Module 11)](../m11-testing-and-testability/code-coverage.md)
