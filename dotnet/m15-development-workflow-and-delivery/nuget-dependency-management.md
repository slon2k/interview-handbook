# NuGet Dependency Management

## Definition

NuGet is .NET's package manager — a way to reference, version, and restore external libraries a project depends on. Managing dependencies well means understanding versioning semantics, keeping dependencies patched, and being deliberate about what a project actually pulls in, since every dependency is also a piece of code you're trusting and a potential source of vulnerabilities (Module 12's OWASP content mentioned this briefly).

```xml
<ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
</ItemGroup>
```

```bash
dotnet add package Newtonsoft.Json
dotnet restore
```

## Alternatives & Trade-offs

Pinning exact package versions gives maximum build reproducibility (the same version every time, everywhere) but means security patches and bug fixes require a deliberate, manual update. Using floating/range versions (`Version="13.*"`) automatically picks up patches, but risks an unexpected breaking change sneaking in silently between builds. Most teams pin exact versions for reproducibility and update deliberately, rather than floating and hoping.

## How It Works

### Semantic versioning — what a version number is supposed to communicate

```
MAJOR.MINOR.PATCH  (e.g., 13.0.3)

MAJOR: breaking changes — upgrading requires reviewing what changed
MINOR: new functionality, backward compatible — usually safe to upgrade
PATCH: bug fixes only, backward compatible — usually safe, often security-relevant
```

This convention (when followed correctly by a package author) lets you reason about upgrade risk from the version number alone — a PATCH bump should be safe to take without review; a MAJOR bump warrants reading the changelog first.

### Transitive dependencies — the dependencies of your dependencies

```
Your project references PackageA, which itself references PackageB version 2.0.
Your project never directly mentioned PackageB, but it's part of the build anyway —
a TRANSITIVE dependency. If PackageB has a known vulnerability, your project is affected
even though you never directly chose to depend on it.
```

```bash
dotnet list package --vulnerable --include-transitive
```

Auditing for vulnerable dependencies needs to include transitive ones, not just what's directly referenced — a real security exposure can hide several layers deep in a dependency tree nobody directly chose.

### Lock files — pinning the exact resolved dependency tree

```
packages.lock.json records the EXACT version of every direct and transitive dependency
actually used in a build, so a build today and the same build next month resolve to
identical versions even if a newer, floating-range-compatible version has since been published.
```

### Keeping dependencies patched without breaking things

```
A regular (e.g., monthly) dependency-update pass, updating patch/minor versions and running
the full test suite (Module 11), catches security patches and bug fixes before they become
urgent, rather than only updating reactively after a vulnerability is publicly disclosed and
already under active exploitation.
```

## Application

Pin exact package versions for build reproducibility, and update them deliberately on a regular cadence rather than floating or letting them go stale indefinitely. Audit for vulnerable dependencies including transitive ones, not just direct references. Use lock files where reproducibility across environments/time matters.

## Common Mistakes

- Letting dependencies go unupdated for a long time, accumulating both technical debt and unpatched known vulnerabilities.
- Auditing only direct dependencies for vulnerabilities, missing exposure through transitive dependencies several layers deep.
- Using floating version ranges without understanding the risk of an unexpected, silent breaking change between builds.
- Adding a new dependency for functionality that's trivial to implement directly, taking on the ongoing maintenance and security-surface cost of an external package for little real benefit.

## Common Interview Questions

### Basic
- What does semantic versioning (MAJOR.MINOR.PATCH) communicate about upgrade risk?
- What is a transitive dependency?

### Intermediate
- Why might floating/range package versions be riskier than pinned exact versions for build reproducibility?
- How would you check a project for known-vulnerable dependencies, including transitive ones?

### Advanced
- How would you design a regular dependency-update process that catches security patches proactively rather than reactively?
- How would you decide whether a new functionality need justifies adding an external dependency versus implementing it directly?

### Follow-up Questions
- Does a MAJOR version bump always mean an upgrade will break something?
- What's the purpose of a lock file if package versions are already pinned in the project file?

### Code Prediction
A project references `PackageA` version `2.1.0`, which transitively depends on `PackageB` version `1.5.0`, a version with a publicly known vulnerability. The project's own `.csproj` never mentions `PackageB` at all. Would a vulnerability scan limited to direct dependencies catch this exposure?

## Practical Tasks

- Run `dotnet list package --vulnerable --include-transitive` on a sample project and interpret the results.
- Set up a regular dependency-update process for a hypothetical project, including running the test suite after each update batch.
- Evaluate whether a specific piece of functionality justifies a new external dependency versus a direct implementation.

## Readiness Criteria

Reason about dependency versioning risk using semantic versioning, audit for vulnerabilities including transitive dependencies, and maintain a deliberate, proactive dependency-update discipline.

## References

### Microsoft Learn

- [NuGet package management](https://learn.microsoft.com/nuget/consume-packages/overview-and-workflow)
- [dotnet list package command](https://learn.microsoft.com/dotnet/core/tools/dotnet-list-package)
