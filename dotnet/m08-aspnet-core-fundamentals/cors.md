# CORS

## Definition

Cross-Origin Resource Sharing (CORS) is a browser-enforced security mechanism that blocks JavaScript running on one origin (scheme + host + port) from reading responses from a different origin, unless the server explicitly opts in via CORS response headers. It's purely a browser-side protection — CORS does not protect a server from non-browser clients (a mobile app, `curl`, or a server-to-server call) at all.

```csharp
builder.Services.AddCors(options =>
{
    options.AddPolicy("AllowFrontend", policy =>
        policy.WithOrigins("https://app.example.com")
              .AllowAnyMethod()
              .AllowAnyHeader());
});

app.UseCors("AllowFrontend");
```

## Alternatives & Trade-offs

Allowing all origins (`AllowAnyOrigin()`) is simplest to set up and fine for a fully public, unauthenticated API, but combined with credentials (cookies, `Authorization` headers) it becomes a real security risk — the browser will refuse to combine `AllowAnyOrigin()` with credentialed requests specifically for this reason. Explicitly listing allowed origins is more setup but is the only safe option once the API serves authenticated, credentialed requests from a browser-based frontend.

## How It Works

### Why CORS exists — same-origin policy

```
https://app.example.com  calling  https://api.example.com
```
Different origins (different hosts). Without CORS headers from `api.example.com`, the browser blocks the frontend JavaScript from reading the response — even though the request itself may have already reached the server and executed.

### Preflight requests

```
OPTIONS /orders HTTP/1.1
Origin: https://app.example.com
Access-Control-Request-Method: POST
Access-Control-Request-Headers: Content-Type, Authorization

HTTP/1.1 204 No Content
Access-Control-Allow-Origin: https://app.example.com
Access-Control-Allow-Methods: POST
Access-Control-Allow-Headers: Content-Type, Authorization
```

For "non-simple" requests (custom headers, methods other than `GET`/`POST` with simple content types), the browser automatically sends a preflight `OPTIONS` request first, and only proceeds with the actual request if the server's preflight response allows it.

### CORS with credentials (cookies/auth headers)

```csharp
options.AddPolicy("AllowFrontendWithCredentials", policy =>
    policy.WithOrigins("https://app.example.com") // must be a specific origin, not "*"
          .AllowCredentials()
          .AllowAnyMethod()
          .AllowAnyHeader());
```

`AllowCredentials()` cannot be combined with `AllowAnyOrigin()` — the browser spec forbids it, since it would let any site read credentialed responses on the user's behalf.

### CORS does not protect the server itself

```
curl -X POST https://api.example.com/orders   # CORS headers are irrelevant here — curl isn't a browser
```

A request from a non-browser client, or a server-to-server call, is never subject to CORS at all — the restriction is enforced entirely by the browser reading the response, not by the server refusing to process the request. Real server-side protection against unauthorized access requires authentication/authorization (Module 12), not CORS.

## Application

Configure CORS with an explicit allow-list of trusted origins for any browser-based frontend calling the API from a different origin, especially once authentication/cookies are involved. Never use `AllowAnyOrigin()` combined with credentials. Understand that CORS is not a security boundary against non-browser clients — it exists solely to protect users' browsers from malicious cross-origin JavaScript.

## Common Mistakes

- Treating CORS as a general access-control/security mechanism, when it only affects browser-based JavaScript and does nothing to stop direct API calls from any other kind of client.
- Using `AllowAnyOrigin()` on an API that also uses cookies or `Authorization` headers, which browsers will reject when combined with `AllowCredentials()` — and which would be a real security hole if it were somehow allowed.
- Forgetting to allow the specific headers or methods the frontend actually sends, causing preflight failures that manifest as a confusing browser-console CORS error even though the actual endpoint works fine when called directly.
- Debugging a CORS error by trying to fix it server-side when the real symptom is a completely unrelated server error — some browsers report a masked CORS error when the actual response was a `500` with missing CORS headers, since the browser can't distinguish "server error" from "server didn't send CORS headers" in some failure modes.

## Common Interview Questions

### Basic
- What is CORS, and what does it protect against?
- Does CORS provide any security against non-browser clients like `curl` or a mobile app?

### Intermediate
- What is a CORS preflight request, and when does the browser send one?
- Why can't `AllowAnyOrigin()` be combined with `AllowCredentials()`?

### Advanced
- Explain, precisely, what actually happens at the network level when a CORS-blocked request is made — does the server ever receive/process it?
- How would you diagnose a CORS error that's actually masking an unrelated server-side `500` error?

### Follow-up Questions
- If a request is blocked by CORS, has the server already processed it?
- Does CORS protect a REST API from unauthorized access by non-browser clients?

### Code Prediction
A frontend at `https://app.example.com` calls a `POST /orders` endpoint at `https://api.example.com` that has no CORS policy configured at all. Does the server receive and process the request? Does the frontend JavaScript receive the response body?

## Practical Tasks

- Configure a CORS policy allowing a specific frontend origin, including credentials support, and verify a request from an unlisted origin is blocked.
- Reproduce a preflight failure by sending a custom header the CORS policy doesn't allow, and fix the policy.
- Explain, for a teammate confused by a browser CORS error, why `curl` to the same endpoint works fine.

## Readiness Criteria

Explain what CORS actually protects and what it doesn't, correctly configure origin/credential policies, and diagnose CORS versus unrelated server-error symptoms.

## References

### Microsoft Learn

- [Enable Cross-Origin Requests (CORS) in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/cors)

### Other

- [MDN: Cross-Origin Resource Sharing (CORS)](https://developer.mozilla.org/en-US/docs/Web/HTTP/CORS)
