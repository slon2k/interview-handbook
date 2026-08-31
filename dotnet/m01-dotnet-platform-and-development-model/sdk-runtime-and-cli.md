# SDK, Runtime, and the `dotnet` CLI

## Definition

The **SDK** (Software Development Kit) includes everything needed to *build* .NET applications — compilers, project templates, the CLI tooling. The **runtime** includes only what's needed to *run* an already-built application — no compiler, no build tooling. The `dotnet` CLI is the command-line entry point to both, dispatching to the right underlying tool based on the command.

```bash
dotnet --list-sdks        # SDKs installed — needed to BUILD
dotnet --list-runtimes     # runtimes installed — needed to RUN
```

## Alternatives & Trade-offs

Installing only the runtime on a deployment target keeps that machine's footprint smaller and its attack surface narrower (no compiler, no build tooling that isn't needed there), appropriate for a production server that only ever runs already-built applications. Installing the full SDK is necessary on any machine that builds code — a developer's machine, a CI runner — since the runtime alone can't compile anything.

## How It Works

### The SDK contains the runtime, but not vice versa

```
SDK installation includes:  the runtime + compilers (Roslyn for C#) + MSBuild + project
                              templates (`dotnet new`) + the full `dotnet` CLI surface
Runtime-only installation:  just what's needed to execute an already-compiled application —
                              `dotnet MyApp.dll` works, `dotnet build` does not
```

This is why a production deployment target often only needs the runtime (or, per Module 8/15, might not need even that if the app is published self-contained) while a CI build agent needs the full SDK.

### Multiple SDKs and runtimes can coexist

```bash
dotnet --list-sdks
# 6.0.418 [/usr/share/dotnet/sdk]
# 8.0.101 [/usr/share/dotnet/sdk]
```

A machine can have several SDK versions installed side by side; a `global.json` file in a project/repo can pin which specific SDK version `dotnet` commands should use for that project, ensuring consistent behavior across different developers' machines and CI.

```json
{ "sdk": { "version": "8.0.100" } }
```

### The `dotnet` CLI as a dispatcher

```bash
dotnet new webapi       # scaffolds a new project from a template
dotnet restore           # resolves NuGet dependencies (Module 15's territory for the ongoing management of this)
dotnet build              # compiles (SDK required)
dotnet run                 # builds and runs in one step, for local development
dotnet MyApp.dll            # runs an ALREADY-BUILT application directly — this specific
                              # form works even with only the runtime installed, no SDK needed
dotnet publish -c Release    # produces deployment-ready output (Module 15's build-and-publish
                                # content covers this in depth)
```

### Runtime and SDK selection when multiple are installed

```
A project's TargetFramework (e.g., net8.0) determines which RUNTIME version it needs to run.
The SDK used to BUILD it can be a newer SDK than the target framework, as long as that SDK
supports building for the older target — SDK version and target framework version are related
but independently chosen.
```

## Application

Install the full SDK on any machine that builds code (developer machines, CI runners); install only the runtime on deployment targets that don't need to build anything, unless a self-contained publish (Module 15) removes even that requirement. Use `global.json` to pin a specific SDK version across a team/CI for consistency.

## Common Mistakes

- Assuming the runtime alone is sufficient to run `dotnet build` or `dotnet run`, when those specifically require the SDK.
- Not pinning an SDK version via `global.json` for a team project, leading to inconsistent build behavior across different developers' machines with different SDK versions installed.
- Confusing "which .NET version does my project target" (the `TargetFramework` in the `.csproj`) with "which SDK version is installed" — related, but not the same setting.
- Installing the full SDK on a production deployment target unnecessarily, when only the runtime (or nothing, for a self-contained publish) is actually needed there.

## Common Interview Questions

### Basic
- What's the difference between the .NET SDK and the .NET runtime?
- What does `dotnet run` do that `dotnet MyApp.dll` doesn't require?

### Intermediate
- Why might a production server only need the runtime installed, not the full SDK?
- What does `global.json` do, and why would a team use it?

### Advanced
- How does SDK version relate to a project's target framework — can a newer SDK build for an older target framework?
- How would you decide what to install on a CI runner versus a production deployment target?

### Follow-up Questions
- Can multiple SDK versions be installed side by side on the same machine?
- Does `dotnet MyApp.dll` require the SDK to be installed?

### Code Prediction
A production server has only the .NET 8 runtime installed (no SDK). A deployment script attempts to run `dotnet build` on that server. What happens, and what should the deployment script have done instead?

## Practical Tasks

- Check which SDKs and runtimes are installed on a given machine using `dotnet --list-sdks`/`--list-runtimes`.
- Create a `global.json` pinning a specific SDK version for a sample project.
- Explain, for a given deployment target, whether it needs the SDK, the runtime only, or neither (for a self-contained publish).

## Readiness Criteria

Distinguish the SDK from the runtime precisely, explain what each `dotnet` CLI command actually requires, and use `global.json` to pin SDK versions consistently across a team.

## References

### Microsoft Learn

- [.NET SDK overview](https://learn.microsoft.com/dotnet/core/sdk)
- [dotnet command](https://learn.microsoft.com/dotnet/core/tools/dotnet)
- [global.json overview](https://learn.microsoft.com/dotnet/core/tools/global-json)
