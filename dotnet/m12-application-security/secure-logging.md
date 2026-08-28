# Secure Logging

## Definition

Secure logging means capturing enough operational detail to diagnose issues without ever writing sensitive data — passwords, tokens, full card numbers, personal data — into logs, which are often less tightly access-controlled, more widely retained, and more broadly readable (by developers, log-aggregation tools, third-party services) than the primary application data store itself.

```csharp
// WRONG: the token itself is now sitting in every log aggregation system this flows through
_logger.LogInformation("Authenticated request with token {Token}", bearerToken);

// RIGHT: log identifying, non-sensitive context instead
_logger.LogInformation("Authenticated request for user {UserId}", userId);
```

## Alternatives & Trade-offs

Logging everything, including full request/response bodies, makes debugging maximally easy in the moment but risks capturing sensitive data indefinitely in a system with different (often weaker) access controls than the primary database. Logging deliberately less — specifically excluding known-sensitive fields — trades some debugging convenience for a meaningfully smaller exposure surface if the logging system itself is ever breached or over-broadly accessed.

## How It Works

### What routinely ends up in logs by accident

```csharp
// Logging an entire request object for debugging convenience — easy to do, easy to regret
_logger.LogInformation("Received request: {@Request}", request); // if `request` has a Password field, it's now logged in full
```

Structured logging (Module 8) makes this mistake especially easy — logging an entire object is one line of code, and it's simple to forget that object might carry a sensitive field months after the logging statement was written.

### Redacting or excluding sensitive fields deliberately

```csharp
public class LoginRequest
{
    public string Email { get; set; } = "";
    [JsonIgnore] // or a custom attribute/converter specifically for logging serialization
    public string Password { get; set; } = "";
}

_logger.LogInformation("Login attempt for {Email}", request.Email); // deliberately log only the safe field
```

### Logging identifiers, not the data itself

```csharp
// Instead of logging the actual sensitive value, log an identifier that lets you look it up
// through a properly access-controlled system if genuinely needed for investigation
_logger.LogInformation("Payment processed for order {OrderId}", order.Id); // not the card number, not the amount details if sensitive
```

### Log access control and retention matter as much as what's logged

```
Even carefully-redacted logs can still leak sensitive metadata (IP addresses, user identifiers,
behavioral patterns) — log storage itself should have appropriate access controls and retention
limits, treated as seriously as the primary data store, not as a permissive dumping ground.
```

### Audit logging — a distinct, deliberate category

```csharp
// Audit logs (who did what, when) are often a distinct stream from general diagnostic logs,
// with different retention and access requirements — e.g., "Admin {UserId} deleted Order {OrderId} at {Timestamp}"
_auditLogger.LogInformation("User {UserId} deleted Order {OrderId}", currentUserId, order.Id);
```

## Application

Review logging statements specifically for sensitive fields before they ship, not after an incident reveals them. Log identifiers and context rather than raw sensitive values. Treat log storage's access control and retention policy with the same seriousness as the primary database, and maintain audit logs as a distinct, more carefully governed category where compliance or security investigation needs it.

## Common Mistakes

- Logging an entire request/response object for debugging convenience, capturing any sensitive fields it happens to contain.
- Logging tokens, passwords, or full card numbers directly, even temporarily "for debugging," and forgetting to remove it before shipping.
- Assuming log storage is inherently as secure as the primary database, when it's often more broadly accessible and more loosely retained.
- Not distinguishing audit logs (who did what) from general diagnostic logs, missing the different access-control and retention requirements each often needs.

## Common Interview Questions

### Basic
- Why is it risky to log an entire request object without reviewing its fields first?
- What kinds of data should never appear in application logs?

### Intermediate
- How would you prevent a sensitive field from accidentally being logged via structured logging's object-serialization convenience?
- Why should log storage's access control be treated as seriously as the primary database's?

### Advanced
- How would you audit an existing codebase for logging statements that might be capturing sensitive data?
- What's the difference in purpose and governance between audit logs and general diagnostic logs?

### Follow-up Questions
- Does redacting a field from a DTO's logging output also protect it from appearing in a stack trace or exception message elsewhere?
- Should log retention be time-limited, and why might that matter for sensitive-adjacent data even after redaction?

### Code Prediction
Given `_logger.LogInformation("Received request: {@Request}", request)` where `LoginRequest` includes a `Password` property with no `[JsonIgnore]` or equivalent exclusion, what ends up in the log output for every login attempt?

## Practical Tasks

- Audit a set of logging statements in a hypothetical codebase for sensitive-field exposure and fix the ones found.
- Design a redaction/exclusion strategy for a DTO with mixed sensitive and non-sensitive fields, ensuring it applies consistently across serialization and logging.
- Design a separate audit-logging stream for a specific set of sensitive administrative actions, distinct from general diagnostic logs.

## Readiness Criteria

Recognize and prevent sensitive data from reaching logs, including via structured-logging object serialization, and design appropriate governance for audit logs distinct from general diagnostics.

## References

### Microsoft Learn

- [Logging in .NET Core and ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/logging/)

### Other

- [OWASP: Logging Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Logging_Cheat_Sheet.html)
