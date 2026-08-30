# Docker Fundamentals

## Definition

Docker packages an application together with everything it needs to run — runtime, dependencies, configuration — into a portable, isolated **container**, built from a **Dockerfile** describing how to assemble it. The core promise: "it works on my machine" becomes "it works in this exact, reproducible container," regardless of what's installed on the host machine underneath.

```dockerfile
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS base
WORKDIR /app
COPY . .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

```bash
docker build -t myapp:latest .
docker run -p 8080:80 myapp:latest
```

## Alternatives & Trade-offs

Deploying directly onto a host machine (installing the .NET runtime, copying files, configuring a web server) works but ties the deployment to that specific machine's exact installed software versions and configuration — "works here" can silently mean "works only because of this machine's specific accumulated state." A container bundles everything the application needs into one portable unit, guaranteeing the same behavior across any host with a container runtime, at the cost of an extra abstraction layer and some learning curve for the Docker-specific concepts and tooling.

## How It Works

### Multi-stage builds — smaller, more secure final images

```dockerfile
# Stage 1: build (needs the full SDK — compiler, tooling)
FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build
WORKDIR /src
COPY . .
RUN dotnet publish -c Release -o /app/publish

# Stage 2: run (only needs the runtime, not the full SDK)
FROM mcr.microsoft.com/dotnet/aspnet:8.0 AS final
WORKDIR /app
COPY --from=build /app/publish .
ENTRYPOINT ["dotnet", "MyApp.dll"]
```

The final image only contains the runtime and the published output — none of the SDK, source code, or build tooling used to produce it — resulting in a smaller image with a smaller attack surface (Module 12's least-privilege thinking, applied to what ships in a container).

### Layer caching — why Dockerfile instruction order matters for build speed

```dockerfile
COPY *.csproj .
RUN dotnet restore          # this layer is cached and REUSED as long as the .csproj files haven't changed
COPY . .                     # copying the rest of the source AFTER restore means source-only changes
RUN dotnet build              # don't invalidate the (often slow) restore layer's cache
```

Docker caches each instruction's result as a layer; if an earlier instruction's inputs haven't changed, Docker reuses the cached layer instead of re-running it — ordering instructions from least-frequently-changing to most-frequently-changing (dependencies before source code) keeps rebuilds fast.

### Images vs. containers — the blueprint vs. the running instance

```
Image:     the built, immutable blueprint (like a class)
Container: a running (or stopped) INSTANCE of that image (like an object) — you can run
           multiple containers from the same image simultaneously, each with its own isolated
           filesystem/process space, sharing the same underlying image layers
```

### Environment configuration inside a container

```bash
docker run -e ASPNETCORE_ENVIRONMENT=Production -e ConnectionStrings__Default="..." myapp:latest
```

The environment-specific-settings topic's approach (the same binary, configured via environment variables) maps directly onto how a container is typically configured at `docker run`/orchestration time — the image itself stays identical across environments.

## Application

Use multi-stage Dockerfiles to keep final images small and free of build-time tooling. Order Dockerfile instructions to maximize layer-cache reuse, restoring dependencies before copying the rest of the source. Configure environment-specific behavior via environment variables passed at container-run time, keeping the image itself identical across environments.

## Common Mistakes

- Using a single-stage Dockerfile that ships the full SDK and source code in the final image, unnecessarily bloating size and attack surface.
- Ordering Dockerfile instructions so that copying source code happens before dependency restoration, invalidating the cache on every single source change and slowing every rebuild.
- Baking environment-specific configuration directly into the image instead of injecting it at run time, requiring a separate image build per environment.
- Confusing an image with a container, leading to confusion about why multiple containers can run from the same image simultaneously without interfering with each other.

## Common Interview Questions

### Basic
- What is a Docker container, and how does it differ from a Docker image?
- What is a multi-stage Docker build, and what problem does it solve?

### Intermediate
- Why does Dockerfile instruction order affect build speed, via layer caching?
- Why is it preferable to configure environment-specific behavior at container-run time rather than baking it into the image?

### Advanced
- How would you design a multi-stage Dockerfile that minimizes both final image size and rebuild time for a typical ASP.NET Core application?
- How does containerization relate to the statelessness discussion from Module 14, in terms of what makes a container easy to scale horizontally?

### Follow-up Questions
- Can multiple containers run simultaneously from the same image?
- Does Docker layer caching work the same way across different machines/CI runners, or is it local to one machine by default?

### Code Prediction
Given a Dockerfile that copies all source code (including the `.csproj` files) before running `dotnet restore`, if only a single `.cs` file's implementation changes (no dependency changes), does Docker's layer cache still need to re-run `dotnet restore`? What Dockerfile reordering would avoid that?

## Practical Tasks

- Write a multi-stage Dockerfile for a small ASP.NET Core application, verifying the final image excludes the SDK and source code.
- Reorder a Dockerfile's instructions to maximize layer-cache reuse across source-only changes, and measure the rebuild-speed difference.
- Run the same image as multiple simultaneous containers, configured differently via environment variables at run time.

## Readiness Criteria

Write efficient multi-stage Dockerfiles, understand layer caching well enough to order instructions for fast rebuilds, and configure environment-specific behavior at run time rather than baking it into images.

## References

### Microsoft Learn

- [Containerize a .NET app](https://learn.microsoft.com/dotnet/core/docker/build-container)

### Other

- [Docker documentation: Dockerfile best practices](https://docs.docker.com/build/building/best-practices/)
