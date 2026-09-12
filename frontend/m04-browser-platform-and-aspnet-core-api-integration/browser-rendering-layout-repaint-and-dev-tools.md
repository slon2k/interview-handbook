# Browser Rendering, Layout, Repaint, and Developer Tools

## Definition

The browser turns HTML, CSS, and JavaScript into visible pixels. The render pipeline includes parsing, style calculation, layout, paint, and compositing. A front-end candidate should know where expensive work happens and how browser developer tools expose it.

## How It Works

- Layout calculates the geometry of elements based on the DOM, CSS, and viewport.
- Repaint redraws pixels when visual styles change but layout does not.
- Reflow and layout work happens when size, position, or content changes affect geometry.
- Developer tools can show CPU usage, paint events, layout thrashing, and network timing.
- Heavy DOM updates, repeated measurements, and forced reflow are common causes of janky UI.

## Application

Use browser developer tools to inspect performance issues and isolate whether the problem is rendering, reflow, or data loading. Good UI work minimizes redundant DOM reads and writes and avoids layout thrashing during loops.

## Common Mistakes

- Reading a layout value repeatedly within a loop and forcing reflow on each iteration.
- Assuming every style change causes the same cost; some styles are much more expensive than others.
- Calling browser features like `offsetWidth` or `getBoundingClientRect` without considering timing.
- Confusing a network issue with a rendering issue when the page is visibly blocked.

## Common Interview Questions

### Foundation

- What is the difference between layout and paint?
- Why is a browser repaint different from a full reflow?

### Intermediate

- How can reading layout values and then writing styles cause jank?
- What does a browser developer tool show when a page is slow?

### Advanced and Follow-up

- Why does batching DOM reads and writes improve rendering performance?
- How would you explain a page that appears to “freeze” even though the JavaScript has already finished?

### Code Prediction

Given a loop that reads `element.offsetHeight` and then writes `element.style.height` repeatedly, explain why this can trigger excessive layout and how batching could help.

## Practical Tasks

- Inspect a page in DevTools and identify whether the slow behavior is paint, layout, or network latency.
- Rewrite a style loop that reads and writes DOM values in a way that avoids repeated reflow.

## Readiness Criteria

You can explain the render pipeline, identify the source of a janky UI, and reason about where layout, paint, and DOM work happen in the browser.

## References

- [MDN: Rendering in browser](https://developer.mozilla.org/en-US/docs/Mozilla/Gecko/Rendering_Engine)
- [Chrome DevTools Performance docs](https://developer.chrome.com/docs/devtools/performance/)
- [MDN: `getBoundingClientRect()`](https://developer.mozilla.org/en-US/docs/Web/API/Element/getBoundingClientRect)
