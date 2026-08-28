# Input Validation and Injection Prevention

## Definition

Input validation checks that incoming data conforms to what's expected (type, format, range, allowed values) before it's used — a first line of defense against a broad class of attacks, of which SQL injection (covered in depth in Module 9) is the most well-known specific instance. Validation works best as an **allow-list** (explicitly define what's acceptable) rather than a **deny-list** (try to block known-bad patterns), since deny-lists are perpetually incomplete against creative attackers.

```csharp
// Deny-list: fragile — trying to enumerate every dangerous pattern is a losing game
if (input.Contains("<script>") || input.Contains("DROP TABLE")) { /* reject */ }

// Allow-list: robust — only permits what's explicitly known to be safe
if (!Regex.IsMatch(input, @"^[a-zA-Z0-9\-]{1,50}$")) return BadRequest();
```

## Alternatives & Trade-offs

A deny-list tries to enumerate every known-bad pattern, which is fundamentally incomplete — there's always another encoding, another edge case, another way to express the same attack that wasn't on the list. An allow-list defines the much smaller, much more tractable set of what's actually valid, rejecting everything else by default — a stronger, more maintainable posture, though it requires actually knowing and encoding the legitimate shape of the data upfront.

## How It Works

### SQL injection — a recap, with the real fix living in Module 9

```csharp
// The vulnerability (Module 9's sql-injection-prevention.md covers this in full):
var sql = $"SELECT * FROM Users WHERE Username = '{username}'"; // NEVER concatenate untrusted input into SQL

// The actual fix is parameterization, not "better" input validation:
var sql = "SELECT * FROM Users WHERE Username = @Username";
command.Parameters.AddWithValue("@Username", username);
```

Input validation is a useful defense-in-depth layer here (rejecting a username containing SQL-special characters before it even reaches the query), but the *structural* fix against SQL injection is parameterization — validation alone is not a substitute for it.

### Command injection — the same principle, a different sink

```csharp
// VULNERABLE: untrusted input concatenated into a shell command
Process.Start("ping", $"-c 4 {userSuppliedHost}"); // userSuppliedHost = "google.com; rm -rf /" is catastrophic

// SAFER: use argument arrays instead of a concatenated command string where the API supports it,
// and validate the input against a strict allow-list (e.g., a valid hostname pattern) regardless
if (!Regex.IsMatch(userSuppliedHost, @"^[a-zA-Z0-9\.\-]+$")) return BadRequest();
```

Any time untrusted input flows into something that gets *interpreted* (SQL, a shell command, a regular expression engine, an XML parser resolving external entities), the same pattern applies: validate strictly, and prefer an API that separates data from the interpreted structure entirely (parameterization, argument arrays) over string concatenation.

### Validating at the right layer(s) — defense in depth, not either/or

```csharp
// Request-level validation (Module 8's validation.md) — is this well-formed?
[Required, RegularExpression(@"^[a-zA-Z0-9\-]{1,50}$")]
public string ProductCode { get; set; } = "";

// Still parameterize the actual query, regardless of validation having already run
var sql = "SELECT * FROM Products WHERE Code = @Code";
```

Validation and parameterization aren't competing solutions — validation narrows what's accepted at the boundary; parameterization/safe APIs prevent misinterpretation regardless of what slips through.

## Application

Validate every external input against an explicit allow-list of what's actually valid for that field, as close to the boundary as possible. Treat validation as one layer of defense-in-depth, not a substitute for the structural fix (parameterized queries, argument arrays, safe APIs) appropriate to wherever that input eventually flows.

## Common Mistakes

- Relying on a deny-list of known-bad patterns instead of an allow-list of known-good ones, leaving the door open to any pattern the list didn't anticipate.
- Treating input validation as sufficient protection against SQL injection on its own, without also parameterizing queries.
- Concatenating untrusted input into any interpreted context (SQL, shell commands, XML, regular expressions) and assuming validation alone makes that safe.
- Validating only at one layer (e.g., only client-side, or only in a UI form) and trusting that the same data reaching an API or backend service through a different path is therefore already safe.

## Common Interview Questions

### Basic
- What's the difference between an allow-list and a deny-list approach to input validation?
- Why is a deny-list generally weaker than an allow-list?

### Intermediate
- Why doesn't input validation alone fully prevent SQL injection, even though it helps?
- What is command injection, and how is it structurally similar to SQL injection?

### Advanced
- How would you validate and safely handle user-supplied input that needs to be passed to an external process or a regular expression engine?
- How would you audit a codebase for places where untrusted input flows into an interpreted context without adequate protection?

### Follow-up Questions
- Should input be validated only at the API boundary, or at every layer that receives it?
- Is client-side-only validation ever sufficient for security purposes?

### Code Prediction
An API validates that a `hostname` field matches a hostname-shaped regex before passing it to a shell command via string concatenation. Does this validation alone make the shell command construction safe, or is a further structural change also needed?

## Practical Tasks

- Convert a deny-list-based validation rule into an allow-list-based one for a specific input field.
- Identify a place where validated input still flows into an interpreted context via string concatenation, and fix the underlying construction (not just the validation).
- Design validation and safe-construction rules for a field that will be used in both a SQL query and a shell command.

## Readiness Criteria

Prefer allow-lists over deny-lists for input validation, recognize that validation is one layer of defense rather than a complete fix, and correctly pair validation with structural protections (parameterization, argument arrays) for whatever interpreted context the input eventually reaches.

## References

### Microsoft Learn

- [Model validation in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/models/validation)

### Other

- [OWASP: Input Validation Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Input_Validation_Cheat_Sheet.html)
- [SQL injection prevention (Module 9)](../m09-relational-databases-and-sql/sql-injection-prevention.md)
