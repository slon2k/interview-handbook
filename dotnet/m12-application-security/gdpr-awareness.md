# GDPR Awareness

## Definition

GDPR (General Data Protection Regulation) is EU law governing how personal data of EU individuals is collected, processed, and stored, regardless of where the processing company is based. For a developer, the practical relevance is a handful of specific rights and obligations that translate directly into concrete system requirements — this is awareness-level knowledge, not legal expertise; real compliance decisions involve legal counsel.

```
Data subject rights that typically require real system support:
  - Right to access   (export a user's personal data)
  - Right to erasure  ("right to be forgotten" — actually delete or anonymize a user's data)
  - Right to rectification (let a user correct inaccurate data)
  - Data portability  (export in a structured, commonly-used format)
```

## Alternatives & Trade-offs

Building these capabilities in from the start (data export, deletion/anonymization, granular consent tracking) costs upfront design effort. Retrofitting them later — after data is scattered across many tables, caches, logs, backups, and third-party services with no clear ownership — is significantly more expensive and error-prone, which is why "privacy by design" is a recurring theme in how GDPR-aware systems are actually built rather than an afterthought.

## How It Works

### "Right to erasure" interacts directly with earlier topics in this handbook

```csharp
// Soft delete (Module 10's global-query-filters.md) hides a row from ordinary queries,
// but the data is still physically present in the database — NOT sufficient for a genuine erasure request
public class Order { public bool IsDeleted { get; set; } }

// Genuine erasure requires either real deletion, or anonymization of personal fields specifically
public async Task AnonymizeUserAsync(int userId)
{
    var user = await _context.Users.FindAsync(userId);
    user.Email = $"deleted-{Guid.NewGuid()}@anonymized.local";
    user.Name = "Deleted User";
    // ... anonymize every field that constitutes personal data, while potentially retaining
    // non-personal aggregate data (like order totals for financial/legal record-keeping requirements)
}
```

This is a concrete, common gotcha: a system that implements "soft delete" for ordinary business reasons (Module 10) can create a false sense of GDPR compliance, when the personal data is still fully present and readable in the database.

### Data scattered across a system is harder to erase or export completely

```
Personal data often lives in more places than the primary database:
  - Application logs (Module 12's secure-logging.md — another reason not to log raw personal data)
  - Caches (Module 8/13's caching topics)
  - Backups
  - Third-party analytics/email services the data was ever sent to
```

A genuine erasure or export request needs to account for all of these, not just the primary `Users` table — which is exactly why minimizing where personal data is copied to in the first place (data minimization) makes both features and compliance easier.

### Data minimization — collect and retain only what's actually needed

```csharp
// Collecting a birthdate "in case it's useful someday" creates GDPR obligations (accuracy, erasure, etc.)
// for data that provides no current business value — minimizing collection reduces this surface area
```

### Consent and lawful basis (awareness level)

Processing personal data generally needs a lawful basis (consent, contractual necessity, legitimate interest, etc.) — for a developer, this usually surfaces as a requirement to track *what* consent was given, *when*, and to make it revocable, rather than requiring a developer to make the legal determination of which basis applies.

## Application

Design data models and features (especially anything holding personal data) with export and genuine erasure/anonymization in mind from the start, and be aware that soft delete alone doesn't satisfy a real erasure request. Minimize personal data collection and how widely it's copied (logs, caches, third parties) to keep both compliance and ordinary security exposure smaller. Escalate specific compliance determinations to legal/compliance expertise rather than treating this awareness-level knowledge as a substitute for it.

## Common Mistakes

- Assuming a soft-delete flag satisfies a "right to be forgotten" request, when the underlying personal data is still fully present in the database.
- Not accounting for personal data copied into logs, caches, backups, or third-party services when responding to an erasure or export request.
- Collecting more personal data than a feature actually needs, expanding both compliance obligations and general security exposure unnecessarily.
- Treating GDPR awareness as something only a legal team needs to think about, missing the concrete system-design requirements (export, erasure, consent tracking) that fall squarely on engineering to implement.

## Common Interview Questions

### Basic
- What is GDPR, at a high level relevant to a developer?
- What is the "right to erasure," and what does it typically require a system to support?

### Intermediate
- Why doesn't a soft-delete flag satisfy a genuine erasure request?
- Why does data minimization make both GDPR compliance and general security easier?

### Advanced
- How would you design a user-data-deletion feature that accounts for personal data scattered across logs, caches, and third-party services, not just the primary database?
- How would "privacy by design" change how you'd approach a new feature's data model from the start, compared to retrofitting compliance later?

### Follow-up Questions
- Is GDPR relevant only to companies based in the EU?
- Should engineers be expected to make legal determinations about lawful basis for processing, or escalate them?

### Code Prediction
A user submits a "right to be forgotten" request. The system sets `IsDeleted = true` on their user row (the existing soft-delete mechanism) and considers the request fulfilled. What personal data is still present and readable in the database and in any logs referencing that user's actions?

## Practical Tasks

- Design a genuine anonymization routine for a user entity, distinguishing personal fields (to anonymize) from non-personal aggregate data (potentially retained for other legitimate reasons).
- Identify all the places (logs, caches, third-party services) a specific piece of personal data might end up copied to in a hypothetical system, as input to a real erasure-request design.
- Propose a data-minimization review for a feature currently collecting more personal data than it uses.

## Readiness Criteria

Explain the practical, system-design implications of key GDPR rights (access, erasure, portability) at an awareness level, recognize that soft delete alone doesn't satisfy erasure, and design with data minimization and erasure/export in mind from the start.

## References

### Other

- [GDPR official text (Regulation (EU) 2016/679)](https://gdpr-info.eu/)
- [OWASP: Personal Data Protection](https://cheatsheetseries.owasp.org/cheatsheets/User_Privacy_Protection_Cheat_Sheet.html)
