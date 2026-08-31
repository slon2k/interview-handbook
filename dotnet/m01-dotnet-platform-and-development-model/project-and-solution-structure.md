# Project and Solution Structure, and `.csproj` Files

## Definition

A **project** (`.csproj`) describes one buildable unit — its source files, dependencies, and target framework. A **solution** (`.sln`, or the newer `.slnx`) groups multiple related projects together for tooling convenience (opening them all in an IDE at once, building them together) without implying they're deployed as one unit.

```xml
<Project Sdk="Microsoft.NET.Sdk.Web">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>
  </PropertyGroup>
  <ItemGroup>
    <PackageReference Include="Newtonsoft.Json" Version="13.0.3" />
  </ItemGroup>
</Project>
```

## Alternatives & Trade-offs

Putting everything in one project is simpler to navigate initially but makes it harder to enforce boundaries (Module 14's dependency-direction content) — nothing stops a domain class from referencing an infrastructure-specific type if they're compiled together in one project. Splitting into multiple projects (domain, application, infrastructure, API) within one solution lets project references *physically enforce* those boundaries — a project simply cannot compile if it references something not allowed — at the cost of more solution structure to navigate and maintain.

## How It Works

### Modern `.csproj` — much shorter than it used to be

```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
  </PropertyGroup>
</Project>
```

Modern SDK-style project files (post-.NET Core) are dramatically shorter than the old .NET Framework-era `.csproj` format, which explicitly listed every single source file — modern projects implicitly include all `.cs` files in their directory by convention, needing explicit entries only for exclusions or special cases.

### Project references enforce architectural boundaries at compile time

```xml
<!-- In Domain.csproj: no references to infrastructure at all -->
<ItemGroup>
  <!-- deliberately empty of any infrastructure project reference -->
</ItemGroup>

<!-- In Infrastructure.csproj: references Domain, implementing its interfaces -->
<ItemGroup>
  <ProjectReference Include="..\Domain\Domain.csproj" />
</ItemGroup>
```

This is the concrete mechanism behind Module 14's "dependency direction should point inward" — if `Domain.csproj` simply has no reference to `Infrastructure.csproj` at all, it's not just a convention that infrastructure code stays out of the domain; it's a compile-time impossibility for the domain project to accidentally reference infrastructure types.

### Solutions as an organizational, not architectural, concept

```
MySolution.sln
  Domain/Domain.csproj
  Application/Application.csproj
  Infrastructure/Infrastructure.csproj
  Api/Api.csproj
```

The solution file itself doesn't enforce anything — it's a convenience for opening and building related projects together. The *actual* architectural enforcement comes from which projects reference which others, exactly as shown above.

### Multi-targeting a single project for multiple frameworks

```xml
<TargetFrameworks>net8.0;net472</TargetFrameworks>
```

Relevant specifically for library authors needing to support consumers on different .NET versions (Module 15 covers this at the build/publish level) — an application project usually targets just one framework.

## Application

Split a codebase into multiple projects within one solution specifically to enforce architectural boundaries at compile time (Module 14), not just for organizational tidiness. Keep `.csproj` files minimal, relying on SDK-style conventions rather than explicitly listing every file.

## Common Mistakes

- Keeping an entire application in one project when the architecture (Module 14) actually calls for enforced boundaries between layers, missing the chance to have those boundaries checked by the compiler rather than just code review.
- Adding an unnecessary `ProjectReference` "just in case," accidentally opening a boundary that was meant to stay closed.
- Assuming the solution file itself enforces any structure, when it's purely an IDE/build convenience and the real enforcement is in project references.
- Using the old, verbose .NET Framework-era `.csproj` format style out of habit when the modern SDK-style format is shorter and equally capable.

## Common Interview Questions

### Basic
- What's the difference between a project (`.csproj`) and a solution (`.sln`)?
- What does a `PackageReference` in a `.csproj` file represent?

### Intermediate
- How do project references enforce architectural boundaries at compile time?
- Why are modern `.csproj` files much shorter than older ones?

### Advanced
- How would you structure a solution's projects to enforce the dependency-direction discipline from Module 14?
- What's the risk of adding an unnecessary project reference "just in case" it's needed later?

### Follow-up Questions
- Does a solution file get compiled into the application?
- Can one project multi-target more than one framework version?

### Code Prediction
Given `Domain.csproj` with no reference to `Infrastructure.csproj`, what happens if a developer tries to write code in the Domain project that directly instantiates a type defined only in Infrastructure?

## Practical Tasks

- Design a multi-project solution structure (domain, application, infrastructure, API) for a small application, with project references enforcing the correct dependency direction.
- Convert an old-style, verbose `.csproj` file to the modern SDK-style format.
- Attempt to create an architecturally-invalid reference (infrastructure referenced from domain) and observe the compile-time consequence.

## Readiness Criteria

Distinguish projects from solutions precisely, use project references to physically enforce architectural boundaries, and write minimal, modern SDK-style project files.

## References

### Microsoft Learn

- [.csproj file format](https://learn.microsoft.com/dotnet/core/project-sdk/msbuild-props)
- [Solution files (.sln)](https://learn.microsoft.com/visualstudio/extensibility/internals/solution-dot-sln-file)
