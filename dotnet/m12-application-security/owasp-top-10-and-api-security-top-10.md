# OWASP Top 10 and OWASP API Security Top 10

## Definition

The **OWASP Top 10** is a periodically-updated, community-curated list of the most critical web application security risks, used widely as a shared vocabulary and baseline checklist across the industry. The **OWASP API Security Top 10** is a parallel, API-specific list, since APIs have risk patterns (broken object-level authorization, excessive data exposure) that don't map as directly onto the general web-application list. Neither is a complete security program on its own — they're a well-known, high-value baseline.

```
OWASP Top 10 (illustrative categories, general web apps):
  Broken Access Control, Cryptographic Failures, Injection, Insecure Design, ...

OWASP API Security Top 10 (illustrative categories, APIs specifically):
  Broken Object Level Authorization, Broken Authentication, Broken Object Property Level Authorization, ...
```

## Alternatives & Trade-offs

Treating the OWASP lists as a checklist to memorize verbatim misses their actual value — the specific numbered rankings and exact category names change between revisions, so memorizing the current list is a weaker skill than understanding the underlying *patterns* well enough to recognize them regardless of what they're currently called. The lists are most useful as a shared vocabulary for discussing risk with a team, and as a prompt to check whether a design has addressed each category, not as an exhaustive specification.

## How It Works

### Broken Object-Level Authorization (BOLA) — the API-specific risk most worth knowing well

```csharp
// VULNERABLE: authenticates the caller, but never checks whether THIS caller should access THIS specific order
[Authorize]
[HttpGet("orders/{id}")]
public async Task<IActionResult> GetOrder(int id)
{
    var order = await _repository.GetByIdAsync(id); // returns ANY order matching the ID, regardless of who's asking
    return Ok(order);
}

// FIXED: verifies the specific resource belongs to (or is otherwise accessible by) the caller
[Authorize]
[HttpGet("orders/{id}")]
public async Task<IActionResult> GetOrder(int id)
{
    var order = await _repository.GetByIdAsync(id);
    if (order is null || order.CustomerId != GetCurrentUserId()) return NotFound(); // or Forbid(), see the 403-vs-404 discussion in Module 7
    return Ok(order);
}
```

This is precisely the resource-based authorization gap from `claims-roles-and-policies.md` — knowing the API list's specific name for it (BOLA) is less important than recognizing the pattern: authentication succeeded, but object-level authorization was never actually checked.

### Excessive Data Exposure — returning more than the client needs

```csharp
// Returns the FULL entity, including fields the client never needed and shouldn't see (e.g., internal notes, cost basis)
return Ok(order); // the entire Order entity, as-is

// Returns only what the response actually needs to expose — connects directly to Module 10's projection topic
return Ok(new OrderResponseDto { Id = order.Id, Total = order.Total, Status = order.Status });
```

### How the general list maps onto concepts already covered elsewhere in this handbook

```
Injection                      -> Module 9 (SQL injection), this module's input-validation topic
Cryptographic Failures         -> this module's password-storage, HTTPS, and secret-management topics
Broken Access Control          -> this module's authentication-vs-authorization and claims-roles-policies topics
Security Misconfiguration      -> this module's HTTPS, CORS-vs-CSRF, and secret-management topics
Vulnerable and Outdated Components -> keeping NuGet dependencies patched (a workflow/process concern, Module 15)
```

Most of what the OWASP lists describe is content this module (and Module 9) already covers by name and mechanism — the lists are valuable primarily as an organizing checklist and shared vocabulary, not as new material.

## Application

Use the OWASP Top 10 and API Security Top 10 as a periodic design-review checklist — "have we addressed each of these categories for this feature?" — and as shared vocabulary when discussing risk with a team, rather than treating the specific current rankings as something to memorize. Recognize the underlying patterns (especially BOLA, since it's specific to APIs and easy to miss) regardless of how a given revision names them.

## Common Mistakes

- Memorizing the current numbered list and category names verbatim, rather than understanding the underlying vulnerability patterns, which is what actually transfers when the list is revised.
- Treating "we checked against OWASP Top 10" as equivalent to "our application is secure," when the list is a baseline, not a completeness guarantee.
- Missing Broken Object-Level Authorization specifically, since it's easy to implement authentication and general role-based authorization correctly while still never checking whether a specific caller should access a specific resource.
- Not connecting the OWASP categories back to concrete mechanisms already covered elsewhere (injection to parameterized queries, cryptographic failures to password hashing) — treating them as a separate body of knowledge rather than an organizing lens on things already understood.

## Common Interview Questions

### Basic
- What is the OWASP Top 10, and what's it typically used for?
- What's the difference between the general OWASP Top 10 and the OWASP API Security Top 10?

### Intermediate
- What is Broken Object-Level Authorization, and how does it differ from a general authentication or role-based authorization failure?
- Why is memorizing the exact current list less valuable than understanding the underlying patterns?

### Advanced
- How would you run a design review for a new API feature using the OWASP API Security Top 10 as a structured checklist?
- How would you explain, to a team unfamiliar with the specific list, why BOLA is a particularly easy vulnerability to introduce even with authentication and RBAC already correctly implemented?

### Follow-up Questions
- Does passing an OWASP Top 10 checklist review guarantee an application has no security vulnerabilities?
- How often is the OWASP Top 10 revised, and does that matter for how it should be used?

### Code Prediction
Given the vulnerable `GetOrder` example above, if `[Authorize]` is present and the caller has a valid token, does the request succeed? What specific check is missing that would prevent one customer from viewing another customer's order by simply changing the `id` in the URL?

## Practical Tasks

- Run a design review of a small existing API against the OWASP API Security Top 10 categories, identifying at least one gap.
- Fix a Broken Object-Level Authorization vulnerability in a sample endpoint.
- Map each OWASP Top 10 category to a specific mechanism already covered elsewhere in this module or Module 9, building the "pattern, not the label" understanding.

## Readiness Criteria

Use the OWASP lists as a design-review lens rather than a memorized checklist, and specifically recognize and fix Broken Object-Level Authorization, since it's the API-specific risk most likely to slip through otherwise-correct authentication and role-based authorization.

## References

### Other

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x00-header/)
