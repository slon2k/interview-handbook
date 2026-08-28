# Password Storage

## Definition

Passwords must never be stored in plain text or reversibly encrypted — they're stored as the output of a **slow, salted hash function** designed specifically for password hashing (bcrypt, Argon2, PBKDF2), so that even a full database leak doesn't directly expose usable passwords, and cracking them requires expensive, deliberately slow brute-force effort per guess.

```csharp
// ASP.NET Core Identity uses PBKDF2 by default
var hasher = new PasswordHasher<User>();
string hashedPassword = hasher.HashPassword(user, "PlainTextPassword123!");
PasswordVerificationResult result = hasher.VerifyHashedPassword(user, hashedPassword, "PlainTextPassword123!");
```

## Alternatives & Trade-offs

A fast, general-purpose hash (SHA-256, MD5) computes in microseconds — great for verifying file integrity, terrible for passwords, because that same speed lets an attacker with a leaked hash database try billions of guesses per second. Password-specific hash functions (bcrypt, Argon2, PBKDF2) are deliberately slow and tunable (an explicit "work factor" or iteration count), making each guess expensive — the whole point is trading a small, tolerable delay for legitimate logins against a much larger delay for brute-force attacks.

## How It Works

### Why a fast general-purpose hash is wrong for passwords

```csharp
// WRONG: SHA-256 is fast, deterministic, and has no salt — identical passwords produce identical hashes,
// and a leaked database can be brute-forced or matched against precomputed "rainbow tables" quickly
var hash = SHA256.HashData(Encoding.UTF8.GetBytes(password));
```

### Salting — defeating precomputed lookup tables

```
Same password, no salt:   "password123" -> always the same hash, for every user, everywhere
Same password, with salt: "password123" + random salt per user -> a different hash for every user,
                           even though the underlying password is identical
```

A unique, random salt per password means an attacker can't precompute one lookup table that works against every leaked hash in the database — they'd need to redo the expensive computation separately for every single salt.

### PBKDF2/bcrypt/Argon2 — deliberately slow, with a tunable cost factor

```csharp
var hasher = new PasswordHasher<User>(); // ASP.NET Core Identity's default: PBKDF2 with a configurable iteration count
string hash = hasher.HashPassword(user, plainTextPassword);
// internally: a random salt is generated, and the password is hashed thousands of iterations of PBKDF2 —
// deliberately expensive per guess, and the salt+hash+iteration count are stored together
```

The iteration count (work factor) can be increased over time as hardware gets faster, keeping the deliberate slowness meaningful even years later — a fixed-cost fast hash has no equivalent way to "catch up."

### Verifying — never by re-hashing and string-comparing manually with a fast hash

```csharp
PasswordVerificationResult result = hasher.VerifyHashedPassword(user, storedHash, providedPassword);
if (result == PasswordVerificationResult.SuccessRehashNeeded)
{
    // the stored hash used an older, weaker work factor — re-hash with current settings on next successful login
    var newHash = hasher.HashPassword(user, providedPassword);
}
```

## Application

Always use a library or framework feature purpose-built for password hashing (ASP.NET Core Identity's `PasswordHasher<T>`, or bcrypt/Argon2 directly) rather than a general-purpose hash function. Never build custom salting or hashing logic from scratch — this is exactly the kind of "don't roll your own crypto" area where a purpose-built, widely-reviewed implementation matters far more than cleverness.

## Common Mistakes

- Using a fast, general-purpose hash function (MD5, SHA-256) for passwords instead of a purpose-built, deliberately slow algorithm.
- Storing passwords without a unique per-user salt, leaving them vulnerable to precomputed lookup-table attacks.
- Rolling a custom password-hashing scheme instead of using an established library, introducing subtle vulnerabilities a well-reviewed library would have already handled correctly.
- Never increasing the work factor over time as hardware improves, letting a hashing scheme that was adequately slow years ago become comparatively fast and crackable today.

## Common Interview Questions

### Basic
- Why shouldn't passwords be stored in plain text or with reversible encryption?
- Why is a fast hash function (SHA-256) inappropriate for password storage?

### Intermediate
- What does salting a password hash protect against specifically?
- What does it mean for a password hashing algorithm to have a tunable "work factor," and why does that matter?

### Advanced
- How would you migrate an existing user base from a weaker hashing scheme to a stronger one without forcing every user to reset their password immediately?
- Why is "don't roll your own crypto" specifically important advice in the context of password storage?

### Follow-up Questions
- Does salting alone make a fast hash function (like SHA-256) acceptable for passwords?
- What's the practical difference between bcrypt, Argon2, and PBKDF2 for most application purposes?

### Code Prediction
Two users both choose the password `"password123"`, hashed with a purpose-built algorithm using a unique random salt per user. Are their stored hashes identical? Why does that matter for an attacker with access to the leaked hash database?

## Practical Tasks

- Implement password hashing and verification using ASP.NET Core Identity's `PasswordHasher<T>`.
- Explain, for a code review, why a proposed custom SHA-256-based password scheme is inadequate, and what to use instead.
- Design a migration strategy for gradually upgrading stored password hashes to a stronger work factor without disrupting users.

## Readiness Criteria

Explain why password-specific hashing (not general-purpose hashing) is required, correctly use a purpose-built library, and design a safe migration path for strengthening hash parameters over time.

## References

### Microsoft Learn

- [Account confirmation and password recovery in ASP.NET Core](https://learn.microsoft.com/aspnet/core/security/authentication/accconfirm)
- [PasswordHasher<TUser> class](https://learn.microsoft.com/dotnet/api/microsoft.aspnetcore.identity.passwordhasher-1)

### Other

- [OWASP: Password Storage Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Password_Storage_Cheat_Sheet.html)
