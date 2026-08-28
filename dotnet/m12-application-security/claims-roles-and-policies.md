# Claims, Roles, and Policies

## Definition

A **claim** is a key-value statement about an identity ("email: alice@example.com", "department: Engineering"), issued by whoever authenticated the user. A **role** is a specific, common kind of claim representing a named group membership ("Admin", "Manager"). A **policy** is a named, reusable authorization rule — potentially combining multiple claims and custom logic — that can be more expressive than role checks alone.

```csharp
[Authorize(Roles = "Admin")]                    // role-based
[Authorize(Policy = "MustBeOver18")]              // policy-based, can combine arbitrary claims/logic
```

## Alternatives & Trade-offs

Role-based checks are simple and sufficient when permissions map cleanly onto a small number of named groups. Claims-based/policy-based authorization is more flexible — it can express rules that don't fit a role hierarchy at all ("must be in the same department as the resource," "must be over 18," "must have completed onboarding") — at the cost of being less immediately readable than a simple role name and requiring policies to be registered explicitly.

## How It Works

### Claims — the raw facts about an identity

```csharp
var claims = new List<Claim>
{
    new(ClaimTypes.Name, "alice@example.com"),
    new(ClaimTypes.Role, "Admin"),
    new("department", "Engineering") // a custom claim type, not one of the built-in ClaimTypes
};
var identity = new ClaimsIdentity(claims, "MyAuthScheme");
```

Every piece of information ASP.NET Core's authorization system checks against ultimately comes from claims attached to the authenticated identity — roles are simply claims of type `ClaimTypes.Role` by convention.

### Roles — a specific, common claim type

```csharp
[Authorize(Roles = "Admin,Manager")] // user needs to be in AT LEAST ONE of these roles
public IActionResult ApproveExpense() { }
```

### Policies — named, reusable, and arbitrarily expressive

```csharp
builder.Services.AddAuthorization(options =>
{
    options.AddPolicy("MustBeOver18", policy =>
        policy.RequireAssertion(context =>
            context.User.HasClaim(c => c.Type == "birthdate") &&
            DateTime.Parse(context.User.FindFirst("birthdate")!.Value).AddYears(18) <= DateTime.UtcNow));
});

[Authorize(Policy = "MustBeOver18")]
public IActionResult PurchaseRestrictedItem() { }
```

A policy centralizes arbitrarily complex logic under one reusable name, rather than repeating a raw claims check inline at every endpoint that needs it.

### Custom authorization requirements and handlers — for logic beyond a simple assertion

```csharp
public class SameDepartmentRequirement : IAuthorizationRequirement { }

public class SameDepartmentHandler : AuthorizationHandler<SameDepartmentRequirement, Resource>
{
    protected override Task HandleRequirementAsync(AuthorizationHandlerContext context, SameDepartmentRequirement requirement, Resource resource)
    {
        if (context.User.FindFirst("department")?.Value == resource.Department)
            context.Succeed(requirement);
        return Task.CompletedTask;
    }
}
```

This is EF Core-style extensibility for authorization: a full handler class for resource-specific rules that a simple policy assertion can't express (it needs to know about the specific *resource* being accessed, not just the user).

## Application

Use roles for simple, small-cardinality group membership. Use policies once authorization logic needs to combine multiple claims, involve custom logic, or needs a descriptive, reusable name shared across several endpoints. Use resource-based authorization handlers when a decision depends on the specific object being accessed, not just the caller's identity.

## Common Mistakes

- Overusing role checks for logic that's really about a specific attribute (age, department, ownership) rather than group membership, forcing an awkward role explosion (`"Over18AndInEngineering"` as a fake role) instead of a proper policy.
- Hardcoding claim checks inline at every endpoint instead of centralizing them into a named, reusable policy.
- Confusing a role with a general claim — a role is just a claim of a specific, conventional type, not a fundamentally different mechanism.
- Not implementing resource-based authorization when a decision genuinely depends on the specific resource being accessed, instead trying to force that logic into a resource-agnostic policy.

## Common Interview Questions

### Basic
- What is a claim? What is a role, in relation to claims?
- What is an authorization policy?

### Intermediate
- When would you use a policy instead of a simple role check?
- How does resource-based authorization differ from a standard policy check?

### Advanced
- How would you design authorization for a rule that depends on the relationship between the caller and the specific resource being accessed (e.g., "only the order's owner or an admin can view it")?
- What's the trade-off between many small, specific roles versus a smaller number of roles combined with claims-based policies?

### Follow-up Questions
- Is a role technically a claim, or a separate mechanism?
- Can a single policy combine multiple claims and custom logic?

### Code Prediction
Given `[Authorize(Roles = "Admin,Manager")]`, does a user with only the `"Manager"` role pass this check? What if the user has neither role but does have a custom claim `"IsSupervisor": "true"` — would that satisfy this specific attribute as written?

## Practical Tasks

- Implement a named policy combining a custom claim check with a date comparison (e.g., age verification).
- Implement a resource-based authorization handler for a rule depending on the specific resource being accessed.
- Refactor a set of inline claims checks scattered across several endpoints into one reusable, named policy.

## Readiness Criteria

Distinguish claims, roles, and policies precisely, design policies for logic that doesn't fit a role hierarchy, and implement resource-based authorization when a decision depends on the specific object being accessed.

## References

### Microsoft Learn

- [Claims-based authorization](https://learn.microsoft.com/aspnet/core/security/authorization/claims)
- [Policy-based authorization](https://learn.microsoft.com/aspnet/core/security/authorization/policies)
- [Resource-based authorization](https://learn.microsoft.com/aspnet/core/security/authorization/resourcebased)
