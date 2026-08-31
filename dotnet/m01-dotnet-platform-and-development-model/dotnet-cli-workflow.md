# The `dotnet new`, `restore`, `build`, `run`, and `publish` Workflow

## Definition

These five commands cover the full lifecycle of a .NET project from creation to deployable output. This topic covers the workflow conceptually, end to end; Module 15 covers `build` and `publish` specifically in much greater practical depth (Debug vs. Release, self-contained vs. framework-dependent).

```bash
dotnet new webapi -n MyApi     # scaffold a new project from a template
cd MyApi
dotnet restore                  # resolve NuGet dependencies
dotnet build                     # compile
dotnet run                        # build + run, for local development
dotnet publish -c Release -o ./out # produce deployable output
```

## Alternatives & Trade-offs

`dotnet run` bundles restore, build, and execution into one convenient command for local development iteration — fast to type, but conflates several distinct steps that a deployment pipeline (Module 15) needs to control independently and verify separately. Running each step explicitly (`restore`, then `build`, then a separate test/publish step) gives that independent control, which is exactly why a CI/CD pipeline never just runs `dotnet run`.

## How It Works

### `dotnet new` — scaffolding from a template

```bash
dotnet new list                 # see available templates
dotnet new webapi -n MyApi       # scaffold a new ASP.NET Core Web API project named MyApi
dotnet new classlib -n MyLibrary  # scaffold a class library instead
```

Templates provide a working starting structure (a `.csproj`, a `Program.cs`, sensible defaults) rather than starting from a completely empty folder — third-party templates can also be installed for more specialized starting points.

### `dotnet restore` — resolving dependencies before anything can compile

```bash
dotnet restore
```

This is usually run implicitly by `build`/`run`/`publish` if needed, but can be run explicitly — relevant in CI pipelines wanting a distinct, cacheable "restore" step separate from "build" (Module 15's Docker layer-caching content uses exactly this separation).

### `dotnet build` vs. `dotnet run` vs. `dotnet publish`

```
build:    compiles the project — output is suitable for local testing/debugging, not deployment
run:      build + immediately execute — the fast local development loop
publish:  produces deployment-ready output (Module 15 covers Debug/Release and self-contained/
          framework-dependent choices for this command in depth)
```

The relationship: `publish` does everything `build` does, plus additional steps (optimization, potentially bundling the runtime) to produce something actually appropriate to deploy — `build`'s output alone isn't what you'd want running in production.

### The order these commands make sense in, end to end

```
dotnet new       -> create the project structure
dotnet restore    -> resolve dependencies (often implicit)
dotnet build       -> verify it compiles, during development
dotnet run          -> iterate locally
dotnet publish       -> produce the actual deployable artifact, once ready to ship
```

## Application

Use `dotnet new` to scaffold new projects from an appropriate template rather than building the initial structure by hand. Use `dotnet run` for the fast local development loop. Use `dotnet build` and `dotnet publish` as distinct, separately-verifiable steps in any real deployment pipeline (Module 15), never conflating them with the convenience of `dotnet run`.

## Common Mistakes

- Using `dotnet run` (or its underlying `build` output) as if it were appropriate for deployment, when `dotnet publish` specifically produces the output meant for that purpose.
- Not understanding that `restore` is a distinct, cacheable step, missing the Docker/CI layer-caching opportunity Module 15 covers.
- Building custom project scaffolding by hand instead of starting from an appropriate `dotnet new` template.
- Assuming `dotnet build` and `dotnet publish` produce equivalent output, when publish includes additional steps specifically for deployment readiness.

## Common Interview Questions

### Basic
- What does `dotnet new` do?
- What's the difference between `dotnet build` and `dotnet publish`?

### Intermediate
- Why does `dotnet run` bundle several steps together, and why might a CI pipeline avoid doing the same?
- What's the practical difference between `dotnet build`'s output and `dotnet publish`'s output?

### Advanced
- How would you structure a CI pipeline's use of `restore`, `build`, and `publish` as separate, cacheable, independently-verifiable steps?
- Why might restore be run as an explicitly separate step in a Dockerfile rather than relying on it happening implicitly during build?

### Follow-up Questions
- Does `dotnet publish` always require running `dotnet restore` and `dotnet build` first, or does it handle that internally?
- Can custom project templates be installed beyond the built-in ones?

### Code Prediction
A CI pipeline runs only `dotnet run` as its single build step, expecting it to produce a deployable artifact. What's actually missing from this pipeline, given what `dotnet run` is actually designed to do?

## Practical Tasks

- Scaffold a new project using `dotnet new`, then walk through `restore`, `build`, `run`, and `publish` in sequence, noting what each step actually produces.
- Design a CI pipeline step sequence using `restore`, `build`, and `publish` as distinct steps rather than relying on `dotnet run`.
- Explore available `dotnet new` templates and choose an appropriate one for a hypothetical new class library project.

## Readiness Criteria

Explain what each of the five core CLI commands does and how they relate to each other, and correctly use `restore`/`build`/`publish` as distinct pipeline steps rather than conflating them with `dotnet run`'s local-development convenience.

## References

### Microsoft Learn

- [dotnet new command](https://learn.microsoft.com/dotnet/core/tools/dotnet-new)
- [dotnet restore command](https://learn.microsoft.com/dotnet/core/tools/dotnet-restore)
- [Build and publish commands, and project configuration (Module 15)](../m15-development-workflow-and-delivery/build-publish-and-project-configuration.md)
