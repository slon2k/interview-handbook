# Secret Management

## Definition

Secret management is how an application stores and accesses sensitive configuration values — connection strings, API keys, signing keys — without ever committing them to source control or leaving them exposed in plain configuration files. .NET provides the Secret Manager tool for local development and integrates with dedicated secret stores (Azure Key Vault, AWS Secrets Manager, HashiCorp Vault) for production.

```bash
dotnet user-secrets set "ConnectionStrings:Default" "Server=...;Password=..."
```

```csharp
builder.Configuration.AddAzureKeyVault(keyVaultUri, new DefaultAzureCredential()); // production
```

## Alternatives & Trade-offs

Committing secrets directly into `appsettings.json` (or worse, into source control) is the simplest thing to do and the most dangerous — a leaked repository, a misconfigured public repo, or even just broad internal read access exposes every secret at once, permanently, since git history retains it even after deletion. Environment variables are a step up (not committed to source control) but still visible to anything with process/environment access. A dedicated secret store adds infrastructure but provides access auditing, rotation support, and centralized revocation that files and environment variables don't.

## How It Works

### Local development — Secret Manager, kept outside the project directory

```bash
dotnet user-secrets init
dotnet user-secrets set "ExternalApi:ApiKey" "sk_test_abc123"
```

Secret Manager stores values in a JSON file in the user's profile directory, entirely outside the project folder — so it's structurally impossible to accidentally commit it to source control the way a stray `appsettings.Development.json` edit could be.

### Why environment variables alone aren't sufficient for production secrets

```bash
export ConnectionStrings__Default="Server=...;Password=SuperSecret123"
```

Environment variables avoid source control exposure, but are visible to anything with process-inspection access on the host, aren't rotated automatically, and provide no audit trail of who accessed which secret and when — all things a dedicated secret store provides.

### Azure Key Vault (or an equivalent) in production

```csharp
builder.Configuration.AddAzureKeyVault(
    new Uri("https://my-vault.vault.azure.net/"),
    new DefaultAzureCredential()); // uses managed identity — no secret needed just to access the secret store itself
```

Using a managed identity to authenticate *to* the secret store (rather than a connection string or key that itself would need to be secured) avoids the classic "but how do you securely store the secret needed to access your secrets" circularity.

### What NEVER belongs in source control, regardless of how it's protected elsewhere

```
Connection strings with credentials, API keys, signing keys, certificates, OAuth client secrets —
none of these belong in appsettings.json committed to git, even in a private repository.
```

Even a private repository can be exposed by a misconfigured permission, an accidental push to a public mirror, or simply anyone with legitimate repo access who shouldn't have production credentials.

## Application

Use the Secret Manager tool for local development secrets. Use a dedicated secret store (Key Vault or equivalent) in production, authenticated via managed identity where the hosting platform supports it, rather than yet another secret needed just to access the secret store. Never commit real secrets to source control at any point, even temporarily.

## Common Mistakes

- Committing a real connection string or API key to `appsettings.json` in source control, even briefly — git history retains it indefinitely even after the file is later "fixed."
- Relying solely on environment variables for production secrets without any rotation, auditing, or centralized revocation capability.
- Using a connection string or static key to authenticate to the secret store itself, recreating the same "secret to protect the secret" problem a managed identity would avoid.
- Not having a plan for rotating a secret after a suspected leak, discovering only in the moment of an incident that rotation isn't actually straightforward.

## Common Interview Questions

### Basic
- Why shouldn't secrets be committed to source control, even in a private repository?
- What is the .NET Secret Manager tool used for?

### Intermediate
- What does a dedicated secret store (like Azure Key Vault) provide that environment variables don't?
- Why is using a managed identity to access a secret store preferable to using a static credential?

### Advanced
- How would you design a secret-rotation strategy that doesn't require redeploying the application every time a secret changes?
- How would you audit an existing codebase for accidentally committed secrets, and what would you do if you found one?

### Follow-up Questions
- Does deleting a committed secret from the latest commit remove it from the repository's history?
- Should Secret Manager values ever be used in a production deployment?

### Code Prediction
A connection string containing a database password was committed to a public GitHub repository, then removed in a later commit. Is the password still retrievable by anyone who clones the repository? What's the only real remediation?

## Practical Tasks

- Set up local development secrets using the Secret Manager tool, and verify they're stored outside the project directory.
- Configure an application to load production secrets from Azure Key Vault using managed identity authentication.
- Audit a small codebase for any committed secrets and design a remediation and rotation plan for one found.

## Readiness Criteria

Explain why source control and environment variables alone are insufficient for production secrets, configure a dedicated secret store correctly, and design a rotation strategy for a suspected leak.

## References

### Microsoft Learn

- [Safe storage of app secrets in development](https://learn.microsoft.com/aspnet/core/security/app-secrets)
- [Azure Key Vault configuration provider](https://learn.microsoft.com/aspnet/core/security/key-vault-configuration)

### Other

- [OWASP: Secrets Management Cheat Sheet](https://cheatsheetseries.owasp.org/cheatsheets/Secrets_Management_Cheat_Sheet.html)
