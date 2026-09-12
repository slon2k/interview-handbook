# Cookies, Storage, Auth Flows, and `401`/`403` Handling

## Definition

Browser storage and request credentials determine whether a user is recognized across requests and whether sensitive state remains available within the browser. Frontend code often interacts with auth flows through cookies, bearer tokens, local storage, and server response status codes such as `401 Unauthorized` and `403 Forbidden`.

## How It Works

- Cookies are sent automatically with requests to matching domains and can be used for session-based auth.
- `localStorage` and `sessionStorage` are browser persistence mechanisms, but they are not automatically secure or protected from script access.
- A `401` usually means the current request is unauthenticated or the session is invalid; a `403` usually means the client is authenticated but not allowed.
- Cookie-based and bearer-token flows have different consequences for CSRF, storage, and redirect behavior.
- Client logic must decide whether to redirect to login, refresh a token, or show a generic error message.

## Application

Treat browser storage as a UI or session convenience, not as a trusted security boundary. When a request fails with a status code, map that to a deliberate user action instead of a generic toast or silent no-op.

## Common Mistakes

- Storing JWTs in `localStorage` without considering XSS exposure.
- Treating a `401` as a generic failure and never redirecting to login or refreshing credentials.
- Using `localStorage` for sensitive data that should live in a server-side session or secure cookie.
- Ignoring CSRF when using cookie-based auth in a browser application.

## Common Interview Questions

### Foundation

- What is the difference between a cookie-based session and a bearer-token flow?
- Why are `401` and `403` handled differently?

### Intermediate

- When should client code redirect to login and when should it show a permission error?
- How does browser storage differ from HTTP cookies?

### Advanced and Follow-up

- What risks are introduced by storing tokens in `localStorage`?
- Why can a browser app be vulnerable to CSRF even when the API is protected?

### Code Prediction

Given a response with `401` for an expired token, explain whether the client should retry automatically, clear the user session, or redirect to a login flow.

## Practical Tasks

- Explain the difference between session cookies and token-based auth in a browser app.
- Map a `401`/`403` result to one of several UI states: login prompt, permission error, or retry after refresh.

## Readiness Criteria

You can explain browser storage, cookie behavior, auth flow consequences, and how to handle `401`/`403` consistently without hiding real authorization problems.

## References

- [MDN: HTTP cookies](https://developer.mozilla.org/en-US/docs/Web/HTTP/Cookies)
- [MDN: Web Storage API](https://developer.mozilla.org/en-US/docs/Web/API/Web_Storage_API)
- [OWASP: CSRF Prevention Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Cross-Site_Request_Forgery_Prevention_Cheat_Sheet.html)
- [MDN: HTTP status 401](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/401)
- [MDN: HTTP status 403](https://developer.mozilla.org/en-US/docs/Web/HTTP/Status/403)
