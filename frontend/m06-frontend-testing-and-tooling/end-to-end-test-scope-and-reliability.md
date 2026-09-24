# End-to-End Test Scope and Reliability

## Definition

E2E test scope is the deliberate selection of a few journeys whose failure would be expensive or invisible to lower-level tests. Reliability means those journeys use isolated state, known dependencies, deterministic configuration, and a clear failure diagnosis path.

## Alternatives & Trade-offs

An E2E suite that uses real third-party services can find integration surprises but becomes slow and unstable. A fully mocked suite is reliable but can miss deployment and contract issues. Use controlled test environments by default and reserve scheduled or pre-release smoke tests for real external integration where the risk warrants it.

## How It Works

Give each worker its own account, tenant, records, or namespaced data. Seed only what the test needs, clean it deterministically, and avoid coupling tests through execution order. Define authentication setup deliberately: a reusable storage state can speed independent tests, while a dedicated sign-in journey still verifies the authentication flow.

## Application

Select journeys around money, access, irreversible writes, and primary business workflows. Keep detailed field validation, alternate error formats, and component layout branches in faster tests. When an E2E failure occurs, classify it as product, test, environment, test-data, or external-dependency failure before changing timeouts.

## Common Mistakes

- Using one shared administrator account in all parallel tests.
- Making every test sign in through the UI regardless of what it intends to verify.
- Calling a test "E2E" while stubbing every meaningful system boundary.
- Deleting a flaky test instead of identifying the uncontrolled dependency.

## Common Interview Questions

### Foundation

- Which risks should E2E tests own?
- Why is test data isolation important?

### Intermediate

- How would you test authenticated pages efficiently without removing sign-in coverage?
- How should a team triage a flaky E2E failure?

### Advanced and Follow-up

- What would you run per pull request versus nightly or before release?

### Code Prediction

Why can two passing E2E tests fail when executed together if they both edit the same seeded record?

## Practical Tasks

- Classify a feature's test cases into component versus E2E coverage.
- Make two parallel browser tests independent by namespacing their accounts or created data.

## Readiness Criteria

You can limit E2E tests to high-value journeys, isolate their state, and investigate intermittent failures without treating retries as a cure.

## References

- [Playwright parallelism](https://playwright.dev/docs/test-parallel)
- [Playwright authentication](https://playwright.dev/docs/auth)
