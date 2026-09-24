# Visual Regression and Cross-Browser Awareness

## Definition

Visual regression testing compares rendered screenshots against an approved baseline. Cross-browser testing runs selected behavior in more than one browser engine. Both add confidence for layout and compatibility risks, but neither replaces semantic, keyboard, interaction, or product review.

## Alternatives & Trade-offs

Screenshot comparison can catch unintended spacing, typography, and responsive changes that DOM assertions miss. It can also produce noise from font rendering, animation, dynamic data, and platform differences. Cross-browser suites find engine differences but multiply runtime. Apply both where the interface or audience makes their cost worthwhile.

## How It Works

Stabilize viewport, fonts, data, locale, time, animations, and network state before comparing an image. Maintain small, purposeful visual cases. Run the primary browser on each change and expand to supported browsers for critical interactions, targeted fixes, or scheduled confidence runs.

## Application

Use visual checks for design-system components, key responsive layouts, or regressions that previously escaped review. Use cross-browser coverage for supported browsers and features likely to vary, such as form controls, CSS layout, file input, clipboard, or payment redirects. Verify keyboard and assistive-technology behavior separately.

## Common Mistakes

- Approving screenshot updates without examining their diff.
- Comparing pages with live timestamps, random data, animations, or unpinned fonts.
- Treating pixel equality as accessibility testing.
- Running every browser for every low-risk assertion without a proportional reason.

## Common Interview Questions

### Foundation

- What does visual regression testing detect?
- Why can screenshot tests be flaky?

### Intermediate

- Which inputs must be controlled before comparing screenshots?
- How would you choose browsers for a support matrix?

### Advanced and Follow-up

- What important UI failures can a pixel-perfect screenshot still miss?

### Code Prediction

Why might a screenshot test change on CI after a font package update even when application code is unchanged?

## Practical Tasks

- Create a stable visual test for a responsive component at two viewport sizes.
- List the semantic and keyboard tests needed alongside that visual test.

## Readiness Criteria

You can use visual and cross-browser tests as targeted confidence tools, control their non-deterministic inputs, and explain what they cannot prove.

## References

- [Playwright visual comparisons](https://playwright.dev/docs/test-snapshots)
- [MDN: browser compatibility](https://developer.mozilla.org/docs/Learn_web_development/Extensions/Testing/Testing_strategies)
