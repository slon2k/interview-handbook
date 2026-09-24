# Test Doubles and Avoiding Implementation Details

## Definition

A test double replaces a collaborator with a controlled substitute. A stub returns known data, a spy records interactions, a mock verifies an expected collaboration, and a fake is a simplified working implementation. Good frontend tests use doubles at an intentional boundary and assert observable behavior rather than private component structure.

## Alternatives & Trade-offs

Directly testing internal state, private helpers, call order, or a third-party component's markup gives quick feedback but makes tests fail during harmless refactors. Behavior tests are more resilient, but they may need a narrower, explicit double when an external side effect would otherwise make the test slow or unsafe.

## How It Works

Choose the closest stable boundary. Use MSW for HTTP. Use an injected callback or adapter for browser APIs that cannot run in the test environment. Spy on a navigation or telemetry adapter only when its invocation is itself the observable contract; otherwise assert the resulting route or UI.

## Application

Prefer real local code and controlled external boundaries. A formatter normally needs no mock. A payment gateway, analytics transport, clock, or browser clipboard may need an adapter or fake. Keep the double's interface small so production changes are not mirrored as test maintenance.

## Common Mistakes

- Mocking every import by default.
- Asserting a component called `setState` or an internal helper a certain number of times.
- Writing a fake whose behavior contradicts the real contract.
- Using a spy assertion where a user-visible route or message could be asserted instead.

## Common Interview Questions

### Foundation

- What is the difference between a stub, spy, mock, and fake?
- Why are implementation-detail tests fragile?

### Intermediate

- When would you mock an API client instead of use MSW?
- How would you test clipboard failure without relying on the real clipboard?

### Advanced and Follow-up

- How can excessive mocks hide integration defects?

### Code Prediction

Why can a test that asserts a private helper call fail after a valid component refactor with no changed user behavior?

## Practical Tasks

- Replace one internal-call assertion with an assertion of visible behavior.
- Design a small adapter for a browser API and use a fake to test its failure branch.

## Readiness Criteria

You can name the role of a test double, place it at a stable external boundary, and avoid assertions coupled to incidental implementation.

## References

- [Testing Library guiding principles](https://testing-library.com/docs/guiding-principles/)
- [Martin Fowler: Mocks Aren't Stubs](https://martinfowler.com/articles/mocksArentStubs.html)
