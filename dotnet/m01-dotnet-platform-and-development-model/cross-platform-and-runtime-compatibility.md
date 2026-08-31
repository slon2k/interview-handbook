# Cross-Platform Execution, and Runtime/Target Framework Compatibility

## Definition

Modern .NET runs natively on Windows, Linux, and macOS, across multiple CPU architectures (x64, ARM64) — a deliberate design goal since .NET Core, in contrast to the Windows-only .NET Framework. A project's **target framework** (`net8.0`, etc., set in the `.csproj`) declares which .NET version's APIs it's written against; the actual **runtime** installed on a given machine must be compatible with that target for the application to run there at all.

```xml
<TargetFramework>net8.0</TargetFramework>
```

```bash
dotnet MyApp.dll   # requires a compatible net8.0 (or later, depending on rollForward policy) runtime installed
```

## Alternatives & Trade-offs

Targeting the exact same .NET version everywhere (development, CI, production) is the simplest, most predictable setup. Targeting a range or allowing roll-forward to a newer runtime patch version trades some of that exact predictability for automatically picking up runtime-level bug/security fixes without needing to recompile — a reasonable default for most applications, since patch-level runtime updates are meant to be safe.

## How It Works

### Why cross-platform matters practically, not just as a feature checkbox

```
The SAME compiled application (IL, from the compilation topic) can run on a Windows
development machine, a Linux CI runner, and a Linux production container — without
recompiling for each — because IL is CPU/OS-independent and the CLR itself is available
on each platform.
```

This is a big part of why Docker (Module 15) containerizing .NET applications is so straightforward — the same build output genuinely runs the same way across the Linux-based container images most commonly used in production, regardless of what platform a developer built it on locally.

### Target framework vs. installed runtime — compatibility rules

```
A net8.0-targeted application requires a .NET 8 (or later, if configured to roll forward)
runtime to actually run. It will NOT run on a machine with only .NET 6 installed — a
target framework declares a MINIMUM requirement, not a preference.
```

```xml
<!-- runtimeconfig.json, generated automatically, controls roll-forward behavior -->
{ "rollForward": "LatestMinor" }
```

Roll-forward policies control whether an application targeting `net8.0` can run on a slightly newer installed runtime (e.g., 8.0.5 instead of exactly 8.0.0) automatically, which matters for patch-level updates reaching production without requiring a full application rebuild.

### Architecture-specific considerations

```
Framework-dependent deployment (Module 15/this module's assemblies topic): the SAME output
  can generally run on x64 or ARM64, since the installed runtime handles the architecture-
  specific details.
Self-contained/AOT deployment: bundles or compiles for a SPECIFIC architecture — a
  self-contained build for x64 will not run on an ARM64 machine without a separate,
  architecture-specific build.
```

## Application

Rely on cross-platform execution as a genuine, dependable property of modern .NET for both development convenience (developing on any OS) and deployment flexibility (containerizing for Linux regardless of development platform). Understand target-framework-vs-runtime compatibility as a minimum-version requirement, and account for architecture-specific builds when using self-contained or AOT deployment across mixed-architecture environments.

## Common Mistakes

- Assuming a self-contained or AOT build for one CPU architecture will run unmodified on a different architecture, when those specific deployment modes are architecture-bound in a way framework-dependent deployment isn't.
- Confusing "target framework" (a minimum version requirement declared by the application) with "any installed runtime will do," missing that an older installed runtime than the target genuinely won't work.
- Not considering roll-forward policy, either missing beneficial patch-level runtime updates or being surprised by unexpected behavior from an unintentionally newer runtime.
- Assuming cross-platform support is a recent or partial feature, when it's been a core design goal since .NET Core and is now simply how the platform works.

## Common Interview Questions

### Basic
- Does modern .NET run on operating systems other than Windows?
- What does a project's target framework declare?

### Intermediate
- Can an application targeting `net8.0` run on a machine with only the .NET 6 runtime installed?
- What is roll-forward, and what does it control?

### Advanced
- Why does a self-contained or AOT-published application need a separate build per target CPU architecture, while a framework-dependent one generally doesn't?
- How does IL's platform-independence (from the compilation topic) enable the same build output to run correctly across Windows, Linux, and macOS?

### Follow-up Questions
- Does targeting `net8.0` mean an application can only run on exactly .NET 8.0.0?
- Is cross-platform support equally complete for every kind of .NET application (console, web API, desktop UI)?

### Code Prediction
An application is published self-contained for `linux-x64`. An attempt is made to run this exact published output on an ARM64-based Linux machine. What happens, and what would need to change about the publish command to support that target instead?

## Practical Tasks

- Explain, for a hypothetical mixed-architecture deployment (some x64 servers, some ARM64), whether framework-dependent or self-contained deployment simplifies the situation more.
- Configure a project's roll-forward policy and explain what runtime versions it would then accept.
- Verify that a given application's target framework is compatible with a specific installed runtime version before attempting to run it.

## Readiness Criteria

Explain cross-platform execution and why it's a genuine platform property (not just a checkbox), reason correctly about target-framework/runtime compatibility as a minimum-version rule, and account for architecture-specific builds where relevant.

## References

### Microsoft Learn

- [.NET on different platforms](https://learn.microsoft.com/dotnet/core/introduction)
- [Target frameworks in SDK-style projects](https://learn.microsoft.com/dotnet/standard/frameworks)
