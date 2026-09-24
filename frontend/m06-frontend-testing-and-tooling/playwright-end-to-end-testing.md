# Playwright End-to-End Testing

## Definition

Playwright drives a real browser to test a complete user journey. It can exercise navigation, browser storage, rendering, focus, network behavior, and integration with a deployed or locally hosted application. Cypress is a viable alternative; this module uses Playwright examples because of its browser isolation, tracing, and multi-browser support.

## Alternatives & Trade-offs

E2E tests give the highest frontend realism but take longer to run and diagnose than component tests. They should protect a small number of critical cross-boundary journeys, not duplicate every component test. Prefer resilient locators and observable completion conditions over arbitrary waits.

## How It Works

Use role and label locators, perform a real workflow, and assert the user-visible result:

```ts
await page.getByLabel("Email").fill("person@example.test");
await page.getByRole("button", { name: "Sign in" }).click();
await expect(page.getByRole("heading", { name: "Dashboard" })).toBeVisible();
```

Playwright auto-waits for actionability and assertions. This is not permission to omit synchronization reasoning: the assertion must still describe the state that means the journey has completed.

## Application

Test a narrow critical path such as sign-in, a protected action, a purchase, or a significant create/edit flow. Use isolated accounts and data. Capture traces, screenshots, videos, and console/network output for failures so CI results are diagnosable.

## Common Mistakes

- Locating elements by unstable CSS or position rather than role and name.
- Testing every validation branch through E2E instead of component tests.
- Sharing an account or mutable record across parallel workers.
- Waiting for a timeout instead of a visible or network-idle condition justified by the product.

## Common Interview Questions

### Foundation

- What does E2E testing provide that `jsdom` cannot?
- Why should E2E locators prefer roles and labels?

### Intermediate

- Which journeys justify E2E coverage?
- What artifacts help investigate a CI-only failure?

### Advanced and Follow-up

- How would you make a sign-in journey safe to run in parallel?

### Code Prediction

Why is `page.waitForTimeout(1000)` less reliable than expecting the dashboard heading to become visible?

## Practical Tasks

- Write a Playwright test for a sign-in or create-resource journey using role locators.
- Configure traces and screenshots on failure, then inspect a deliberately failing run.

## Readiness Criteria

You can choose a journey appropriate for real-browser coverage, write resilient locators and assertions, and collect enough artifacts to diagnose a failure.

## References

- [Playwright introduction](https://playwright.dev/docs/intro)
- [Playwright locators](https://playwright.dev/docs/locators)
