# Application Execution Lifecycle

## Definition

The sequence a .NET application actually goes through from being launched to running your code: the OS starts a process, the apphost (previous topic) locates and starts the CLR, the CLR loads the entry assembly and its dependencies, JIT-compiles and executes the entry point (`Main`, or the top-level statements in modern C#), and — for a hosted application like ASP.NET Core (Module 8) — the generic host builds configuration, DI, and the request pipeline before the application actually starts serving traffic.

```
OS starts process -> apphost locates/starts CLR -> CLR loads entry assembly + dependencies
                   -> JIT compiles entry point -> Main()/top-level statements execute
                   -> (ASP.NET Core specifically) host builds config/DI/pipeline -> app.Run()
```

## Alternatives & Trade-offs

Understanding this lifecycle isn't about choosing between alternatives — it's foundational context that explains *why* certain things behave the way they do elsewhere in this handbook: why Module 8's `WebApplicationBuilder`/`WebApplication` split exists (configuration/DI setup happens before the pipeline starts handling requests), why a missing assembly produces a startup-time failure rather than a runtime one, and why the very first request to a freshly-started application can be slower (JIT warmup, previous topic).

## How It Works

### A plain console application's lifecycle

```csharp
// Modern C# top-level statements — this IS the entry point, no explicit Main() needed
Console.WriteLine("Starting");
DoWork();
```

```
1. OS starts the process, running the apphost
2. Apphost locates and starts the CLR
3. CLR loads the entry assembly (containing this code) and resolves its dependencies
   (other assemblies it references)
4. JIT compiles the entry point's IL just before executing it (previous topic)
5. Code runs top to bottom; process exits when it completes (or is terminated)
```

### An ASP.NET Core application's more elaborate startup

```csharp
var builder = WebApplication.CreateBuilder(args); // configuration, DI container setup begins here
builder.Services.AddControllers();
var app = builder.Build();                          // DI container is finalized/built HERE
app.UseAuthentication();
app.MapControllers();
app.Run();                                            // NOW the app actually starts accepting requests
```

```
1-4: same as the console app (process start, CLR start, assembly load, JIT of entry point)
5. WebApplicationBuilder assembles configuration (Module 8) and DI registrations — nothing
   is actually built or running yet, just being configured
6. builder.Build() finalizes the DI container and middleware pipeline configuration
7. app.Run() starts Kestrel (Module 8) and begins actually accepting HTTP requests
```

This explains why a missing required configuration value or a broken DI registration (Module 8's `ValidateOnBuild`) can be caught as a *startup* failure — the app never reaches step 7 at all — rather than only failing later when a specific request finally exercises the broken path.

### What happens on a missing dependency

```
If the CLR can't locate an assembly a loaded assembly depends on, this fails at the point
that assembly is actually needed (often, but not always, at startup) — a
FileNotFoundException or similar, distinct from a compile-time error, since this is purely
a RUNTIME resolution failure that the compiler couldn't have caught (e.g., if a dependency
was present at build time but missing from the deployed output).
```

## Application

Use this lifecycle model to reason about *when* a given failure could occur — a missing assembly, a broken DI registration, an invalid configuration value — and why some failures happen at startup (before `app.Run()`/before the entry point even finishes) while others only happen when a specific code path is actually exercised later.

## Common Mistakes

- Assuming all failures are either "compile-time" or "happen when a specific feature is used," missing the distinct category of startup-time failures that occur during host/DI/configuration setup, before any request is even served.
- Not understanding why `WebApplicationBuilder`/`builder.Build()`/`app.Run()` are separate steps, treating ASP.NET Core startup as one opaque operation rather than a sequence with distinct, individually-reasoned-about phases.
- Confusing a missing-assembly runtime failure with a compile-time error, when the assembly might have been present at build time but simply missing from what was actually deployed.

## Common Interview Questions

### Basic
- What are the rough stages an application goes through from being launched to running your code?
- What's the difference in startup lifecycle between a plain console app and an ASP.NET Core app?

### Intermediate
- Why can a broken DI registration or missing configuration value cause a startup failure rather than a runtime failure during a specific request?
- What's the difference between `WebApplicationBuilder`, `builder.Build()`, and `app.Run()` in terms of what's happening at each step?

### Advanced
- How would you explain, to someone confused by a "FileNotFoundException" at startup, why this is a distinct failure category from a compile-time error?
- How does understanding this lifecycle explain the timing of `ValidateOnBuild`/`ValidateOnStart` checks (Module 8) catching problems before any request is served?

### Follow-up Questions
- Does every .NET application go through the exact same lifecycle stages, regardless of application type?
- Can a startup failure occur after `app.Run()` has already started accepting requests?

### Code Prediction
A required configuration value is missing, and the application uses `ValidateOnStart()` (Module 8) for that setting. At which stage of the lifecycle described above does this failure actually surface — during configuration building, during `builder.Build()`, or only once a specific request needs that value?

## Practical Tasks

- Trace, step by step, what happens between launching a console application and its first line of code executing.
- Trace the same for an ASP.NET Core application, identifying the point at which a broken DI registration would surface as a failure.
- Reproduce a missing-assembly runtime failure by removing a dependency from a deployed output (but not from the build), and observe when the failure actually occurs.

## Readiness Criteria

Explain the application execution lifecycle for both plain and hosted (ASP.NET Core) applications, and reason precisely about at which stage a given kind of failure would actually surface.

## References

### Microsoft Learn

- [.NET application startup](https://learn.microsoft.com/dotnet/core/tutorials/top-level-templates)
- [ASP.NET Core fundamentals overview](https://learn.microsoft.com/aspnet/core/fundamentals/)
