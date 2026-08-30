# Module 15 - Development Workflow and Delivery Fundamentals

**Status:** Complete  
**Priority:** High for practical readiness, although usually lighter in theoretical interviews.  
**Prerequisites:** [Testing and Testability](../m11-testing-and-testability/README.md), [Entity Framework Core](../m10-entity-framework-core/README.md)

## Scope

This module covers the normal daily workflow expected from a professional developer — Git, code review, debugging, dependency management, build/publish, containerization, CI/CD, and release practices. Azure-specific delivery details are deliberately out of scope here and would belong in a later, dedicated cloud-platform module if this handbook adds one. Several topics build directly on earlier modules — CI running tests (Module 11), migration mechanics (Module 10), configuration layering (Module 8), and metrics/observability for release monitoring (Module 13) — applying them to the delivery-pipeline context rather than re-teaching the mechanics.

## Learning Outcomes

By the end of this module, you should be able to:

- Reason about Git as a commit graph, use short-lived branches and well-structured PRs, and resolve merge conflicts by understanding intent rather than mechanically picking a side.
- Write commits and give code review feedback that's genuinely useful for future debugging and collaboration.
- Debug efficiently using conditional breakpoints and the call stack, and manage dependencies with an eye toward versioning risk and vulnerabilities.
- Distinguish build from publish, configure environments correctly, and enforce static analysis and formatting automatically rather than through human review.
- Write efficient multi-stage Dockerfiles and design CI/CD pipelines as fast, well-ordered gates.
- Sequence database migrations safely for rolling deployments, and use feature flags to enable trunk-based development and fast rollback.
- Choose deployment shapes proportional to risk and have a concrete rollback plan before every release.

## Topics

### 1. Git and Collaboration

- [Git fundamentals](git-fundamentals.md)
- [Branching and pull requests](branching-and-pull-requests.md)
- [Commit quality](commit-quality.md)
- [Merge conflicts](merge-conflicts.md)
- [Code review](code-review.md)

### 2. Local Development Tooling

- [Debugging in Visual Studio/Rider, and breakpoints](debugging-and-breakpoints.md)
- [NuGet dependency management](nuget-dependency-management.md)

### 3. Build and Configuration

- [Build and publish commands, and project configuration](build-publish-and-project-configuration.md)
- [Environment-specific settings](environment-specific-settings.md)
- [Static analysis, analyzers, and formatting](static-analysis-and-formatting.md)

### 4. Containerization and Pipelines

- [Docker fundamentals](docker-fundamentals.md)
- [CI/CD concepts, and running tests in a pipeline](ci-cd-concepts-and-pipeline-testing.md)

### 5. Release Practices

- [Database migrations during deployment](database-migrations-during-deployment.md)
- [Feature flags](feature-flags.md)
- [Basic release and rollback thinking](release-and-rollback-thinking.md)

## Scope Boundaries

- Running tests as an enforced CI gate is covered in depth in [Module 11 - Testing and Testability](../m11-testing-and-testability/tests-in-ci.md); this module's CI/CD topic covers the broader pipeline concept, including what happens after tests pass.
- EF Core migration mechanics (generating, reviewing, writing data migrations) belong in [Module 10 - Entity Framework Core](../m10-entity-framework-core/migrations.md); this module covers sequencing migrations safely within a deployment.
- Configuration layering mechanics belong in [Module 8 - ASP.NET Core Fundamentals](../m08-aspnet-core-fundamentals/configuration-and-options-pattern.md); this module covers the environment-management practice built on top of it.
- Metrics, tracing, and resilience patterns belong in [Module 13 - Performance, Diagnostics, and Observability](../m13-performance-diagnostics-observability/README.md); this module's release-thinking topic assumes that monitoring exists and focuses on release/rollback decisions built on top of it.
- Azure-specific or other cloud-provider-specific delivery tooling is out of scope entirely.

## Suggested Learning Sequence

1. Git fundamentals, branching/PRs, commit quality, merge conflicts, code review.
2. Debugging and NuGet dependency management.
3. Build/publish, project configuration, environment-specific settings, static analysis/formatting.
4. Docker fundamentals and CI/CD pipeline concepts.
5. Database migrations during deployment, feature flags, and release/rollback thinking.

## Practical Deliverables

- Practice a full Git workflow: branch, small focused commits, PR, conflict resolution, and a chosen merge strategy.
- Give and receive code review feedback on a sample PR, focused on correctness and design.
- Debug a sample bug efficiently using a conditional breakpoint and the call stack.
- Write a multi-stage Dockerfile and a CI/CD pipeline definition with parallelized, well-ordered stages.
- Design an expand/contract migration sequence for a rolling deployment, and a feature-flag-based rollout plan for a risky change.
- Write an explicit rollback plan for a hypothetical deployment before "deploying" it.

## Interview Coverage

Each topic should include:

- Basic questions for terminology and tool familiarity.
- Intermediate questions involving common workflow decisions and trade-offs.
- Advanced questions involving pipeline/deployment design and incident-response reasoning.
- Follow-up questions that test practical judgment rather than memorized command lists.
- Code-prediction questions grounded in concrete workflow scenarios, since this module is explicitly about daily practical readiness rather than pure theory.

## References

### Microsoft Learn

- [.NET application publishing overview](https://learn.microsoft.com/dotnet/core/deploying/)
- [Containerize a .NET app](https://learn.microsoft.com/dotnet/core/docker/build-container)

### Other

- [Pro Git book](https://git-scm.com/book/en/v2)
- [Martin Fowler's writing on delivery practices (bliki)](https://martinfowler.com/bliki/)
