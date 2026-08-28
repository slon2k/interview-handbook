# Principle of Least Privilege, and Rate Limiting as a Security Control

## Definition

The **principle of least privilege** means every identity — a user, a service account, an API key, a database connection — should have only the permissions it actually needs to do its job, nothing more. **Rate limiting** (mechanically covered in Module 8) is also a security control from this angle: it limits how much damage a single compromised credential or abusive client can do, rather than only being about fair capacity sharing.

```csharp
// Least privilege: the application's database user should not have permission to DROP TABLE,
// even though the application code itself never intends to — the point is limiting blast radius
// if the application's credentials are ever compromised
GRANT SELECT, INSERT, UPDATE, DELETE ON Orders TO app_user; -- not GRANT ALL
```

## Alternatives & Trade-offs

Granting broad permissions ("just give the app `db_owner`, it's simpler") avoids friction when requirements change, but means a single compromised credential — a leaked connection string, a hijacked service account — grants an attacker everything that credential could ever do, not just what the application actually uses. Least privilege requires more upfront thought about exactly what each identity needs, and occasional friction when a legitimate new need arises, in exchange for dramatically reducing the blast radius of any single compromise.

## How It Works

### Database permissions — the application doesn't need what it doesn't use

```sql
-- The application's own database login should not have DDL permissions (CREATE/DROP/ALTER)
-- at all, since the application itself never issues those statements — only migrations do,
-- typically run under a separate, more privileged, and more carefully controlled account
GRANT SELECT, INSERT, UPDATE, DELETE ON SCHEMA::dbo TO app_login;
```

If the application's own credentials are ever leaked (a misconfigured secret, a logged connection string), an attacker with only `SELECT`/`INSERT`/`UPDATE`/`DELETE` can still do real damage, but categorically less than one with `DROP TABLE` or `db_owner` rights.

### Service-to-service permissions in a multi-service system

```
Order Service's identity: can read/write Orders, can call Inventory Service's "reserve stock" endpoint —
                            cannot directly access the Inventory Service's own database, cannot call
                            unrelated services' administrative endpoints
```

Each service's credentials should be scoped to exactly the cross-service calls it legitimately makes, not a blanket "trusted internal service" pass that grants access to everything.

### Rate limiting as blast-radius reduction, not just fairness

```csharp
// Module 8 covered this mechanically; the SECURITY angle is specifically:
// a compromised API key, even with correctly-scoped least-privilege permissions,
// can still be used to exfiltrate data or cause damage at whatever rate the API allows —
// a rate limit caps how much damage is possible per unit time, regardless of what the credential is authorized to do
options.AddFixedWindowLimiter("perApiKey", limiterOptions => { limiterOptions.PermitLimit = 100; });
```

Rate limiting doesn't replace least-privilege permission scoping — it's a complementary control specifically bounding the *rate* of damage, on top of already bounding the *scope* of what's possible.

### Least privilege applies to people too, not just service accounts

```
A developer with production database read access for debugging doesn't need write access.
An on-call engineer with incident-response permissions doesn't need permanent standing access
to every production system — time-boxed, audited elevation ("just-in-time access") is a common
pattern for reducing standing privilege even for legitimate administrative needs.
```

## Application

Scope every credential — application database logins, service-to-service identities, developer/operator access — to exactly what's needed, and no more, reviewing and tightening over time rather than defaulting to broad access for convenience. Layer rate limiting on top as a bound on how much damage is possible per unit time, regardless of what a given credential is scoped to do.

## Common Mistakes

- Granting broad database or service permissions "to avoid friction later," rather than scoping tightly and expanding deliberately when a genuine need arises.
- Treating rate limiting as purely an availability/fairness concern, missing its role as a security control limiting the blast radius of a compromised credential.
- Giving developers or operators permanent standing access to production systems for occasional needs, instead of time-boxed, audited elevation.
- Not revisiting and tightening permissions over time as a system evolves, letting scope creep accumulate unnoticed.

## Common Interview Questions

### Basic
- What is the principle of least privilege?
- How does rate limiting function as a security control, beyond just fairness?

### Intermediate
- Why should an application's own database credentials typically lack DDL permissions?
- What's the risk of granting broad "trusted internal service" access between microservices instead of scoping each service-to-service call?

### Advanced
- How would you design a least-privilege permission model for a multi-service system with several service-to-service dependencies?
- What is "just-in-time access," and what problem does it solve for human operators specifically?

### Follow-up Questions
- Does least privilege apply only to application/service credentials, or also to human operators?
- Can rate limiting alone substitute for properly scoped permissions?

### Code Prediction
An application's database credentials are leaked via a misconfigured logging statement. If those credentials have `db_owner` rights, what's the worst-case impact compared to credentials scoped only to `SELECT`/`INSERT`/`UPDATE`/`DELETE` on specific tables?

## Practical Tasks

- Design database permissions for an application, scoping its runtime credentials away from DDL operations entirely.
- Design service-to-service permission scoping for a small multi-service system, identifying the minimum access each service actually needs.
- Propose a rate-limiting policy specifically justified as a blast-radius control for a compromised API key scenario.

## Readiness Criteria

Apply least privilege to database, service, and human access scoping, and explain rate limiting's role as a security control complementary to, not a substitute for, properly scoped permissions.

## References

### Other

- [OWASP: Authorization Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Authorization_Cheat_Sheet.html)
