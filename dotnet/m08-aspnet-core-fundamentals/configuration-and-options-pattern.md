# Configuration and the Options Pattern

## Definition

ASP.NET Core configuration is layered: `appsettings.json`, environment-specific overrides (`appsettings.Production.json`), environment variables, command-line arguments, and secret stores all merge into one `IConfiguration`, with later sources overriding earlier ones. The **options pattern** binds a section of that configuration into a strongly-typed class, injected wherever it's needed, instead of scattering raw string-keyed lookups through the codebase.

```json
// appsettings.json
{ "Email": { "SmtpHost": "smtp.example.com", "Port": 587 } }
```

```csharp
public class EmailOptions
{
    public string SmtpHost { get; set; } = "";
    public int Port { get; set; }
}

builder.Services.Configure<EmailOptions>(builder.Configuration.GetSection("Email"));
```

## Alternatives & Trade-offs

Raw `IConfiguration["Email:SmtpHost"]` lookups are quick for a one-off value but scatter magic strings throughout the codebase with no compile-time checking and no single place documenting what configuration the app actually needs. The options pattern centralizes each configuration section into a typed class, catching typos and missing values earlier, at the cost of a small amount of setup per section.

## How It Works

### Configuration layering and precedence

```
appsettings.json                    <- base
appsettings.{Environment}.json      <- overrides base for the current environment
Environment variables               <- override both files (useful for secrets/deployment-specific values)
Command-line arguments              <- override everything else
```

```bash
# Overrides Email:SmtpHost from any environment, without touching a file
export Email__SmtpHost="smtp.override.example.com"  # double underscore maps to the nested "Email:SmtpHost" key
```

### Injecting options

```csharp
public class EmailService
{
    private readonly EmailOptions _options;
    public EmailService(IOptions<EmailOptions> options) => _options = options.Value; // resolved once, cached
}
```

### `IOptions<T>` vs. `IOptionsSnapshot<T>` vs. `IOptionsMonitor<T>`

```csharp
IOptions<T>         // resolved once, same value for the app's lifetime — fine for singletons and static config
IOptionsSnapshot<T> // re-read per scope (per request) — use when config can change and a scoped service needs the latest value
IOptionsMonitor<T>  // supports live reload notifications via OnChange — use in singletons that need to react to config changes
```

```csharp
public class NotificationService
{
    public NotificationService(IOptionsMonitor<EmailOptions> monitor)
    {
        monitor.OnChange(updated => Console.WriteLine($"Email config changed: {updated.SmtpHost}"));
    }
}
```

### Validating options at startup

```csharp
builder.Services.AddOptions<EmailOptions>()
    .Bind(builder.Configuration.GetSection("Email"))
    .Validate(o => !string.IsNullOrEmpty(o.SmtpHost), "SmtpHost is required")
    .ValidateOnStart(); // fails fast at startup instead of the first time EmailOptions is used
```

## Application

Use the options pattern for any configuration section consumed by application code, rather than scattering `IConfiguration["..."]` lookups. Use `IOptionsSnapshot<T>` for scoped services that should see updated configuration per request, and `IOptionsMonitor<T>` for singletons that need to react to configuration changes without restarting. Validate required configuration at startup with `ValidateOnStart()` so misconfiguration fails immediately rather than at first use in production.

## Common Mistakes

- Scattering raw `IConfiguration["Section:Key"]` string lookups throughout the codebase instead of binding to a typed options class.
- Injecting `IOptions<T>` into a scoped or transient service that actually needs live-reloaded configuration, missing updates that `IOptionsSnapshot<T>`/`IOptionsMonitor<T>` would have provided.
- Not validating required configuration at startup, letting a missing or malformed value surface only when the relevant code path first runs in production.
- Storing secrets directly in `appsettings.json` instead of environment variables, a secret manager, or a dedicated secret store.

## Common Interview Questions

### Basic
- What is the options pattern, and what problem does it solve compared to raw `IConfiguration` lookups?
- What is the precedence order of ASP.NET Core's configuration sources?

### Intermediate
- What's the difference between `IOptions<T>`, `IOptionsSnapshot<T>`, and `IOptionsMonitor<T>`?
- How do environment variables map to nested configuration keys?

### Advanced
- How would you validate required configuration at startup so the app fails fast instead of at first use?
- How would you design configuration for a singleton service that needs to react to configuration changes without an app restart?

### Follow-up Questions
- Can command-line arguments override values set in `appsettings.json`?
- Should secrets ever be stored in `appsettings.json`?

### Code Prediction
A singleton service injects `IOptions<EmailOptions>`, and the underlying configuration value changes at runtime (e.g., via a reloaded config file). Does the singleton see the updated value? What would need to change to make it see the update?

## Practical Tasks

- Bind a configuration section to a typed options class and inject it into a service.
- Add startup validation for a required configuration value and verify the app fails to start when it's missing.
- Implement a singleton service that reacts to configuration changes using `IOptionsMonitor<T>.OnChange`.

## Readiness Criteria

Explain configuration layering and precedence, choose the correct `IOptions` variant for a given service lifetime and reload requirement, and validate configuration at startup rather than at first use.

## References

### Microsoft Learn

- [Configuration in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/configuration/)
- [Options pattern in ASP.NET Core](https://learn.microsoft.com/aspnet/core/fundamentals/configuration/options)
