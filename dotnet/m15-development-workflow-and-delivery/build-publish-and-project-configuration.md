# Build and Publish Commands, and Project Configuration

## Definition

`dotnet build` compiles a project into assemblies for local running/testing; `dotnet publish` produces the self-contained output actually meant for deployment — including only what's needed to run, potentially bundling the runtime itself. Project configuration (the `.csproj` file, build configurations like Debug/Release) controls how these commands behave.

```bash
dotnet build                     # compiles for local development/testing
dotnet publish -c Release -o ./out  # produces deployment-ready output
```

## Alternatives & Trade-offs

`dotnet build`'s output is optimized for fast local iteration (Debug configuration by default — includes debug symbols, skips some optimizations for faster builds and better debugging experience) and isn't what you'd actually want running in production. `dotnet publish` in Release configuration produces optimized, deployment-ready output, at the cost of being slower to produce and less convenient for the tight edit-run-debug loop of active development — which is exactly why they're different commands for different purposes, not the same thing used interchangeably.

## How It Works

### Debug vs. Release configuration

```xml
<PropertyGroup>
    <Configuration>Debug</Configuration> <!-- or Release -->
</PropertyGroup>
```

```bash
dotnet build -c Release   # Release: optimizations enabled, no debug-specific behavior
dotnet build -c Debug     # Debug: optimizations disabled for a more predictable debugging experience
```

Shipping a Debug build to production is a common, easy mistake — it's typically slower and behaves subtly differently (some code, like `Debug.Assert` calls, only runs in Debug builds at all) from what was actually tested as the "real" build.

### Self-contained vs. framework-dependent publish

```bash
dotnet publish -c Release --self-contained -r linux-x64  # bundles the .NET runtime itself — larger output, no runtime install needed on the target
dotnet publish -c Release                                   # framework-dependent — smaller output, requires the matching .NET runtime already installed on the target
```

Choosing between these is a real deployment-environment decision: a container image built from a base image that already has the runtime might prefer framework-dependent (smaller image); a bare deployment target with no guaranteed runtime needs self-contained.

### Key `.csproj` settings that affect build/publish behavior

```xml
<PropertyGroup>
    <TargetFramework>net8.0</TargetFramework>
    <Nullable>enable</Nullable>          <!-- Module 2's nullable reference types -->
    <TreatWarningsAsErrors>true</TreatWarningsAsErrors> <!-- ties into the static-analysis topic -->
    <InvariantGlobalization>true</InvariantGlobalization> <!-- smaller container images, at the cost of some culture-specific behavior -->
</PropertyGroup>
```

### Multi-targeting — building for more than one framework version from one project

```xml
<TargetFrameworks>net8.0;net472</TargetFrameworks>
```

Useful for a library that needs to support consumers on different .NET versions, producing separate outputs for each target from a single source project.

## Application

Use `dotnet build` for local development iteration and `dotnet publish -c Release` for anything actually being deployed. Choose self-contained vs. framework-dependent publish based on the target environment's guaranteed runtime availability. Set project-level configuration (`Nullable`, `TreatWarningsAsErrors`) deliberately as part of the team's quality bar, not just accepting scaffolded defaults.

## Common Mistakes

- Deploying a Debug-configuration build to production, shipping something slower and subtly different from what was actually tested.
- Confusing `dotnet build`'s output with what's actually appropriate to deploy, when `dotnet publish` is the command meant for that purpose.
- Not choosing deliberately between self-contained and framework-dependent publish for a given deployment target, leading to either unnecessarily large images or a missing-runtime failure.
- Leaving default, unreviewed `.csproj` settings in place rather than deliberately configuring things like nullable reference types or warnings-as-errors as part of the team's actual quality standards.

## Common Interview Questions

### Basic
- What's the difference between `dotnet build` and `dotnet publish`?
- What's the difference between Debug and Release configuration?

### Intermediate
- What's the difference between self-contained and framework-dependent publish, and when would you choose each?
- Why might shipping a Debug build to production cause subtle problems beyond just being slower?

### Advanced
- How would you decide, for a containerized deployment, whether self-contained or framework-dependent publish produces the better trade-off?
- How would you configure a project's `.csproj` to enforce a specific quality bar (nullable reference types, warnings-as-errors) across a whole team?

### Follow-up Questions
- Does `dotnet build` produce output suitable for production deployment?
- Can a single project multi-target more than one .NET version?

### Code Prediction
A container image is built from a base image that already includes the .NET runtime, but the application is published with `--self-contained`. What's the practical consequence for the resulting image size, and would framework-dependent publish have been the better choice here?

## Practical Tasks

- Compare the output of `dotnet build -c Debug` and `dotnet publish -c Release` for the same project, noting the differences.
- Publish a project as both self-contained and framework-dependent, comparing output size and runtime requirements.
- Configure a project's `.csproj` with `Nullable` and `TreatWarningsAsErrors` enabled, and resolve any resulting build issues.

## Readiness Criteria

Distinguish build from publish and Debug from Release configuration precisely, choose appropriately between self-contained and framework-dependent publish, and configure project settings deliberately rather than accepting unreviewed defaults.

## References

### Microsoft Learn

- [dotnet build command](https://learn.microsoft.com/dotnet/core/tools/dotnet-build)
- [dotnet publish command](https://learn.microsoft.com/dotnet/core/tools/dotnet-publish)
- [.NET application publishing overview](https://learn.microsoft.com/dotnet/core/deploying/)
