# Vitest Test Environment and Configuration

## Definition

Vitest is a Vite-native test runner. A React application usually needs a browser-like environment such as `jsdom`, setup code for DOM matchers and cleanup, TypeScript-aware configuration, and predictable scripts for local and CI execution.

## Alternatives & Trade-offs

Vitest shares Vite's transform pipeline and is a natural default for Vite applications. Jest remains common and has a large ecosystem, but its configuration can require separate transforms and environment setup. Tool choice matters less than a reproducible, fast configuration that runs the same assertions locally and in CI.

## How It Works

Keep test configuration near the build configuration and make its assumptions explicit:

```ts
// vite.config.ts
import { defineConfig } from "vitest/config";

export default defineConfig({
  test: {
    environment: "jsdom",
    setupFiles: ["./src/test/setup.ts"],
    globals: false,
  },
});
```

The setup file commonly imports `@testing-library/jest-dom/vitest` and registers `afterEach(cleanup)` when the project does not enable automatic cleanup. The test environment imitates browser APIs; it is not a replacement for real-browser E2E tests.

## Application

Provide distinct scripts for one run, watch mode, and coverage when coverage is useful. Pin behavior through the lockfile, run the same command in CI, and centralize shared providers or test helpers instead of copying setup into every test.

## Common Mistakes

- Assuming `jsdom` behaves exactly like Chromium, especially for layout, focus, and browser-only APIs.
- Hiding production configuration in a test-only alias or transform.
- Adding global helpers without documenting them, making test dependencies invisible.
- Letting CI install a different dependency graph from local development.

## Common Interview Questions

### Foundation

- What does a test environment such as `jsdom` provide?
- Why does a Vite application often choose Vitest?

### Intermediate

- Which behavior still needs a real browser test?
- What should a shared test setup file own?

### Advanced and Follow-up

- How would you diagnose a test that passes locally but fails in CI after a dependency update?

### Code Prediction

Would a passing `jsdom` test prove that an element is correctly laid out and visible after a CSS animation? Explain the environment boundary.

## Practical Tasks

- Configure a React project with Vitest, `jsdom`, and Testing Library DOM matchers.
- Add scripts for a single run and watch mode, then execute the single-run script from a clean install.

## Readiness Criteria

You can configure a minimal reproducible React test environment, explain the boundary between a simulated DOM and a real browser, and choose Jest only when its trade-offs suit the project.

## References

- [Vitest configuration](https://vitest.dev/config/)
- [Testing Library setup](https://testing-library.com/docs/react-testing-library/setup/)
