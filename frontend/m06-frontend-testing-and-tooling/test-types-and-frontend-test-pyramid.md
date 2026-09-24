# Test Types and the Frontend Test Pyramid

## Definition

A frontend test strategy distributes confidence across test layers. Unit tests check focused logic, component tests verify a rendered component and its interactions, integration tests exercise multiple frontend boundaries together, and end-to-end (E2E) tests exercise a critical journey in a real browser. The useful "pyramid" is not a fixed ratio: it means most feedback should be fast and local, while a smaller number of broader tests protect high-risk journeys.

## Alternatives & Trade-offs

Testing every behavior only through E2E tests gives high realism but slow, costly feedback and difficult failure diagnosis. Testing only isolated functions is fast but cannot establish that accessible controls, router state, providers, and API outcomes work together. Choose the lowest layer that credibly proves the risk in question.

## How It Works

| Layer | Main question | Typical example |
| --- | --- | --- |
| Unit | Does focused logic transform data correctly? | Filter or formatter |
| Component | Can a user observe and operate this component? | Form validation and submit state |
| Integration | Do feature boundaries cooperate? | Routed page plus API outcome |
| E2E | Does a critical browser journey work? | Sign in and create an order |

Code coverage reports executed lines, not missing user journeys or meaningful assertions. Use it as a signal for unexplored code, not a target that makes low-value tests valuable.

## Application

Start by listing user risks: a payment action must not submit twice, a failed save must be visible, and an authorized user must reach a feature. Put pure transformation rules at unit level, user-visible component behavior at component level, boundary coordination at integration level, and a few revenue, access, or data-loss journeys at E2E level.

## Common Mistakes

- Treating a fixed percentage split or coverage threshold as a test strategy.
- Writing all tests through a browser when a component test would isolate the failure faster.
- Calling a shallow render a component test even though providers and interactions are absent.
- Creating a separate test for every implementation branch without identifying a user-visible risk.

## Common Interview Questions

### Foundation

- What distinguishes unit, component, integration, and E2E tests?
- Why is code coverage not proof of quality?

### Intermediate

- How would you decide whether a form validation rule belongs in a component or E2E test?
- Why should an application not rely exclusively on E2E tests?

### Advanced and Follow-up

- How would you change the strategy for a checkout flow compared with a decorative settings panel?
- Which failures are cheaper to diagnose at each layer?

### Code Prediction

If a test clicks a button, checks its accessible error message, and intercepts an API response without launching a browser, which layer is it closest to and why?

## Practical Tasks

- Take one feature and list its risks, then assign each to the smallest useful test layer.
- Review a coverage report and identify two untested behaviors that line coverage alone cannot reveal.

## Readiness Criteria

You can choose a test layer from the behavior and risk being protected, explain the confidence and maintenance trade-off, and avoid using coverage as the sole measure of test quality.

## References

- [Testing Library guiding principles](https://testing-library.com/docs/guiding-principles/)
- [Playwright best practices](https://playwright.dev/docs/best-practices)
