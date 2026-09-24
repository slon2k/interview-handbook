# Module 6 - Frontend Testing and Tooling

**Status:** Complete  
**Priority:** High  
**Prerequisites:** [Module 3 - TypeScript](../m03-typescript/README.md) and [Module 5 - React](../m05-react/README.md)

## Scope

This module covers the practices and tools that turn a React feature into a reliably tested, maintainable, and deliverable frontend application. It uses Vitest, React Testing Library, Mock Service Worker (MSW), and Playwright as the primary examples while explaining where Jest and Cypress are viable alternatives.

The focus is on testing behavior a user can observe, choosing the narrowest useful test layer, and using build and delivery tooling deliberately. Tool APIs change; the ability to identify a valuable assertion, create a credible API boundary, investigate a flaky failure, or diagnose a slow page is the durable interview skill.

## Why This Matters in Interviews

Interviewers look beyond whether a candidate can write a passing assertion. They commonly ask how a React component should be tested, why an effect mock is brittle, how API failures should be represented, why a test is flaky in CI, or how a production bundle became expensive. Strong answers connect a test to user-visible behavior and explain the trade-off between confidence, speed, and maintenance cost.

## Learning Outcomes

By the end of this module, you should be able to:

- Choose an appropriate balance of unit, component, integration, and end-to-end tests.
- Configure and use Vitest, React Testing Library, MSW, and Playwright for a typical React application.
- Test accessible user interactions and asynchronous loading, empty, error, and success states without relying on implementation details.
- Create request-level API test scenarios and keep tests deterministic under local and CI execution.
- Explain the practical role and limits of snapshots, visual checks, and cross-browser testing.
- Reason about packages, Vite builds, linting, formatting, code splitting, bundle size, browser performance, caching, and frontend CI.

## Topics

### 1. Test Strategy and Component Tests

- [Test types and the frontend test pyramid](test-types-and-frontend-test-pyramid.md)
- [Vitest test environment and configuration](vitest-test-environment-and-configuration.md)
- [React Testing Library queries and user events](react-testing-library-queries-and-user-events.md)
- [Testing asynchronous UI states and data fetching](testing-async-ui-states-and-data-fetching.md)

### 2. API Integration and Test Reliability

- [Mock Service Worker and API mocking](mock-service-worker-and-api-mocking.md)
- [Component integration tests and test boundaries](component-integration-tests-and-test-boundaries.md)
- [Test doubles and avoiding implementation details](test-doubles-and-avoiding-implementation-details.md)
- [Test reliability, flakiness, and determinism](test-reliability-flakiness-and-determinism.md)
- [Snapshot testing trade-offs](snapshot-testing-trade-offs.md)

### 3. End-to-End and Browser Confidence

- [Playwright end-to-end testing](playwright-end-to-end-testing.md)
- [End-to-end test scope and reliability](end-to-end-test-scope-and-reliability.md)
- [Visual regression and cross-browser awareness](visual-regression-and-cross-browser-awareness.md)

### 4. Tooling, Performance, and Delivery

- [npm, `package.json`, lockfiles, and dependencies](npm-package-json-lockfiles-and-dependencies.md)
- [Vite development, production builds, and environment variables](vite-development-production-builds-and-environment-variables.md)
- [TypeScript, ESLint, Prettier, and automated quality checks](typescript-eslint-prettier-and-automated-quality-checks.md)
- [Bundling, tree shaking, lazy loading, and code splitting](bundling-tree-shaking-lazy-loading-and-code-splitting.md)
- [Bundle analysis and asset optimization](bundle-analysis-and-asset-optimization.md)
- [Core Web Vitals, network waterfalls, and render profiling](core-web-vitals-network-waterfalls-and-render-profiling.md)
- [Browser caching and frontend delivery](browser-caching-and-frontend-delivery.md)
- [Frontend CI and quality gates](frontend-ci-and-quality-gates.md)

## Scope Boundaries

- React component behavior, hooks, rendering, and routing belong in [Module 5](../m05-react/README.md); this module tests those behaviors.
- Browser requests, CORS, cookies, caching mechanics, and browser developer tools belong in [Module 4](../m04-browser-platform-and-aspnet-core-api-integration/README.md); this module applies them to test and delivery decisions.
- Semantic HTML and baseline accessibility behavior belong in [Module 1](../m01-web-platform-foundations/README.md); this module uses those semantics as reliable test interfaces.
- TypeScript modeling and compiler semantics belong in [Module 3](../m03-typescript/README.md).
- Backend test implementation, observability, and deployment infrastructure remain in the related .NET modules.

## Suggested Learning Sequence

1. Define which test layer owns which risk before learning framework APIs.
2. Configure the test environment and write behavior-focused React Testing Library tests.
3. Add asynchronous state and API-boundary scenarios.
4. Add request-level API scenarios, test reliability practices, and a few high-value Playwright journeys.
5. Finish with package management, builds, quality checks, performance diagnosis, cache behavior, and CI quality gates.

## Practical Deliverables

- Write a component test that uses accessible queries to verify a realistic user interaction.
- Test loading, empty, error, and success states without arbitrary delays.
- Explain why a test belongs at its chosen layer and which behavior it intentionally leaves to another layer.
- Test a Module 5 searchable-catalog feature through component, request, and browser layers, then analyze its production bundle and CI checks.

## Interview Coverage

Each topic includes foundation, intermediate, and advanced questions plus a debugging or design prompt. Practice explaining the observable behavior a test protects, the test's boundary, and the trade-off behind its tools.

## References

- [Vitest documentation](https://vitest.dev/guide/)
- [React Testing Library documentation](https://testing-library.com/docs/react-testing-library/intro/)
- [Testing Library guiding principles](https://testing-library.com/docs/guiding-principles/)
- [Mock Service Worker documentation](https://mswjs.io/docs/)
- [Playwright documentation](https://playwright.dev/docs/intro)
- [Vite documentation](https://vite.dev/guide/)
