# Basic Legacy .NET Framework Awareness

## Definition

.NET Framework (versions up to 4.8, the final release) is Windows-only, in permanent maintenance mode with no new features planned, but still runs a large amount of existing production software. This is awareness-level knowledge: recognizing what you'd be dealing with if you encountered a .NET Framework codebase, not deep expertise in its specifics — the roadmap explicitly treats historical version-by-version detail as unnecessary.

```
.NET Framework 4.8 — the final version, receiving only security patches, no new features,
                      no planned successor, still officially supported for the lifetime of
                      the Windows versions it ships with/is supported on.
```

## Alternatives & Trade-offs

Maintaining an existing .NET Framework application in place (rather than migrating it) avoids the real cost and risk of a migration project, and is entirely reasonable for a stable application with no pressing need to change. Migrating to modern .NET unlocks cross-platform deployment, active feature development, and performance improvements, at the cost of a genuine porting effort — API differences, removed features (some .NET Framework-only APIs, like certain WCF server-hosting scenarios, have no direct modern-.NET equivalent), and testing the migrated application thoroughly.

## How It Works

### What's genuinely different, at a level worth knowing

```
.NET Framework:  Windows-only; uses the older, verbose .csproj format (mostly, though newer
                  SDK-style is partially supported); ships with Windows itself in many
                  versions rather than being a separate install; some APIs (older ASP.NET
                  Web Forms/MVC, WCF hosting) have no direct modern-.NET equivalent.
Modern .NET:      cross-platform; SDK-style projects by default; installed and versioned
                  independently of the OS; actively developed with new features every year.
```

### Why migration is a real project, not a recompile

```
Common migration blockers:
  - Windows-specific APIs (registry access, certain WMI usage) that don't exist or behave
    differently cross-platform
  - Third-party libraries that were never updated for modern .NET
  - WCF server-hosting (client-side WCF has a modern-.NET-compatible path; hosting a WCF
    SERVICE does not, without a separate community project)
  - ASP.NET Web Forms specifically has no modern-.NET equivalent at all
```

The .NET Upgrade Assistant tool can automate much of the mechanical work (updating the `.csproj` format, flagging incompatible API usage) but genuine architectural blockers still require real engineering decisions, not just tooling.

### Why this still matters for a mid-level interview

```
Many real companies run a mix of legacy .NET Framework systems and modern .NET services —
being able to recognize what you're looking at, and having a realistic sense of what
migrating it would actually involve, is more valuable awareness than memorizing .NET
Framework's historical version-by-version feature additions.
```

## Application

Recognize a .NET Framework codebase by its characteristic signals (older project file format, Windows-only APIs, an ASP.NET Web Forms/MVC 5-style structure) and have a realistic, high-level sense of what migrating it to modern .NET would involve — without needing deep .NET Framework expertise for a role focused on modern .NET development.

## Common Mistakes

- Assuming .NET Framework and modern .NET are close enough that migration is close to a simple recompile, underestimating real blockers like WCF hosting or Windows-specific API usage.
- Memorizing .NET Framework's historical version-by-version feature timeline, which the roadmap explicitly flags as unnecessary depth for this awareness-level topic.
- Not recognizing common signals of a .NET Framework codebase (older project format, certain namespace usage) when encountering one.
- Assuming .NET Framework is no longer supported at all, when it remains supported (patches only, no new features) for a long time yet, tied to the Windows versions it ships with.

## Common Interview Questions

### Basic
- Is .NET Framework still supported, and is it receiving new features?
- What's a key limitation of .NET Framework compared to modern .NET?

### Intermediate
- What are some real blockers that make migrating a .NET Framework application to modern .NET a genuine project rather than a simple recompile?
- What does the .NET Upgrade Assistant help with, and what does it not solve automatically?

### Advanced
- How would you assess, at a high level, the migration effort for an unfamiliar .NET Framework codebase before committing to a migration project?
- Why does ASP.NET Web Forms specifically have no modern-.NET migration path, unlike most other .NET Framework APIs?

### Follow-up Questions
- Can a .NET Framework and a modern .NET project exist in the same solution?
- Does .NET Framework run on Linux or macOS?

### Code Prediction
A team estimates migrating a large .NET Framework application to modern .NET will take "a week" based on running the .NET Upgrade Assistant once. What kinds of real blockers (from the list above) could this estimate be missing entirely?

## Practical Tasks

- Identify the characteristic signals that would tell you a codebase targets .NET Framework rather than modern .NET, without needing to check the target framework setting directly.
- Describe, at a high level, the migration assessment you'd want to do before estimating effort for porting a hypothetical .NET Framework application.

## Readiness Criteria

Recognize .NET Framework's current status and key differences from modern .NET, and reason realistically about migration effort and blockers, without needing deep .NET-Framework-specific expertise.

## References

### Microsoft Learn

- [.NET Framework overview](https://learn.microsoft.com/dotnet/framework/get-started/overview)
- [.NET Upgrade Assistant overview](https://learn.microsoft.com/dotnet/core/porting/upgrade-assistant-overview)
- [Port from .NET Framework to .NET](https://learn.microsoft.com/dotnet/core/porting/)
