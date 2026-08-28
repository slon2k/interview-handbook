# HTTPS

## Definition

HTTPS is HTTP layered over TLS, encrypting traffic between client and server so it can't be read or tampered with in transit by anyone observing the network path — a coffee-shop Wi-Fi eavesdropper, a compromised router, an ISP. Without it, everything sent over plain HTTP — passwords, session cookies, tokens, response bodies — is visible to anyone positioned to intercept the connection.

```csharp
app.UseHttpsRedirection(); // redirects any plain HTTP request to HTTPS
app.UseHsts();              // tells browsers to remember and enforce HTTPS for this site going forward
```

## Alternatives & Trade-offs

There's essentially no legitimate alternative to HTTPS for any traffic carrying credentials, tokens, or sensitive data — plain HTTP has no confidentiality or integrity guarantee at all. The only real "trade-off" is the small computational and latency cost of the TLS handshake, which modern hardware and protocol optimizations (TLS 1.3, session resumption) have made negligible for essentially all practical purposes.

## How It Works

### What HTTPS actually protects against

```
Without HTTPS: an attacker on the network path can read the entire request/response,
               including Authorization headers, cookies, and response bodies, in plain text.
With HTTPS:    the same attacker sees only encrypted bytes — the content, headers, and even
               the specific URL path (though not the destination host, visible via SNI) are protected.
```

### Redirecting HTTP to HTTPS

```csharp
app.UseHttpsRedirection(); // a plain HTTP request to /orders is redirected (307/308) to https://.../orders
```

This alone still means the *first* request happens over plain HTTP before the redirect — which is exactly what HSTS exists to close.

### HSTS — telling the browser to never even try plain HTTP again

```csharp
builder.Services.AddHsts(options =>
{
    options.MaxAge = TimeSpan.FromDays(365);
    options.IncludeSubDomains = true;
});
app.UseHsts();
```

```
Strict-Transport-Security: max-age=31536000; includeSubDomains
```

Once a browser has seen this header from a site, it remembers to only ever request that site over HTTPS directly — for the specified duration — without even attempting a plain HTTP request first, closing the small window where the very first request in `UseHttpsRedirection()`'s flow was still unencrypted.

### Certificate validation — don't disable it

```csharp
// NEVER do this, even "temporarily" for local testing against a self-signed cert in a shared codebase:
handler.ServerCertificateCustomValidationCallback = (msg, cert, chain, errors) => true; // accepts ANY certificate
```

Disabling certificate validation defeats HTTPS's entire integrity guarantee — it would accept a connection to an attacker impersonating the real server just as readily as the genuine one.

## Application

Enforce HTTPS everywhere in production — redirect HTTP to HTTPS and enable HSTS so browsers stop attempting plain HTTP entirely after the first visit. Never disable certificate validation outside of tightly-scoped, clearly-commented local development configuration that can't accidentally ship to production.

## Common Mistakes

- Relying only on `UseHttpsRedirection()` without HSTS, leaving a small window where the very first request to a site happens over plain HTTP before the redirect occurs.
- Disabling certificate validation to work around a local development certificate issue, and having that configuration accidentally reach a production build.
- Assuming HTTPS protects the destination host/domain being visited — SNI (Server Name Indication) during the TLS handshake reveals which hostname is being connected to, even though the request/response content itself is encrypted.
- Treating HTTPS as optional for "internal" or "non-sensitive" traffic, when internal network segments are far from guaranteed to be free of eavesdropping risk.

## Common Interview Questions

### Basic
- What does HTTPS protect against that plain HTTP doesn't?
- What's the difference between `UseHttpsRedirection()` and `UseHsts()`?

### Intermediate
- What's the remaining window of vulnerability that `UseHttpsRedirection()` alone doesn't close, and how does HSTS close it?
- Why should certificate validation never be disabled, even temporarily?

### Advanced
- What information does HTTPS NOT protect, given that SNI reveals the destination hostname during the TLS handshake?
- How would you roll out HSTS safely for a site that historically hasn't enforced HTTPS, given how long browsers remember the `max-age` setting?

### Follow-up Questions
- Does HTTPS protect against every kind of network-based attack?
- Is HTTPS necessary for internal, server-to-server traffic within a private network?

### Code Prediction
A site enables `UseHttpsRedirection()` but not `UseHsts()`. A user manually types `http://example.com` into their browser for the very first visit ever. What happens to that very first request, before the redirect takes effect?

## Practical Tasks

- Configure HTTPS redirection and HSTS for an ASP.NET Core application, and verify the `Strict-Transport-Security` response header.
- Explain, for a code review, why a proposed `ServerCertificateCustomValidationCallback` bypassing certificate validation is dangerous, even if scoped to "local development only."
- Design a safe rollout plan for enabling HSTS on a previously HTTP-only production site, considering the `max-age` commitment involved.

## Readiness Criteria

Explain what HTTPS protects against and its remaining gaps (SNI visibility), configure HTTPS redirection and HSTS correctly, and never compromise certificate validation.

## References

### Microsoft Learn

- [Enforce HTTPS in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/enforcing-ssl)
- [HTTP Strict Transport Security Protocol (HSTS)](https://learn.microsoft.com/aspnet/core/security/enforcing-ssl#hsts)
