# CORS vs. CSRF

## Definition

**CORS** (covered mechanically in Module 8) is a browser-enforced *relaxation* of the same-origin policy — it controls whether cross-origin JavaScript can *read* a response. **CSRF** (Cross-Site Request Forgery) is an *attack*: a malicious site tricks a victim's browser into *sending* a request to another site the victim is authenticated to, exploiting the fact that the browser automatically attaches cookies regardless of which site triggered the request. These are easy to conflate, but they protect against — and depend on — fundamentally different things.

```
CORS:  controls whether malicious-site.com's JavaScript can READ the response from api.example.com
CSRF:  malicious-site.com tricks the browser into SENDING a request to api.example.com in the first place,
       and the browser attaches api.example.com's cookies automatically, regardless of CORS policy
```

## Alternatives & Trade-offs

CORS and CSRF defenses solve different problems and neither substitutes for the other. CORS being strict doesn't prevent CSRF — the malicious cross-site *request* still gets sent and the cookie still gets attached; CORS only stops the attacker's page from reading the *response*, which classic CSRF attacks (submitting a form, triggering a state change) don't even need to do. Defending against CSRF specifically requires its own mechanism — anti-forgery tokens, or `SameSite` cookies.

## How It Works

### Why CORS doesn't stop CSRF

```html
<!-- On malicious-site.com -->
<form action="https://api.example.com/transfer-funds" method="POST">
  <input type="hidden" name="amount" value="10000" />
  <input type="hidden" name="to" value="attacker-account" />
</form>
<script>document.forms[0].submit();</script>
```

If the victim is logged into `api.example.com` via a cookie, their browser automatically attaches that cookie to this form submission — the request succeeds server-side regardless of any CORS policy, because the attacker's page never needed to *read* the response to cause damage; the state-changing side effect already happened.

### Anti-forgery tokens — the classic CSRF defense

```csharp
[HttpPost]
[ValidateAntiForgeryToken]
public IActionResult TransferFunds(TransferRequest request) { }
```

```html
<form asp-action="TransferFunds" method="post">
    @Html.AntiForgeryToken() <!-- embeds a token the server can verify came from a page it actually served -->
</form>
```

The anti-forgery token is a value the server embeds in a legitimate form and checks on submission — a malicious site has no way to obtain or forge this token, since it never rendered the real page, so its forged form submission fails validation even though the victim's cookie was attached correctly.

### `SameSite` cookies — a second, complementary defense

```csharp
options.Cookie.SameSite = SameSiteMode.Strict; // browser refuses to attach this cookie to a cross-site request at all
```

With `SameSite=Strict` (or `Lax` for many practical cases), the browser itself declines to attach the cookie to a request originating from another site — stopping the attack before the cookie ever reaches the server, rather than relying on the server to detect a missing/invalid anti-forgery token.

### Why bearer-token APIs are naturally more CSRF-resistant

As covered in `cookies-vs-bearer-tokens.md`, a malicious site can trigger a cookie-carrying request automatically, but it can't make the victim's browser attach an `Authorization: Bearer` header the way it can a cookie — bearer-token APIs are inherently less exposed to classic CSRF, though not immune to every related risk.

## Application

Never treat CORS configuration as a CSRF defense — they address different threats. For any cookie-authenticated, state-changing endpoint, implement anti-forgery tokens and/or `SameSite` cookie attributes specifically. Understand that bearer-token APIs reduce (but don't eliminate) CSRF exposure by design.

## Common Mistakes

- Believing a strict CORS policy protects against CSRF, when CSRF doesn't require reading the response at all — only triggering the side effect.
- Implementing CSRF defenses only for browser form submissions and forgetting that any cookie-authenticated state-changing endpoint (including one called via `fetch`/XHR) needs the same protection.
- Relying solely on `SameSite` cookies without anti-forgery tokens, or vice versa, when defense-in-depth (both mechanisms) is more robust against edge cases and older browser behavior.
- Confusing the two concepts in an interview answer — a very common and telling mix-up given how similar the acronyms look.

## Common Interview Questions

### Basic
- What's the fundamental difference between what CORS and CSRF each concern?
- Why doesn't a strict CORS policy prevent CSRF attacks?

### Intermediate
- How does an anti-forgery token defend against CSRF specifically?
- How does `SameSite=Strict` defend against CSRF at the browser level, before a request even reaches the server?

### Advanced
- Why are bearer-token-authenticated APIs naturally more resistant to classic CSRF than cookie-authenticated ones?
- How would you design CSRF protection for an API that must support both a cookie-authenticated web frontend and a bearer-token-authenticated mobile app?

### Follow-up Questions
- Does a GET-only, side-effect-free endpoint need CSRF protection?
- Is CSRF still a relevant concern for a purely JSON-based API with no HTML forms?

### Code Prediction
An API endpoint accepting `POST` requests is cookie-authenticated and has no anti-forgery token check and no `SameSite` restriction on its auth cookie, but does have a strict CORS policy allowing only `https://app.example.com`. Can a malicious site at `https://evil.com` still successfully trigger a state change on this endpoint via a form submission?

## Practical Tasks

- Implement anti-forgery token validation for a cookie-authenticated, state-changing endpoint.
- Configure `SameSite` on an authentication cookie and verify a simulated cross-site request no longer carries it.
- Explain, for a specific endpoint's CORS configuration, why it does or doesn't provide any CSRF protection.

## Readiness Criteria

Distinguish CORS and CSRF precisely as protecting against different things, implement anti-forgery tokens and `SameSite` cookies correctly, and explain why bearer tokens reduce (but don't eliminate) CSRF exposure.

## References

### Microsoft Learn

- [Prevent Cross-Site Request Forgery (CSRF/XSRF) attacks](https://learn.microsoft.com/aspnet/core/security/anti-request-forgery)

### Other

- [OWASP: Cross-Site Request Forgery Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
