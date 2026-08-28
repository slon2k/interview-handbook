# Mass Assignment / Over-Posting

## Definition

Mass assignment (also called over-posting) is a vulnerability where model binding (Module 8) automatically populates more fields than the caller should legitimately be able to set — for example, a client including `"isAdmin": true` in a request body that gets bound directly onto a domain entity with an `IsAdmin` property never intended to be publicly settable.

```csharp
// VULNERABLE: binds the request body directly onto the full entity, including fields never meant to be client-settable
[HttpPut("{id}")]
public async Task<IActionResult> Update(int id, User user)
{
    _context.Users.Update(user); // if the request body includes "isAdmin": true, it gets set — nothing stopped it
    await _context.SaveChangesAsync();
    return Ok();
}
```

## Alternatives & Trade-offs

Binding directly to a domain/database entity is less code upfront — no separate DTO to define and map — but exposes every public property on that entity to client-controlled input implicitly, including fields that should never be client-settable (`IsAdmin`, `CreatedAt`, `AccountBalance`). Using a dedicated request DTO with only the fields that should actually be settable takes a bit more code, but makes the settable surface explicit and auditable at a glance, rather than depending on every entity property staying "safe" indefinitely as the entity evolves.

## How It Works

### The vulnerability, concretely

```csharp
public class User
{
    public int Id { get; set; }
    public string Email { get; set; } = "";
    public bool IsAdmin { get; set; } // never meant to be settable by an ordinary user
}

[HttpPut("{id}")]
public async Task<IActionResult> Update(int id, User user) // binds the FULL entity from the request body
{
    _context.Users.Update(user);
    await _context.SaveChangesAsync();
}
```

```json
PUT /users/42
{ "id": 42, "email": "alice@example.com", "isAdmin": true }
```

Nothing in this code path distinguishes "fields the client should be allowed to change" from "fields that happen to be public properties" — model binding populates all of them from the request body without discrimination.

### The fix — a dedicated request DTO with only the intended fields

```csharp
public class UpdateUserRequest
{
    public string Email { get; set; } = ""; // IsAdmin is deliberately absent — it CANNOT be set via this DTO at all
}

[HttpPut("{id}")]
public async Task<IActionResult> Update(int id, UpdateUserRequest request)
{
    var user = await _context.Users.FindAsync(id);
    user.Email = request.Email; // explicit, field-by-field — IsAdmin is never touched by this code path
    await _context.SaveChangesAsync();
}
```

The DTO's shape *is* the contract — there's no property on it that could accidentally leak client control over a sensitive field, because it was never defined there in the first place.

### `[Bind]`/explicit include-lists — a weaker, more error-prone alternative

```csharp
[HttpPut("{id}")]
public async Task<IActionResult> Update(int id, [Bind("Email")] User user) // only binds the Email property
```

This works but is more fragile than a dedicated DTO — every new sensitive property added to `User` later needs someone to remember this include-list exists and hasn't been bypassed elsewhere; a DTO's absence of the field is self-enforcing.

## Application

Never bind request bodies directly onto domain or database entities that have any field which shouldn't be freely client-settable. Use a dedicated request DTO per operation, containing only the fields that operation should legitimately accept, and map explicitly from the DTO onto the entity.

## Common Mistakes

- Binding directly to a full domain entity for convenience, exposing every public property (including sensitive ones) to client-controlled input implicitly.
- Relying on `[Bind]` include-lists instead of a dedicated DTO, creating a fragile safeguard that must be remembered and maintained as the entity evolves.
- Assuming client-side form fields (which don't include the sensitive field) are sufficient protection, forgetting that an attacker can send an arbitrary request body directly, bypassing any client-side UI entirely.
- Not reviewing new entity properties for over-posting risk as an entity grows over time, especially ones added for internal/administrative purposes.

## Common Interview Questions

### Basic
- What is mass assignment / over-posting?
- Why is binding a request body directly to a domain entity risky?

### Intermediate
- How does a dedicated request DTO prevent over-posting, compared to a `[Bind]` include-list?
- Why isn't client-side form validation sufficient protection against this vulnerability?

### Advanced
- How would you audit an existing codebase for over-posting risk across many endpoints binding directly to entities?
- How does this vulnerability specifically relate to EF Core's change-tracking and `Attach`-based update patterns from Module 10?

### Follow-up Questions
- Does using data annotations for validation (Module 8) prevent mass assignment?
- Is mass assignment specific to `PUT`/`PATCH` operations, or can it affect `POST` (creation) as well?

### Code Prediction
Given the vulnerable `Update` action above, what request body would let an attacker who is an ordinary authenticated user grant themselves admin privileges, assuming no other authorization check exists on the `IsAdmin` field specifically?

## Practical Tasks

- Identify an endpoint binding directly to a domain entity and refactor it to use a dedicated request DTO.
- Reproduce the over-posting vulnerability against a sample entity with a sensitive field, then fix it.
- Audit a small codebase for endpoints at risk of mass assignment and prioritize fixes by the sensitivity of the exposed fields.

## Readiness Criteria

Recognize mass assignment vulnerabilities in model-bound endpoints, and consistently use dedicated request DTOs rather than binding directly to entities with sensitive fields.

## References

### Microsoft Learn

- [Model binding in ASP.NET Core](https://learn.microsoft.com/aspnet/core/mvc/models/model-binding)

### Other

- [OWASP: Mass Assignment Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Mass_Assignment_Cheat_Sheet.html)
