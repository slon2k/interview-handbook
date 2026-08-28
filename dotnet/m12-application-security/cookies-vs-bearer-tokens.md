# Cookies vs. Bearer Tokens

## Definition

Two common ways to carry authentication proof across requests: a **cookie** is set by the server and sent back automatically by the browser on every subsequent request to the same origin; a **bearer token** (typically a JWT) is included explicitly by the client in an `Authorization: Bearer <token>` header, with no automatic browser involvement.

```
Cookie: .AspNetCore.Cookies=CfDJ8...        — sent automatically by the browser
Authorization: Bearer eyJhbGciOiJIUzI1Ni...  — attached explicitly by client code
```

## Alternatives & Trade-offs

Cookies are automatically attached by the browser, which is convenient but is exactly why they're vulnerable to CSRF (a malicious site can trigger a request that the browser automatically attaches the victim's cookie to — see `cors-vs-csrf.md`) and require explicit CSRF defenses. Bearer tokens require the client to manage and attach them explicitly (no automatic browser behavior), which avoids CSRF by design, but shifts the responsibility for secure storage to the client — and a token stored insecurely (e.g., in `localStorage`, readable by any script) is vulnerable to theft via XSS instead.

## How It Works

### Cookie-based authentication — automatic attachment, CSRF risk

```csharp
builder.Services.AddAuthentication(CookieAuthenticationDefaults.AuthenticationScheme)
    .AddCookie(options =>
    {
        options.Cookie.HttpOnly = true;   // not readable by JavaScript — mitigates XSS-based theft
        options.Cookie.SecurePolicy = CookieSecurePolicy.Always; // HTTPS only
        options.Cookie.SameSite = SameSiteMode.Strict; // mitigates CSRF by refusing to send the cookie cross-site
    });
```

`HttpOnly` and `SameSite` are the two cookie attributes doing the most defensive work here — `HttpOnly` blocks script-based theft (XSS), `SameSite` blocks the browser from automatically attaching the cookie to cross-site requests (CSRF).

### Bearer token authentication — explicit attachment, no CSRF exposure by default

```csharp
builder.Services.AddAuthentication(JwtBearerDefaults.AuthenticationScheme)
    .AddJwtBearer(options => { /* token validation configuration — see jwt-structure-and-validation.md */ });
```

```javascript
// Client explicitly attaches the token — nothing happens automatically the way a cookie would
fetch('/orders', { headers: { 'Authorization': `Bearer ${token}` } });
```

Because the browser never attaches an `Authorization` header automatically to a request it didn't construct, a malicious cross-site page can't trigger an authenticated request the way it can exploit an automatically-attached cookie — this is why bearer tokens are inherently CSRF-resistant, though not risk-free overall.

### Where each token type is typically stored, and the corresponding risk

```
Cookie (HttpOnly)         — not readable by JavaScript at all; safest against XSS token theft
localStorage/sessionStorage — readable by any script running on the page; vulnerable if XSS exists anywhere
In-memory (JS variable)  — safest against persistent theft, but lost on page refresh, requiring re-authentication
```

## Application

Use cookies (with `HttpOnly`, `Secure`, and `SameSite` configured correctly) for traditional server-rendered or same-origin web applications where CSRF defenses are already part of the framework's tooling. Use bearer tokens for APIs consumed by SPAs, mobile apps, or third-party clients where CSRF isn't a natural concern but token storage on the client needs careful handling to avoid XSS-based theft.

## Common Mistakes

- Storing a bearer token in `localStorage` without considering that any XSS vulnerability anywhere on the page can read and exfiltrate it.
- Using cookie-based authentication without setting `SameSite` and `HttpOnly`, leaving the application exposed to CSRF and script-based token theft respectively.
- Assuming bearer tokens are immune to all the risks cookies have — they avoid CSRF by design, but introduce their own storage-security responsibility on the client.
- Mixing both mechanisms inconsistently across an application without a clear rule for which is used where, complicating the security review of the whole system.

## Common Interview Questions

### Basic
- What's the fundamental difference in how cookies and bearer tokens are attached to a request?
- Why are cookies vulnerable to CSRF in a way bearer tokens generally aren't?

### Intermediate
- What do the `HttpOnly`, `Secure`, and `SameSite` cookie attributes each protect against?
- Where should a bearer token be stored on the client, and what's the risk of each option?

### Advanced
- How would you design authentication for an application with both a same-origin web frontend and a separate mobile app consuming the same API?
- Why does storing a bearer token in an HttpOnly cookie combine properties of both mechanisms, and what trade-off does that specific combination make?

### Follow-up Questions
- Does `SameSite=Strict` on a cookie eliminate CSRF risk entirely?
- Can a bearer token be stolen via a mechanism other than XSS?

### Code Prediction
A cookie is set without `HttpOnly`. An XSS vulnerability exists elsewhere on the same page. What can an attacker's injected script do with this cookie that it couldn't do if `HttpOnly` were set?

## Practical Tasks

- Configure cookie authentication with `HttpOnly`, `Secure`, and `SameSite` set correctly, and explain what each setting defends against.
- Implement bearer token authentication for an API and discuss where the client should store the token.
- Design the authentication mechanism for a system with both a same-origin web app and a public API consumed by third parties.

## Readiness Criteria

Explain the CSRF/XSS trade-off between cookies and bearer tokens precisely, configure cookie security attributes correctly, and choose the appropriate mechanism for a given client architecture.

## References

### Microsoft Learn

- [Cookie authentication](https://learn.microsoft.com/aspnet/core/security/authentication/cookie)
- [JWT bearer authentication](https://learn.microsoft.com/aspnet/core/security/authentication/configure-jwt-bearer-authentication)
