# JWT Structure and Validation

## Definition

A JWT (JSON Web Token) is a compact, self-contained token format: three base64url-encoded parts separated by dots — a **header** (algorithm/type), a **payload** (claims), and a **signature** (proving the token wasn't tampered with, given the issuer's key). Crucially, a JWT's payload is only base64-*encoded*, not encrypted — anyone can decode and read it; the signature only proves it wasn't altered.

```
eyJhbGciOiJIUzI1NiJ9.eyJzdWIiOiJhbGljZSJ9.SflKxwRJSMeKKF2QT4fwpMeJf36POk6yJV_adQssw5c
└──────header────────┘└──────payload──────┘└─────────────signature─────────────────┘
```

## Alternatives & Trade-offs

A JWT is self-contained and stateless — a server can validate it (checking the signature and claims) without a database lookup or a call to the issuing service, which is what makes JWTs attractive for distributed systems and microservices. The trade-off is that a JWT can't be easily revoked before its natural expiration — once issued and signed, it remains valid to anyone who checks only the signature, unless additional infrastructure (a revocation list, short expirations plus refresh tokens) is layered on top.

## How It Works

### Decoding vs. validating — a critical distinction

```csharp
// Decoding just reads the payload — proves NOTHING about authenticity
var handler = new JwtSecurityTokenHandler();
var token = handler.ReadJwtToken(rawToken); // anyone can do this; it's just base64 decoding

// Validating checks the signature against a trusted key, expiration, issuer, and audience
var principal = handler.ValidateToken(rawToken, new TokenValidationParameters
{
    ValidateIssuerSigningKey = true,
    IssuerSigningKey = signingKey,
    ValidateIssuer = true,
    ValidIssuer = "https://auth.example.com",
    ValidateAudience = true,
    ValidAudience = "my-api",
    ValidateLifetime = true
}, out _);
```

Reading a JWT's claims without validating the signature tells you what the token *claims* to say, not whether it's genuinely from a trusted issuer and hasn't been tampered with — a common and dangerous mistake is trusting decoded claims without validation ever having occurred.

### The payload is visible to anyone — never put secrets in it

```json
{ "sub": "alice@example.com", "role": "Admin", "creditCardNumber": "4111111111111111" }
```

Because the payload is only encoded, not encrypted, anything in it is readable by intercepting or even just copying the token — sensitive data has no business being in a JWT payload at all.

### `alg: none` and algorithm-confusion attacks

```csharp
// A historically real vulnerability class: some libraries, if misconfigured, would accept
// a token whose header claims alg: "none" or a mismatched algorithm, skipping signature
// verification entirely
```

Modern, correctly-configured JWT libraries reject these by default, but it's a well-known attack class worth knowing: never allow the token itself to dictate which algorithm is used to verify it — the verifying side should enforce an expected algorithm independently.

### Expiration and clock skew

```csharp
options.TokenValidationParameters.ClockSkew = TimeSpan.FromMinutes(2); // allows small time differences between servers
```

## Application

Always validate a JWT's signature, issuer, audience, and expiration before trusting any of its claims — never just decode and read it. Never place secrets or sensitive data in a JWT payload. Keep access token lifetimes short, pairing them with refresh tokens (next topic) for renewed access without re-authenticating from scratch, since a compromised long-lived JWT can't be easily revoked.

## Common Mistakes

- Decoding a JWT's payload and trusting its claims without ever validating the signature, issuer, or expiration.
- Storing sensitive data (passwords, full card numbers, secrets) in a JWT payload, forgetting that it's only encoded, not encrypted.
- Trusting the token's own declared algorithm rather than enforcing an expected algorithm on the verifying side.
- Issuing JWTs with very long expiration times with no revocation mechanism, making a leaked token dangerous for far longer than necessary.

## Common Interview Questions

### Basic
- What are the three parts of a JWT?
- Is a JWT's payload encrypted?

### Intermediate
- What's the difference between decoding a JWT and validating it?
- Why is it risky to put sensitive data in a JWT payload?

### Advanced
- What is an algorithm-confusion attack against JWT validation, and how do correctly-configured libraries prevent it?
- How would you design a token expiration and revocation strategy given that a JWT can't be easily invalidated before it naturally expires?

### Follow-up Questions
- Does a longer JWT expiration time make the system more or less secure, all else equal?
- Can a JWT's claims be trusted if the signature validation step is skipped entirely?

### Code Prediction
An application reads a JWT's claims using `ReadJwtToken` (decode only) and grants access based on a `role: "Admin"` claim found in the payload, without ever calling `ValidateToken`. What could an attacker do with a token they constructed themselves, containing whatever claims they want?

## Practical Tasks

- Decode a JWT manually (base64) to inspect its header and payload, then validate the same token programmatically and compare what each step actually verifies.
- Configure `TokenValidationParameters` correctly for issuer, audience, signing key, and expiration.
- Design an access-token expiration and refresh strategy for an API that can't easily support token revocation.

## Readiness Criteria

Explain JWT structure and the decode-vs-validate distinction precisely, correctly configure validation parameters, and design an expiration/revocation strategy appropriate for a token format that can't be easily invalidated.

## References

### Microsoft Learn

- [JWT bearer authentication](https://learn.microsoft.com/aspnet/core/security/authentication/configure-jwt-bearer-authentication)

### Other

- [RFC 7519: JSON Web Token (JWT)](https://www.rfc-editor.org/rfc/rfc7519)
- [jwt.io (token decoder/reference)](https://jwt.io/)
