# .NET Framework, .NET Core, Modern Unified .NET, and Version Support

## Definition

".NET" has referred to three genuinely distinct things over time: **.NET Framework** (Windows-only, the original, now in maintenance mode with no new feature development), **.NET Core** (the cross-platform rewrite, versions 1.0 through 3.1), and **modern .NET** (from .NET 5 onward, which unified .NET Core, Xamarin, and Mono into one platform and dropped the "Core" name entirely). Releases alternate between **STS** (Standard Term Support, ~18 months) and **LTS** (Long Term Support, ~3 years) — odd-numbered versions are STS, even-numbered are LTS.

```
.NET Framework 4.8   -> Windows-only, legacy, maintenance mode
.NET Core 3.1 (LTS)  -> cross-platform, last release still called "Core"
.NET 5 (STS)         -> renamed, unified platform begins
.NET 6 (LTS), .NET 8 (LTS), .NET 10 (LTS, upcoming) -> the even-numbered LTS releases
```

## Alternatives & Trade-offs

.NET Framework is only relevant today for maintaining existing legacy applications — it receives no new features, and no new project should target it. Modern .NET (5+) is cross-platform, actively developed, and the correct default for any new project. Within modern .NET, choosing an LTS release trades a slightly older feature set for three years of support and stability; choosing the latest STS release gets newer features and performance improvements sooner, at the cost of a shorter support window and more frequent required upgrades.

## How It Works

### Why "Core" disappeared from the name

```
.NET Core existed specifically to distinguish the new, cross-platform rewrite from the old
.NET Framework. Once .NET Framework was clearly the legacy option and .NET Core WAS "just .NET"
going forward, Microsoft dropped "Core" from the name at version 5 (skipping 4, to avoid
confusion with .NET Framework 4.x) — there is no ".NET Core 5"; it's just ".NET 5".
```

### LTS vs. STS in practice

```
LTS (even-numbered: 6, 8, 10, ...):  ~3 years of support — the default choice for most
                                       production applications, especially ones that won't be
                                       upgraded frequently.
STS (odd-numbered: 5, 7, 9, ...):    ~18 months of support — appropriate when you want the
                                       newest features and are committed to upgrading regularly.
```

Choosing STS for a system that then isn't upgraded on schedule means running an unsupported, unpatched runtime — a real operational risk (Module 12's patching discipline applies here too).

### .NET Standard — a compatibility layer, mostly historical now

```
.NET Standard was a specification (not a runtime) that a library could target to be usable
from BOTH .NET Framework and .NET Core/modern .NET simultaneously. Modern .NET's own
cross-targeting and the shrinking relevance of .NET Framework have made .NET Standard mostly
a historical detail for new library development, though it still appears in older codebases.
```

## Application

Target the current LTS release for new production applications by default. Use the latest STS release specifically when a new feature it introduces is worth the shorter support window and commitment to upgrade sooner. Never start a new project on .NET Framework.

## Common Mistakes

- Confusing "does this project use .NET Framework or .NET Core" with "which specific version" — these are two different, commonly conflated questions.
- Choosing an STS release for a long-lived production system without a real plan to upgrade before support ends.
- Assuming .NET Framework applications can be trivially moved to modern .NET without any porting effort — the runtimes are different enough that migration is a real project, not a recompile.
- Memorizing exact version-by-version release dates and feature lists, when the roadmap explicitly treats this as unnecessary — understanding the LTS/STS model and current recommendation matters more.

## Common Interview Questions

### Basic
- What's the difference between .NET Framework and modern .NET?
- What does LTS mean, and how often are LTS versions released?

### Intermediate
- Why did Microsoft drop "Core" from the name at version 5?
- When would you choose an STS release over the current LTS release?

### Advanced
- What's involved in migrating an application from .NET Framework to modern .NET, at a high level?
- What role did .NET Standard play, and why is it less relevant for new development today?

### Follow-up Questions
- Is .NET Framework still receiving security patches?
- Can a single solution contain both a .NET Framework project and a modern .NET project?

### Code Prediction
Given a production system on .NET 7 (STS, supported for ~18 months) with no upgrade planned before its support window ends, what operational risk does this create, and what would the safer choice have been at the time of the original decision?

## Practical Tasks

- Identify the current LTS and STS .NET versions, and explain which you'd recommend for a new production service.
- Explain, for a hypothetical legacy .NET Framework application, what a realistic migration path to modern .NET would involve at a high level.

## Readiness Criteria

Distinguish .NET Framework, .NET Core, and modern .NET precisely, explain the LTS/STS support model, and recommend an appropriate target version for a given scenario without needing to memorize historical release details.

## References

### Microsoft Learn

- [.NET release lifecycle](https://learn.microsoft.com/lifecycle/products/microsoft-net-and-net-core)
- [Overview of .NET](https://learn.microsoft.com/dotnet/core/introduction)
