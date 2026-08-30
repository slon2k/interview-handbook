# Environment-Specific Settings

## Definition

The same application binary typically needs to behave slightly differently across environments — development, staging, production — different connection strings, different external API endpoints, different log verbosity. Module 8 already covered *how* ASP.NET Core layers configuration (`appsettings.{Environment}.json`, environment variables); this topic is about the *practice* of managing that difference correctly across a real deployment pipeline.

```bash
export ASPNETCORE_ENVIRONMENT=Production
```

```
appsettings.json                  <- shared defaults
appsettings.Development.json      <- overrides for local development
appsettings.Production.json       <- overrides for production
```

## Alternatives & Trade-offs

Maintaining separate full config files per environment is explicit but risks drifting out of sync (a new setting added to `Development` but forgotten in `Production`). Relying entirely on environment variables set at deploy time avoids that specific drift risk but makes the full set of configurable settings less discoverable by just reading the repository. Most teams use both: `appsettings.json` for structure and non-sensitive defaults, environment variables (or a secret store, Module 12) for the actual environment-specific and sensitive values layered on top.

## How It Works

### The same binary, different behavior, driven by `ASPNETCORE_ENVIRONMENT`

```csharp
var builder = WebApplication.CreateBuilder(args); // automatically reads ASPNETCORE_ENVIRONMENT
// and layers appsettings.{Environment}.json on top of appsettings.json accordingly
```

The application code and compiled binary are identical across environments — only the environment variable and the configuration it causes to load differ. This is important: you're deploying and testing the *same artifact* everywhere, with environment-specific configuration layered on at runtime, not rebuilding different binaries per environment (which would undermine confidence that what was tested in staging is truly what runs in production).

### What belongs in source-controlled config files vs. environment-injected values

```
appsettings.json / appsettings.{Environment}.json (source-controlled):
  - non-sensitive structural settings, feature toggles, logging levels, timeouts

Environment variables / secret store (injected at deploy time, NOT source-controlled):
  - connection strings with credentials, API keys — per Module 12's secret-management content
```

### Configuration validation — catching a missing environment-specific setting early

```csharp
builder.Services.AddOptions<ExternalApiOptions>()
    .Bind(builder.Configuration.GetSection("ExternalApi"))
    .ValidateOnStart(); // fails fast at startup if a required setting wasn't provided for THIS environment
```

Without this, a setting missing specifically in production (but present in development, where it was easy to forget it's environment-specific) might not surface as an error until the exact code path needing it actually runs — potentially much later, and in production.

### Testing configuration for a specific environment before it actually deploys there

```
A staging environment configured as closely as practical to match production's actual
settings (though obviously not sharing production's real secrets) is what actually validates
that production-specific configuration works, rather than discovering a misconfiguration
only after a real production deployment.
```

## Application

Keep non-sensitive, structural configuration in source-controlled `appsettings.{Environment}.json` files, and inject environment-specific/sensitive values via environment variables or a secret store at deploy time. Validate required configuration at startup so a missing environment-specific setting fails immediately and visibly, rather than only when a specific code path eventually needs it.

## Common Mistakes

- Letting environment-specific config files drift out of sync, where a new setting is added to one environment's file but forgotten in another's.
- Committing sensitive, environment-specific values (real connection strings, API keys) directly into a source-controlled `appsettings.{Environment}.json`, reopening Module 12's secret-management concerns.
- Not validating required configuration at startup, letting a missing environment-specific setting surface only when a specific, possibly rare code path finally needs it in production.
- Building different binaries per environment instead of one binary configured differently per environment at runtime, undermining confidence that what was tested elsewhere matches what actually runs in production.

## Common Interview Questions

### Basic
- How does ASP.NET Core determine which environment-specific configuration file to load?
- What kinds of settings belong in source-controlled config files versus environment-injected values?

### Intermediate
- Why is deploying the same binary, configured differently per environment, preferable to building separate binaries per environment?
- How would you catch a missing environment-specific configuration value before it causes a production incident?

### Advanced
- How would you design a configuration-validation strategy that fails fast at startup for every required, environment-specific setting?
- How would you keep environment-specific configuration files from drifting out of sync across environments over time?

### Follow-up Questions
- Should staging environment configuration exactly mirror production's real secrets?
- Does `ASPNETCORE_ENVIRONMENT` affect anything beyond which `appsettings.*.json` file is loaded?

### Code Prediction
A required `ExternalApi:ApiKey` setting is present in `appsettings.Development.json` but was never added to production's environment variables. With no startup validation configured, when does this missing configuration actually surface as a problem — at deployment, at application startup, or only when a specific code path calls the external API for the first time?

## Practical Tasks

- Configure environment-specific settings using `appsettings.{Environment}.json` and verify the correct one loads based on `ASPNETCORE_ENVIRONMENT`.
- Add startup validation (`ValidateOnStart()`) for a required, environment-specific configuration value, and verify it fails fast when missing.
- Design a process for keeping environment-specific configuration files in sync as new settings are added over time.

## Readiness Criteria

Manage environment-specific configuration using the same binary configured differently per environment, keep sensitive values out of source control, and validate required configuration at startup rather than discovering gaps reactively.

## References

### Microsoft Learn

- [Use multiple environments in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/environments)
- [Configuration in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/configuration/)
