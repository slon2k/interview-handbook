# Assemblies, Executables, Class Libraries, and Deployment Models

## Definition

An **assembly** is the unit of compiled output and deployment in .NET — a `.dll` or `.exe` file containing IL, metadata, and a manifest. A **class library** produces a `.dll` meant to be referenced by other projects, not run directly. An **executable** produces something runnable directly. Module 15 covers the two deployment modes — **framework-dependent** (requires the target machine to have a matching runtime installed) and **self-contained** (bundles the runtime itself) — in practical depth; this topic covers the underlying concept.

```
Class library project  -> MyLibrary.dll   -> referenced by other projects, not run directly
Executable project      -> MyApp.dll (+ a native apphost executable) -> run directly
```

## Alternatives & Trade-offs

A framework-dependent deployment produces a smaller output (no bundled runtime) but requires the target environment to already have a compatible .NET runtime installed — appropriate when you control or can guarantee that environment (a container image already based on a .NET runtime image, an internal server with a managed runtime install). A self-contained deployment bundles the runtime into the output, guaranteeing it runs regardless of what's installed on the target, at the cost of a significantly larger deployment artifact.

## How It Works

### Assemblies as the fundamental deployment unit

```
An assembly (.dll/.exe) contains:
  - IL (the compiled code itself)
  - Metadata describing its own types, methods, and their signatures
  - A manifest listing which OTHER assemblies it depends on
```

This self-describing nature (an assembly knows its own dependencies via metadata) is part of what makes .NET's reflection (Module 2) and dependency resolution possible — an assembly doesn't need external documentation to know what it needs.

### Modern executables — a managed assembly plus a native "apphost"

```
Even an "executable" .NET project actually produces a managed .dll containing the IL, PLUS
a small native apphost executable (MyApp.exe on Windows, or a native launcher on other
platforms) that knows how to locate and start the .NET runtime to run that .dll.
```

This is why you sometimes see both `MyApp.dll` and `MyApp.exe` (or its platform equivalent) in a build output directory — they're not duplicates; the apphost is a thin native launcher, and the actual application logic is the managed `.dll` it starts.

### Framework-dependent vs. self-contained, conceptually

```
Framework-dependent: the published output assumes a matching .NET runtime is already present
                      on the target machine — smaller output, requires that assumption to hold.
Self-contained:       the published output bundles the specific runtime version it needs,
                      guaranteeing it runs regardless of what's installed on the target —
                      larger output, no external assumption needed.
```

Module 15 covers choosing between these for real deployment scenarios (containers, bare servers) in depth; the concept itself is a platform-level idea worth understanding independent of any specific deployment pipeline.

## Application

Understand the difference between a class library (referenced, not run) and an executable (run directly) when structuring a solution's projects. Understand framework-dependent vs. self-contained conceptually as a fundamental trade-off (target-environment assumption vs. output size), even though Module 15 covers the practical decision-making for a specific deployment target.

## Common Mistakes

- Assuming a .NET "executable" is a single native binary with no separate managed assembly underneath, missing the apphost/managed-DLL split.
- Confusing framework-dependent deployment's smaller output with it being universally "better," without considering whether the target environment's runtime availability can actually be guaranteed.
- Not understanding that an assembly's manifest self-describes its own dependencies, missing why .NET's reflection and dependency resolution don't require separate external metadata files.
- Referencing a class library project as if it could be run directly, when its output isn't meant to be executed on its own.

## Common Interview Questions

### Basic
- What is an assembly in .NET?
- What's the difference between a class library and an executable project?

### Intermediate
- What's the conceptual difference between framework-dependent and self-contained deployment?
- What is an "apphost," and how does it relate to a managed .dll?

### Advanced
- How does an assembly's manifest enable dependency resolution without external metadata files?
- How would you decide, for a given deployment target, whether framework-dependent or self-contained deployment is the better fit? (See Module 15 for the full practical decision.)

### Follow-up Questions
- Does every .NET executable project produce a native binary with no managed code at all?
- Can a self-contained deployment run on a machine with no .NET runtime installed at all?

### Code Prediction
A framework-dependent application is deployed to a container built from a base image that doesn't include any .NET runtime. What happens when the container attempts to start the application?

## Practical Tasks

- Build a small executable project and inspect the output directory, identifying both the managed assembly and the native apphost.
- Explain, for two different deployment targets (a bare VM with no guaranteed runtime, a container based on a matching runtime image), which deployment mode fits each.
- Reference a class library project from an executable project and explain why the library's own output isn't meant to be run directly.

## Readiness Criteria

Explain assemblies, the apphost/managed-DLL split, and the framework-dependent/self-contained trade-off conceptually, connecting to Module 15's practical deployment decision-making.

## References

### Microsoft Learn

- [.NET application publishing overview](https://learn.microsoft.com/dotnet/core/deploying/)
- [Build and publish commands, and project configuration (Module 15)](../m15-development-workflow-and-delivery/build-publish-and-project-configuration.md)
