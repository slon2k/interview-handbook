# Authentication vs. Authorization

## Definition

**Authentication** answers "who are you?" — verifying an identity, typically via credentials, a token, or a certificate. **Authorization** answers "what are you allowed to do?" — deciding whether an already-authenticated identity may perform a specific action. They're sequential and distinct: a request must be authenticated before authorization can meaningfully apply, which is exactly why `UseAuthentication()` must run before `UseAuthorization()` in the ASP.NET Core middleware pipeline (Module 8).

```csharp
app.UseAuthentication(); // "who are you?" — populates HttpContext.User
app.UseAuthorization();  // "what can you do?" — checks HttpContext.User against policy requirements
```

## Alternatives & Trade-offs

There's no real alternative to having both — a system needs to know who's calling before it can decide what they're allowed to do. The design choice is really about mechanism: how identity is established (password, token, certificate, API key) and how permissions are modeled (roles, claims, policies — covered in the next topic), not whether authentication and authorization are needed at all.

## How It Works

### The status codes that map to each failure

```
401 Unauthorized — "I don't know who you are" (authentication failed or missing entirely)
403 Forbidden     — "I know who you are, but you can't do this" (authentication succeeded, authorization failed)
```

This is the same `401`/`403` distinction introduced in Module 7 — worth restating here because it's the single most common way this topic gets tested in interviews, and getting the two backwards is a frequent, telling mistake.

### Authentication populates the identity; authorization consumes it

```csharp
// After UseAuthentication() runs successfully:
HttpContext.User.Identity.IsAuthenticated // true
HttpContext.User.Identity.Name            // "alice@example.com"

// Authorization then checks THIS identity against a requirement:
[Authorize(Roles = "Admin")]
public IActionResult DeleteOrder(int id) { }
```

If authentication fails or is missing, `HttpContext.User` represents an anonymous identity — authorization checks against it will (correctly) fail, since there's no identity to evaluate permissions for.

### Being authenticated doesn't imply being authorized

```csharp
[Authorize] // requires ANY authenticated user
public IActionResult ViewOrder(int id) { }

[Authorize(Roles = "Admin")] // requires an authenticated user who is ALSO an Admin
public IActionResult DeleteOrder(int id) { }
```

A fully logged-in, legitimately authenticated user can still be correctly denied access to a specific action — these are two independent checks, not one graduated scale.

## Application

Design every protected endpoint with both questions answered explicitly: how is the caller's identity established (authentication), and what specifically must be true about that identity to allow this particular action (authorization). Return `401` for missing/invalid identity and `403` for a valid identity lacking sufficient permission — never conflate the two in either code or in an interview answer.

## Common Mistakes

- Confusing `401` and `403` — treating any access denial as "unauthorized" regardless of whether the caller was actually authenticated.
- Skipping authentication middleware (or misordering it after authorization) and being confused why every authorization check fails.
- Assuming "requires login" and "requires permission" are the same check, when authorization can still deny a legitimately authenticated user.
- Building custom ad hoc authentication logic instead of using the framework's established authentication/authorization pipeline, missing edge cases the framework already handles correctly.

## Common Interview Questions

### Basic
- What's the difference between authentication and authorization?
- Which HTTP status code corresponds to each failure?

### Intermediate
- Why must authentication middleware run before authorization middleware in ASP.NET Core?
- Can an authenticated user still fail an authorization check? Give an example.

### Advanced
- How would you design an API where authentication is handled by an external identity provider but authorization decisions are made locally, based on claims issued by that provider?
- What's the risk of an authorization check running before `HttpContext.User` has been populated by authentication?

### Follow-up Questions
- Does every endpoint need authentication, even ones that don't need authorization?
- Is it possible for an endpoint to require authorization but not authentication?

### Code Prediction
Given `app.UseAuthorization(); app.UseAuthentication();` (the wrong order), what happens when a request to an `[Authorize]`-decorated endpoint arrives with a valid bearer token?

## Practical Tasks

- Configure an endpoint requiring authentication only (`[Authorize]`) and another requiring a specific role, and verify the correct status code for each failure case.
- Reproduce the middleware-ordering bug by swapping `UseAuthentication`/`UseAuthorization` and observe the failure.
- Design the authentication and authorization requirements for a small API with public, logged-in-only, and admin-only endpoints.

## Readiness Criteria

Distinguish authentication and authorization precisely, correctly map each failure to its status code, and design endpoint-level requirements combining both correctly.

## References

### Microsoft Learn

- [Authentication and authorization in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/)
