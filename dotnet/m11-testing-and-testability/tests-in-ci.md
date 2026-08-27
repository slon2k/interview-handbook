# Tests in CI

## Definition

Running the test suite automatically in a CI (continuous integration) pipeline — on every push or pull request — turns tests from something a developer might remember to run locally into an enforced gate that catches regressions before they merge, and produces a shared, visible record of test health over time.

```yaml
# Example GitHub Actions step
- name: Run tests
  run: dotnet test --logger "trx" --collect:"XPlat Code Coverage"
```

## Alternatives & Trade-offs

Running tests only locally, at a developer's discretion, is faster to iterate on individually but provides no guarantee that every change is actually verified before merging — it's easy to forget, skip under time pressure, or simply not run the full suite. Running tests in CI on every change is slower per-change (a pipeline run takes time) but provides a consistent, enforced safety net and a durable history of what passed and when.

## How It Works

### Fast feedback — splitting the suite by speed

```yaml
# Run fast unit tests first, fail fast; only run slower integration tests if those pass
- run: dotnet test --filter Category=Unit
- run: dotnet test --filter Category=Integration
```

Categorizing tests (e.g., via a `[Trait]`/`[Category]` attribute) and running fast tests first lets CI report a failure quickly for the common case, rather than always waiting for the full, slower suite before any feedback is available.

### Parallelizing test execution

```bash
dotnet test --parallel # or configuring test collections to run in parallel where safe
```

Test suites that follow this module's isolation guidance (no shared mutable state, no order dependency) can safely run in parallel, cutting total CI time significantly for a large suite — tests that violate isolation can fail unpredictably or produce wrong results when run in parallel, which is another concrete reason isolation matters beyond just local reliability.

### Failing the build on test failure — the actual enforcement mechanism

```yaml
- name: Run tests
  run: dotnet test # a non-zero exit code here fails the CI job, blocking merge if configured as a required check
```

The gate only works if CI is actually configured to block merging on failure (a "required status check" in most CI/PR tooling) — running tests in CI without enforcing the result is only marginally better than not running them at all.

### Testcontainers/Docker-dependent tests in CI

```yaml
services:
  docker:
    image: docker:dind # most CI providers need Docker-in-Docker or a similar setup for Testcontainers-based tests to run
```

Database integration tests using Testcontainers (from earlier in this module) need the CI environment to actually support running Docker containers — a real infrastructure consideration, not just a test-code concern.

### Reporting and visibility

```
CI dashboard: 1,204 tests passed, 3 failed, coverage 78%, run time 4m 12s
```

A CI-integrated test report gives the whole team visibility into test health trends over time, not just pass/fail for the current change — flagging, for example, a test suite that's gradually getting slower or a specific test that fails intermittently across many runs (flaky test detection).

## Application

Run the full test suite in CI on every pull request, configured as a required check that blocks merging on failure. Split and parallelize the suite by speed/category to keep feedback fast. Ensure the CI environment supports any infrastructure dependencies (Docker for Testcontainers) that slower integration tests require.

## Common Mistakes

- Running tests in CI without configuring them as a required, merge-blocking check, making the CI run informational rather than an actual enforced gate.
- Running the entire test suite (including slow integration tests) serially on every change without splitting or parallelizing, making feedback slow enough that developers start ignoring or working around it.
- Not accounting for Docker/Testcontainers infrastructure requirements in the CI environment, causing an entire category of tests to fail in CI even though they pass locally.
- Ignoring intermittently-failing ("flaky") tests instead of fixing or explicitly quarantining them, eroding trust in CI failures generally (a real failure gets dismissed as "probably just flaky" without investigation).

## Common Interview Questions

### Basic
- Why is running tests in CI more valuable than relying on developers to run them locally?
- What makes a CI test run an actual enforced gate rather than just informational?

### Intermediate
- How would you structure a CI pipeline to give fast feedback on common failures without always waiting for the full slow test suite?
- What infrastructure consideration does Testcontainers-based testing introduce for CI specifically?

### Advanced
- How would you design a strategy for detecting and addressing flaky tests systematically, rather than dismissing intermittent failures ad hoc?
- How would you balance parallelizing tests for speed against the risk of previously-hidden isolation bugs surfacing under parallel execution?

### Follow-up Questions
- Should code coverage thresholds be enforced as a hard CI gate?
- Does running tests in CI eliminate the value of running them locally during development?

### Code Prediction
A test suite passes reliably when run serially locally, but starts failing intermittently only in CI once test parallelization is enabled. What does this most likely reveal about the test suite's isolation properties, referencing the reliability discussion from earlier in this module?

## Practical Tasks

- Configure a CI pipeline step running the test suite and set it as a required, merge-blocking check.
- Split a test suite by category (unit vs. integration) and configure CI to run the fast category first for quicker feedback.
- Diagnose a test that passes serially but fails under parallel execution, and fix its isolation issue.

## Readiness Criteria

Configure CI to run tests as an actual enforced gate, structure the pipeline for fast feedback and appropriate infrastructure support, and diagnose isolation issues that only surface under CI-specific conditions like parallel execution.

## References

### Microsoft Learn

- [.NET test SDK and CI](https://learn.microsoft.com/dotnet/core/testing/unit-testing-with-dotnet-test)
