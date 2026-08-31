# NuGet Packages and Dependency Basics

## Definition

NuGet is .NET's package format and distribution mechanism — a package bundles compiled code (and metadata about its own dependencies) for reuse across projects, retrieved from a package source (nuget.org by default, or a private feed) rather than written from scratch. This topic covers the basic concept; ongoing dependency management (versioning discipline, vulnerability auditing, update cadence) is covered in depth in Module 15.

```xml
<ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
</ItemGroup>
```

```bash
dotnet add package Newtonsoft.Json
dotnet restore   # downloads and resolves all PackageReference entries and their own dependencies
```

## Alternatives & Trade-offs

Writing functionality from scratch avoids taking on an external dependency's maintenance burden and security surface, but re-implements something the ecosystem has often already solved well. Using a NuGet package saves that implementation effort and benefits from wider community testing, at the cost of trusting external code and needing to keep it updated over time (Module 15's dependency-management content).

## How It Works

### A package can itself depend on other packages — resolved automatically

```
Your project references PackageA.
PackageA itself references PackageB (a TRANSITIVE dependency, from your project's perspective).
`dotnet restore` resolves and downloads the ENTIRE dependency graph, not just what you
directly referenced.
```

### Where packages actually come from

```xml
<!-- NuGet.config — specifying package sources, including a private feed alongside nuget.org -->
<configuration>
  <packageSources>
    <add key="nuget.org" value="https://api.nuget.org/v3/index.json" />
    <add key="CompanyFeed" value="https://mycompany.pkgs.visualstudio.com/_packaging/feed/nuget/v3/index.json" />
  </packageSources>
</configuration>
```

Organizations often run a private NuGet feed for internal, proprietary packages shared across teams, alongside the public nuget.org source for third-party packages.

### The package cache — restore doesn't always mean a network download

```
Restored packages are cached locally (typically under ~/.nuget/packages). A subsequent
`dotnet restore` for a package version already in the cache uses the cached copy rather
than re-downloading it — relevant for understanding why restore is sometimes instant and
sometimes slow, depending on cache state.
```

### A package's own metadata drives compatibility

```
A package specifies which target framework(s) it supports (e.g., netstandard2.0, net8.0).
Attempting to reference a package that doesn't support your project's target framework at
all fails at restore/build time, rather than at runtime.
```

## Application

Use NuGet packages for well-solved, non-core-business-logic functionality (JSON serialization, logging providers, cloud SDKs) rather than reimplementing them. Understand that referencing one package can pull in a whole graph of transitive dependencies, which matters for the vulnerability-auditing discipline covered in Module 15.

## Common Mistakes

- Not realizing that a directly-referenced package's own dependencies (transitive dependencies) are part of what actually gets restored and shipped, only thinking about the packages explicitly listed in the `.csproj`.
- Adding a heavyweight package for functionality that's trivial to implement directly, taking on unnecessary maintenance and security surface for little benefit.
- Confusing "restore" with "always downloads from the network," when a populated local package cache often makes restore fast and offline-capable.
- Not configuring a private feed correctly for internal packages, leading to confusing "package not found" errors that are actually a package-source configuration issue.

## Common Interview Questions

### Basic
- What is a NuGet package, and where does `dotnet restore` get them from?
- What is a transitive dependency?

### Intermediate
- How does a package's target framework compatibility affect whether it can be referenced from a given project?
- Why might a company run a private NuGet feed alongside the public nuget.org source?

### Advanced
- How does the local package cache affect restore performance and offline capability?
- How would you decide whether a new functionality need justifies adding a NuGet dependency versus implementing it directly? (See Module 15 for the deeper dependency-management discipline.)

### Follow-up Questions
- Does `dotnet restore` always require a network connection?
- Can two different packages referenced by the same project depend on different versions of a third, shared package?

### Code Prediction
A project references `PackageA`, which transitively depends on `PackageB` version `2.0`. The project never mentions `PackageB` directly in its own `.csproj`. Does `PackageB` still get restored and included in the build output?

## Practical Tasks

- Add a NuGet package reference to a sample project and inspect its transitive dependencies using `dotnet list package --include-transitive`.
- Configure a `NuGet.config` referencing both nuget.org and a hypothetical private feed.
- Explain, for a specific piece of functionality, whether it justifies adding a NuGet dependency or implementing it directly.

## Readiness Criteria

Explain what a NuGet package is and where it comes from, understand transitive dependency resolution, and make a reasoned decision about when a dependency is worth adding.

## References

### Microsoft Learn

- [NuGet package management](https://learn.microsoft.com/nuget/consume-packages/overview-and-workflow)

### Other

- [NuGet dependency management (Module 15)](../m15-development-workflow-and-delivery/nuget-dependency-management.md)
