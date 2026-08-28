# Module 12 - Application Security

**Status:** Complete  
**Priority:** High  
**Prerequisites:** [ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md), [Relational Databases and SQL](../m09-relational-databases-and-sql/README.md)

## Scope

This module focuses on secure application-development fundamentals for a mid-level backend role — authentication and authorization mechanics, the OWASP-catalogued vulnerability classes and their fixes, and secure operational practices (secrets, logging, least privilege). It deliberately stays away from cryptographic implementation details (designing your own crypto, TLS internals) in favor of what a working backend developer actually needs to get right day to day.

Several topics here are explicitly the "security lens" on mechanics already covered elsewhere — CORS (Module 8), rate limiting (Module 8), file uploads (Module 8), SQL injection (Module 9), health checks and caching operational concerns (Module 13). This module cross-references rather than repeats that mechanical content.

## Learning Outcomes

By the end of this module, you should be able to:

- Distinguish authentication from authorization precisely, and design claims/roles/policies for realistic authorization requirements.
- Choose correctly between cookies and bearer tokens, and implement JWT validation and access/refresh token rotation correctly.
- Store passwords and manage secrets using established, purpose-built mechanisms rather than custom solutions.
- Explain and defend against CSRF, XSS, SQL/command injection, mass assignment, and insecure file uploads.
- Apply least privilege to database, service, and human access, and use rate limiting as a security control.
- Use the OWASP Top 10 and API Security Top 10 as a design-review lens, and recognize Broken Object-Level Authorization specifically.
- Explain GDPR's practical, system-design implications at an awareness level.

## Topics

### 1. Authentication and Authorization

- [Authentication vs. authorization](authentication-vs-authorization.md)
- [Claims, roles, and policies](claims-roles-and-policies.md)
- [Cookies vs. bearer tokens](cookies-vs-bearer-tokens.md)
- [JWT structure and validation](jwt-structure-and-validation.md)
- [Access and refresh tokens](access-and-refresh-tokens.md)

### 2. Secrets and Transport

- [Password storage](password-storage.md)
- [Secret management](secret-management.md)
- [HTTPS](https.md)

### 3. Common Vulnerability Classes

- [CORS vs. CSRF](cors-vs-csrf.md)
- [XSS and output encoding](xss-and-output-encoding.md)
- [Input validation and injection prevention](input-validation-and-injection-prevention.md)
- [Mass assignment / over-posting](mass-assignment-and-over-posting.md)
- [File-upload security](file-upload-security.md)

### 4. Defensive Principles and Practice

- [Principle of least privilege, and rate limiting as a security control](least-privilege-and-rate-limiting.md)
- [Secure logging](secure-logging.md)

### 5. Frameworks and Compliance

- [OWASP Top 10 and OWASP API Security Top 10](owasp-top-10-and-api-security-top-10.md)
- [GDPR awareness](gdpr-awareness.md)

## Scope Boundaries

- CORS and rate-limiting mechanics (how to configure them in ASP.NET Core) belong in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md); this module covers the security *rationale* and the specific gaps each does and doesn't cover.
- SQL injection's full treatment (the mechanics of parameterized queries) belongs in [Module 9 - Relational Databases and SQL](../m09-relational-databases-and-sql/README.md); this module recaps it as one instance of the broader injection pattern.
- File-upload mechanics (streaming, `IFormFile`, size limits) belong in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md); this module covers the security-specific verification and storage concerns.
- Cryptographic implementation details, TLS internals, and designing custom cryptographic schemes are out of scope entirely — use established libraries and, for real compliance or cryptographic design decisions, appropriate specialist expertise.
- Broader observability and health-check mechanics belong in [Module 13 - Performance, Diagnostics, and Observability](../m13-performance-diagnostics-observability/README.md).

## Suggested Learning Sequence

1. Authentication vs. authorization, claims/roles/policies.
2. Cookies vs. bearer tokens, JWT structure and validation, access/refresh tokens.
3. Password storage, secret management, HTTPS.
4. CORS vs. CSRF, XSS, input validation/injection, mass assignment, file-upload security.
5. Least privilege, rate limiting as a security control, secure logging.
6. OWASP Top 10 / API Security Top 10, GDPR awareness.

## Practical Deliverables

- Implement role-based and policy-based authorization for a small API with mixed access requirements.
- Implement JWT validation with correct issuer/audience/expiration checks, and an access/refresh token flow with rotation.
- Implement password hashing using a purpose-built library, and load a secret from a dedicated secret store rather than a config file.
- Fix a deliberately introduced CSRF, XSS, mass-assignment, and Broken Object-Level Authorization vulnerability, one at a time.
- Design least-privilege database permissions for an application's runtime credentials.
- Audit a set of logging statements for sensitive-data exposure.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and mechanism familiarity.
- Intermediate questions involving common usage and configuration trade-offs.
- Advanced questions involving attack scenarios and defense-in-depth design.
- Follow-up questions that test precise understanding rather than memorized definitions.
- Code-prediction questions grounded in concrete vulnerable/fixed code pairs, since security topics are frequently tested by asking "what could go wrong here" against a specific snippet.

## References

### Microsoft Learn

- [Overview of ASP.NET Core security](https://learn.microsoft.com/aspnet/core/security/)

### Other

- [OWASP Top 10](https://owasp.org/www-project-top-ten/)
- [OWASP API Security Top 10](https://owasp.org/API-Security/editions/2023/en/0x00-header/)
- [OWASP Cheat Sheet Series](https://cheatsheetseries.owasp.org/)
