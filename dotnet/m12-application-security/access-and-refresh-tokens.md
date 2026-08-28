# Access and Refresh Tokens

## Definition

An **access token** is short-lived and sent with every API request to prove identity/permissions. A **refresh token** is longer-lived, stored more carefully, and used only to obtain a new access token when the current one expires — without forcing the user to log in again. Splitting the two lets access tokens stay short-lived (limiting damage if leaked) while avoiding the poor user experience of frequent re-authentication.

```
Access token:  short-lived (minutes), sent on every request, limited blast radius if leaked
Refresh token: long-lived (days/weeks), sent only to the token endpoint, higher-value target if leaked
```

## Alternatives & Trade-offs

A single long-lived access token is simpler (no refresh flow to implement) but means a leaked token stays dangerous for its entire lifetime, and revoking it early requires infrastructure a stateless JWT doesn't naturally have. The access/refresh split limits an access token's exposure window to minutes while keeping the more sensitive, longer-lived refresh token out of every ordinary API request — it's sent far less often, reducing its exposure surface — at the cost of implementing and securing the refresh flow itself.

## How It Works

### The renewal flow

```
1. Client authenticates, receives an access token (15 min) and a refresh token (30 days)
2. Client calls the API using the access token on every request
3. After 15 minutes, the access token expires; the API starts rejecting it with 401
4. Client calls a dedicated /refresh endpoint with the refresh token
5. Server validates the refresh token, issues a NEW access token (and often a new refresh token — see rotation below)
6. Client resumes calling the API with the new access token, without the user re-entering credentials
```

```csharp
[HttpPost("refresh")]
public async Task<IActionResult> Refresh(RefreshRequest request)
{
    var storedToken = await _tokenStore.FindAsync(request.RefreshToken);
    if (storedToken is null || storedToken.IsExpired || storedToken.IsRevoked)
        return Unauthorized();

    var newAccessToken = _tokenService.GenerateAccessToken(storedToken.UserId);
    return Ok(new { accessToken = newAccessToken });
}
```

### Refresh token rotation — limiting the damage of a leaked refresh token

```csharp
[HttpPost("refresh")]
public async Task<IActionResult> Refresh(RefreshRequest request)
{
    var storedToken = await _tokenStore.FindAsync(request.RefreshToken);
    if (storedToken is null || storedToken.IsExpired) return Unauthorized();

    if (storedToken.IsUsed) // this exact refresh token was already used once before — possible theft/replay
    {
        await _tokenStore.RevokeAllForUserAsync(storedToken.UserId); // treat as compromised, revoke everything
        return Unauthorized();
    }

    storedToken.IsUsed = true;
    var newRefreshToken = _tokenService.GenerateRefreshToken(storedToken.UserId); // issue a fresh one each time
    await _tokenStore.SaveAsync(newRefreshToken);
    return Ok(new { accessToken = _tokenService.GenerateAccessToken(storedToken.UserId), refreshToken = newRefreshToken.Value });
}
```

Rotating the refresh token on every use, and detecting reuse of an already-consumed one, is how a stolen-and-replayed refresh token gets caught — a legitimate client would never present the same, already-exchanged refresh token twice.

### Storage — refresh tokens deserve the more careful treatment

```
Access token:  can reasonably live in memory or a short-lived cookie, given its short exposure window
Refresh token: should be stored in an HttpOnly, Secure cookie or equivalent secure storage — never localStorage,
               given how much more damage a leaked long-lived token could do
```

## Application

Use short-lived access tokens (minutes) for ordinary API calls, and a longer-lived refresh token exchanged only with a dedicated, carefully-secured endpoint. Rotate refresh tokens on every use and detect reuse as a signal of compromise, revoking the whole token family if it occurs.

## Common Mistakes

- Making the access token itself long-lived to avoid implementing a refresh flow, leaving a leaked token dangerous for far too long.
- Storing the refresh token as carelessly as the access token, when its higher value as an attack target warrants stricter storage (HttpOnly cookie, not `localStorage`).
- Not rotating refresh tokens on use, missing the ability to detect and respond to a stolen-and-replayed refresh token.
- Failing to revoke the entire token family when reuse of an already-consumed refresh token is detected, treating it as an isolated event instead of a likely compromise.

## Common Interview Questions

### Basic
- What's the difference between an access token and a refresh token?
- Why not just use one long-lived token for everything?

### Intermediate
- What is refresh token rotation, and what attack does it help detect?
- Where should a refresh token be stored, and why more carefully than an access token?

### Advanced
- How would you design a response to detecting refresh-token reuse — what should happen to the user's other active sessions?
- How does the access/refresh split limit the blast radius of a leaked access token specifically?

### Follow-up Questions
- Does refresh token rotation require server-side storage of issued tokens, given that access tokens (JWTs) are often stateless?
- Can a refresh token itself be a JWT?

### Code Prediction
A refresh token is used to obtain a new access token, but the exact same refresh token is presented again five minutes later (having already been marked used and rotated). Under a rotation-with-reuse-detection scheme, what should happen — should the second request simply fail, or should something more drastic occur?

## Practical Tasks

- Implement an access/refresh token flow, including a `/refresh` endpoint and appropriate token lifetimes.
- Implement refresh token rotation with reuse detection, revoking all tokens for a user when reuse is detected.
- Design secure storage for both token types on a browser-based client, justifying the difference in treatment.

## Readiness Criteria

Explain the access/refresh split and its security rationale, implement rotation with reuse detection, and design appropriate storage for each token type based on its relative risk.

## References

### Microsoft Learn

- [JWT bearer authentication](https://learn.microsoft.com/aspnet/core/security/authentication/configure-jwt-bearer-authentication)

### Other

- [OAuth 2.0 refresh token best practices (IETF draft/BCP reference)](https://datatracker.ietf.org/doc/html/draft-ietf-oauth-security-topics)
