# XSS and Output Encoding

## Definition

Cross-Site Scripting (XSS) occurs when untrusted data is rendered into a page in a way that lets it execute as script in another user's browser — stealing cookies/tokens, performing actions as the victim, or defacing content. **Output encoding** (escaping data for the specific context it's rendered into — HTML, HTML attribute, JavaScript, URL) is the primary defense: it ensures untrusted data is displayed as literal text, never interpreted as executable markup or code.

```html
<!-- Untrusted input: <script>stealCookies()</script> -->

<!-- Unencoded: the browser executes it as a real script tag -->
<div>Comment: <script>stealCookies()</script></div>

<!-- HTML-encoded: the browser displays it as literal text, harmlessly -->
<div>Comment: &lt;script&gt;stealCookies()&lt;/script&gt;</div>
```

## Alternatives & Trade-offs

Modern templating engines (Razor in ASP.NET Core) HTML-encode output by default, which closes the most common XSS vector automatically without any effort from the developer — the trade-off only appears when a developer deliberately opts out of that default (rendering raw HTML) or renders untrusted data into a *different* context (a JavaScript variable, a URL, an HTML attribute) that the default HTML-encoding doesn't protect, since each context needs its own specific encoding rule.

## How It Works

### Razor encodes by default — the safe path requires no extra effort

```razor
@* Automatically HTML-encoded — safe even if Model.Comment contains "<script>..." *@
<div>@Model.Comment</div>

@* Explicitly opts OUT of encoding — only ever use this for content you've deliberately sanitized and trust *@
<div>@Html.Raw(Model.Comment)</div>
```

`@Html.Raw()` is the specific, deliberate escape hatch — its presence in a code review should always prompt the question "is this data actually trusted, and why?"

### Different contexts need different encoding — HTML encoding alone isn't universal

```html
<!-- HTML context: HTML-encoding is correct and sufficient -->
<div>@Model.Comment</div>

<!-- HTML attribute context: needs attribute encoding, which handles quote characters correctly -->
<div title="@Model.Comment"></div>

<!-- JavaScript context: HTML encoding does NOT protect against this -->
<script>var comment = "@Model.Comment";</script> <!-- a value containing a quote can break out of the string literal -->
```

Injecting untrusted data directly into a `<script>` block needs JavaScript-specific encoding (or, better, avoiding this pattern entirely by passing data via a data attribute the script reads, rather than interpolating it directly into script source).

### Content Security Policy (CSP) — a defense-in-depth layer, not a replacement

```
Content-Security-Policy: script-src 'self'; object-src 'none'
```

A well-configured CSP restricts which script sources the browser will execute at all, meaning even if an XSS payload somehow gets injected, the browser may refuse to run it — a valuable additional layer, but not a substitute for correct output encoding in the first place.

### Sanitization — for the rare case where some HTML must actually be allowed

```csharp
// When users genuinely need to submit some HTML (a rich text editor), encoding everything isn't an option —
// a dedicated sanitization library (e.g., HtmlSanitizer) strips dangerous tags/attributes while preserving safe ones
var sanitized = sanitizer.Sanitize(userSubmittedHtml);
```

## Application

Rely on Razor's default HTML encoding for ordinary output, and treat `@Html.Raw()` as a red flag requiring explicit justification. Use context-appropriate encoding (or better, avoid interpolating untrusted data directly into JavaScript/URLs at all) for non-HTML contexts. Add a Content Security Policy as defense-in-depth. Use a real sanitization library, never a hand-rolled one, for the specific case where some user-submitted HTML must be preserved.

## Common Mistakes

- Using `@Html.Raw()` on untrusted data without sanitizing it first, reopening exactly the vulnerability the default encoding was preventing.
- Assuming HTML encoding protects data interpolated directly into a `<script>` block or a URL, when each context needs its own specific encoding rule.
- Writing a custom, hand-rolled HTML sanitizer instead of using an established library, missing edge cases a well-reviewed library already handles.
- Treating a Content Security Policy as sufficient on its own, without also correctly encoding output — CSP is a valuable additional layer, not a substitute.

## Common Interview Questions

### Basic
- What is XSS, and what is output encoding's role in preventing it?
- Does Razor encode output by default?

### Intermediate
- Why is HTML encoding insufficient for data interpolated directly into a JavaScript context?
- What does `@Html.Raw()` do, and why should its use always be scrutinized?

### Advanced
- How would you safely allow a limited subset of user-submitted HTML (e.g., from a rich text editor) without reintroducing XSS risk?
- What does a Content Security Policy add on top of correct output encoding, and why isn't it a substitute?

### Follow-up Questions
- Is XSS only a risk for HTML rendered on a server, or can it also occur in a purely client-side rendered SPA?
- Does encoding output on the way out remove the need to validate input on the way in?

### Code Prediction
A comment field containing `"; alert('xss'); //` is rendered into a page using `<script>var comment = "@Model.Comment";</script>` with only Razor's default HTML encoding (no JavaScript-specific encoding) applied. Does the default HTML encoding prevent this from breaking out of the string literal and executing as script?

## Practical Tasks

- Reproduce an XSS vulnerability using `@Html.Raw()` on untrusted input, then fix it by removing the raw rendering or properly sanitizing the content.
- Identify a case where untrusted data is interpolated into a JavaScript or URL context and apply the correct context-specific encoding or redesign.
- Configure a basic Content Security Policy for an application and verify it blocks an inline script injection attempt.

## Readiness Criteria

Explain XSS and output encoding precisely, recognize when default HTML encoding is insufficient for a given context, and use sanitization libraries correctly for the cases where raw HTML genuinely must be preserved.

## References

### Microsoft Learn

- [Prevent Cross-Site Scripting (XSS) in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/cross-site-scripting)

### Other

- [OWASP: Cross Site Scripting Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross_Site_Scripting_Prevention_Cheat_Sheet.html)
