# Module 1 - The .NET Platform and Development Model

**Status:** Complete  
**Priority:** Medium. Understand the current platform model; historical version-by-version memorisation is unnecessary.  
**Prerequisites:** None — this is the conceptual foundation everything else sits on top of, though it's deliberately placed last in the writing order since its content is lower-stakes for interviews than Modules 2-17.

## Scope

This module establishes what .NET actually is and how an application is built and executed — the platform model underneath everything else in this handbook. Several topics here (NuGet, build/publish, Debug/Release configuration) already have deep, practical treatment in Module 15; this module covers the conceptual "what is this and why does it exist" layer underneath that practical workflow knowledge, and cross-references rather than repeats it.

## Learning Outcomes

By the end of this module, you should be able to:

- Explain the .NET Framework / .NET Core / modern .NET distinction and the LTS/STS support model.
- Distinguish the SDK from the runtime, and use the core `dotnet` CLI workflow end to end.
- Explain the CLR and BCL's role as the common foundation underneath every kind of .NET application.
- Structure projects and solutions to enforce architectural boundaries at compile time.
- Explain compilation to IL, JIT compilation, and when Native AOT's trade-offs are worth it.
- Distinguish assemblies, executables, and class libraries, and reason about deployment models conceptually.
- Trace an application's execution lifecycle and reason about when different failure categories can occur.
- Reason about cross-platform execution and runtime/target-framework compatibility.
- Recognize a legacy .NET Framework codebase and assess migration effort realistically.

## Topics

### 1. The Platform and Its Tooling

- [.NET Framework, .NET Core, modern unified .NET, and version support](dotnet-editions-and-versions.md)
- [SDK, runtime, and the `dotnet` CLI](sdk-runtime-and-cli.md)
- [The CLR and the Base Class Library](clr-and-bcl.md)

### 2. Projects and Dependencies

- [Project and solution structure, and `.csproj` files](project-and-solution-structure.md)
- [NuGet packages and dependency basics](nuget-basics.md)
- [The `dotnet new`, `restore`, `build`, `run`, and `publish` workflow](dotnet-cli-workflow.md)

### 3. Compilation and Execution

- [Compilation to IL, JIT compilation, and basic AOT awareness](compilation-il-and-jit.md)
- [Assemblies, executables, class libraries, and deployment models](assemblies-and-deployment-models.md)
- [Debug and Release configurations (platform concept)](debug-and-release-configurations.md)
- [Application execution lifecycle](application-execution-lifecycle.md)

### 4. Portability and Legacy

- [Cross-platform execution, and runtime/target framework compatibility](cross-platform-and-runtime-compatibility.md)
- [Basic legacy .NET Framework awareness](legacy-net-framework-awareness.md)

## Scope Boundaries

- NuGet dependency *management* (versioning discipline, vulnerability auditing, update cadence) belongs in [Module 15 - Development Workflow and Delivery Fundamentals](../m15-development-workflow-and-delivery/nuget-dependency-management.md); this module covers only the basic concept of what a package is.
- Build/publish *practice* (choosing self-contained vs. framework-dependent for a real deployment target, CI pipeline steps) belongs in [Module 15](../m15-development-workflow-and-delivery/build-publish-and-project-configuration.md); this module covers the underlying platform concepts.
- Docker/containerization belongs in [Module 15](../m15-development-workflow-and-delivery/docker-fundamentals.md); this module's cross-platform topic explains *why* containerizing a .NET app is straightforward, without covering Docker itself.
- Garbage collection mechanics belong in [Module 5 - Exceptions, Resources, and Memory Management](../m05-exceptions-resources-memory/README.md); this module's CLR topic mentions GC only as one of the CLR's responsibilities.
- ASP.NET Core's specific hosting model and middleware pipeline belong in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/README.md); this module's execution-lifecycle topic covers the platform-level startup sequence underneath it.

## Suggested Learning Sequence

1. .NET editions/versions, SDK vs. runtime, the CLR and BCL.
2. Project/solution structure, NuGet basics, the core CLI workflow.
3. Compilation to IL/JIT, assemblies and deployment models, Debug/Release, the execution lifecycle.
4. Cross-platform execution and legacy .NET Framework awareness.

## Practical Deliverables

- Explain, for a new team member, the difference between .NET Framework and modern .NET, and which LTS version you'd recommend for a new project.
- Scaffold a new project with `dotnet new`, structure a multi-project solution enforcing a dependency direction, and walk through `restore`/`build`/`run`/`publish`.
- Explain the IL/JIT/AOT trade-off for a specific scenario (a long-running API vs. a frequently-invoked CLI tool).
- Assess, at a high level, the migration effort for a hypothetical legacy .NET Framework application.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and platform-model familiarity.
- Intermediate questions involving common tooling usage and configuration choices.
- Advanced questions involving migration, deployment, and execution-model reasoning.
- Follow-up questions that test precise understanding rather than memorized version history.
- Code-prediction questions grounded in concrete scenarios, kept lighter than other modules given this module's Medium priority and awareness-level scope.

## References

### Microsoft Learn

- [Overview of .NET](https://learn.microsoft.com/dotnet/core/introduction)
- [.NET release lifecycle](https://learn.microsoft.com/lifecycle/products/microsoft-net-and-net-core)
- [.NET architectural components](https://learn.microsoft.com/dotnet/standard/components)
- [.NET application publishing overview](https://learn.microsoft.com/dotnet/core/deploying/)
