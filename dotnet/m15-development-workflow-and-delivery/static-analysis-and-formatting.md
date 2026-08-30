# Static Analysis, Analyzers, and Formatting

## Definition

Static analysis examines code without running it, catching potential bugs, style violations, and anti-patterns automatically. .NET's **Roslyn analyzers** run as part of the normal build, surfacing issues (some as warnings, some as errors, configurable) directly in the editor and CI. Automated **formatting** (via `dotnet format` or an editor's built-in formatter) enforces consistent style mechanically, removing it as a topic for human code review entirely.

```bash
dotnet format                      # automatically reformats code to match configured style rules
dotnet build /p:TreatWarningsAsErrors=true  # promotes analyzer warnings to build failures
```

## Alternatives & Trade-offs

Relying on human code review to catch style inconsistencies and common bug patterns works but is slow, inconsistent (different reviewers notice different things), and — as covered in the code-review topic — wastes reviewer attention on things a machine could catch instantly and consistently every single time. Automated analyzers and formatters catch the same class of issues instantly and uniformly, at the cost of some setup and occasional friction when a rule doesn't fit a specific situation (usually addressable with a scoped suppression rather than disabling the rule entirely).

## How It Works

### Analyzer severity levels and where they surface

```xml
<PropertyGroup>
    <EnableNETAnalyzers>true</EnableNETAnalyzers>
    <AnalysisLevel>latest</AnalysisLevel>
</PropertyGroup>
```

```
CA2000: Dispose objects before losing scope         <- catches a real resource-leak risk (Module 5)
CA1062: Validate arguments of public methods          <- catches a missing guard clause (Module 4)
IDE0044: Add readonly modifier                          <- a style/best-practice suggestion
```

Analyzers range from "this is very likely a real bug" (like a disposal issue) to "this is a stylistic best practice" — configuring which ones are errors, warnings, or suggestions (and which are suppressed for good reason) is a real team decision, not a one-size-fits-all default.

### `EditorConfig` — the shared source of truth for formatting rules

```ini
# .editorconfig
[*.cs]
indent_size = 4
csharp_new_line_before_open_brace = all
dotnet_sort_system_directives_first = true
```

An `.editorconfig` file, checked into source control, means every contributor's editor (and `dotnet format`, and CI) applies the *same* formatting rules — removing "should this brace be on its own line" from human code review entirely, since the tooling enforces it mechanically and consistently.

### Enforcing formatting/analysis in CI, not just locally

```yaml
- run: dotnet format --verify-no-changes  # fails CI if code isn't already correctly formatted
- run: dotnet build /p:TreatWarningsAsErrors=true  # fails CI on any analyzer warning
```

Without a CI gate, formatting/analyzer rules are only as effective as individual developers remembering to run them locally — enforcing them in the pipeline (Module 11's tests-in-ci.md pattern, applied here to code quality rather than test results) makes them a guarantee, not a suggestion.

### Suppressing a rule deliberately, with justification, rather than disabling it broadly

```csharp
#pragma warning disable CA2000 // Dispose is intentionally deferred to the caller via IDisposable ownership transfer
var stream = new MemoryStream();
#pragma warning restore CA2000
```

A scoped, justified suppression for a specific genuine exception is very different from disabling a whole analyzer rule project-wide because it was noisy once.

## Application

Configure Roslyn analyzers and an `.editorconfig` for every project, enforce both in CI (not just locally), and treat genuinely bug-catching analyzer categories as errors. Reserve human code review attention for correctness and design (the code-review topic), letting tooling handle style and common bug patterns mechanically and consistently.

## Common Mistakes

- Relying on human reviewers to catch style inconsistencies that an `.editorconfig` and automated formatter would catch instantly and consistently.
- Not enforcing formatting/analyzer checks in CI, leaving them as easily-forgotten local-only conveniences rather than an actual guarantee.
- Disabling an entire analyzer rule project-wide to silence one noisy false positive, instead of using a scoped, justified suppression for that specific case.
- Treating every analyzer warning as equally important, rather than distinguishing genuine bug-risk categories (disposal, null-argument validation) from purely stylistic suggestions when deciding what should block a build.

## Common Interview Questions

### Basic
- What do Roslyn analyzers catch, and how does that differ from what a formatter does?
- What is an `.editorconfig` file, and why is it checked into source control?

### Intermediate
- Why should formatting/style concerns generally not be a topic in human code review?
- What's the difference between disabling an analyzer rule broadly versus suppressing it for one scoped, justified case?

### Advanced
- How would you decide which analyzer categories should fail the build (as errors) versus just warn?
- How would you enforce formatting and analysis consistently across a team, including in CI, not just relying on individual developer discipline?

### Follow-up Questions
- Does `dotnet format` catch the same class of issues as Roslyn analyzers?
- Should analyzer rules ever be enforced more strictly in CI than they are configured locally?

### Code Prediction
A team has an `.editorconfig` configured correctly, but no CI step running `dotnet format --verify-no-changes` or checking for analyzer warnings. A developer's editor isn't configured to respect `.editorconfig`. What's the likely outcome for that developer's contributions, compared to a team that enforces both in CI?

## Practical Tasks

- Configure an `.editorconfig` for a project and verify `dotnet format` enforces it correctly.
- Configure a CI pipeline step that fails the build on any analyzer warning or unformatted code.
- Identify a case warranting a scoped, justified analyzer suppression versus one that should actually be fixed.

## Readiness Criteria

Configure and enforce analyzers and formatting consistently including in CI, distinguish genuine bug-risk analyzer categories from stylistic suggestions, and use scoped suppressions rather than broad rule disabling.

## References

### Microsoft Learn

- [Overview of .NET source code analysis](https://learn.microsoft.com/dotnet/fundamentals/code-analysis/overview)
- [dotnet format command](https://learn.microsoft.com/dotnet/core/tools/dotnet-format)
- [EditorConfig for code formatting](https://learn.microsoft.com/dotnet/fundamentals/code-analysis/code-style-rule-options)
