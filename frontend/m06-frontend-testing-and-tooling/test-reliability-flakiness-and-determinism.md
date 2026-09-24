# Test Reliability, Flakiness, and Determinism

## Definition

A deterministic test produces the same result from the same code and declared inputs. A flaky test sometimes passes and sometimes fails without a relevant code change, usually because it depends on uncontrolled time, network, shared state, ordering, animation, external services, or an overly broad selector.

## Alternatives & Trade-offs

Retries can reduce transient infrastructure noise, especially for E2E tests, but they should not be the primary fix for product-test flakiness. A retry turns a deterministic defect into a less visible intermittent signal. First identify whether the instability is in the test, application, environment, or external dependency.

## How It Works

Control time with fake timers only when the code's scheduling is the behavior under test. Control HTTP with MSW, reset state after every test, wait for meaningful visible conditions, and avoid reliance on test execution order. E2E tests should create or isolate their own data and retain traces, screenshots, and logs for failed attempts.

## Application

Treat a flaky test as a defect with an owner and diagnosis path: reproduce it, inspect artifacts, identify the non-deterministic input, replace it with a controlled condition, and keep the regression test only after it is stable. Use selectors based on user-facing semantics rather than timing or layout.

## Common Mistakes

- Fixing intermittent failures by raising timeouts or adding sleeps.
- Sharing database records, accounts, or browser state across parallel tests.
- Leaving a fake timer, mock, or request handler active for the next test.
- Using a retry as evidence that a failure is harmless.

## Common Interview Questions

### Foundation

- What makes a test flaky?
- Why is `setTimeout` usually a poor synchronization strategy in a test?

### Intermediate

- How would you diagnose a test that fails only in CI?
- When are retries acceptable?

### Advanced and Follow-up

- How would you make parallel E2E tests independent while keeping them fast?

### Code Prediction

Why can a test that asserts immediately after clicking a button pass locally and fail in CI when the button starts asynchronous work?

## Practical Tasks

- Repair a test that uses a fixed delay by waiting for its meaningful UI outcome.
- Make a test suite safe for random execution order by removing shared mutable state.

## Readiness Criteria

You can identify a test's uncontrolled dependency, use a deliberate synchronization condition, and distinguish a product defect from test or environment flakiness.

## References

- [Playwright: flaky tests](https://playwright.dev/docs/test-retries)
- [Testing Library async methods](https://testing-library.com/docs/dom-testing-library/api-async/)
