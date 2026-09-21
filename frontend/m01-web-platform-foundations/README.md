# Module 1 - Web Platform Foundations

**Status:** Complete  
**Priority:** High  
**Prerequisites:** None

## Scope

This module establishes the browser-facing foundations needed to build, review, and debug usable, accessible, and responsive interfaces — before any framework enters the picture. It focuses on the decisions interviewers commonly probe rather than attempting to be a complete HTML or CSS reference.

## Why This Matters in Interviews

React does not replace the web platform. A strong frontend candidate can choose the right HTML element, explain how the browser interprets it, build a responsive layout, and diagnose an accessibility or CSS bug before reaching for a framework-specific abstraction. Interviewers are usually testing whether you can choose semantic elements and explain their built-in behavior, make forms and interactive controls usable with a keyboard and assistive technology, predict cascade/box-model/layout behavior from a small example, and inspect and repair a real UI rather than recite an element or property catalogue.

## Learning Outcomes

By the end of this module, you should be able to:

- Structure a page with semantic HTML and choose elements by meaning rather than appearance.
- Build forms with correctly associated labels, native validation, useful error messages, and keyboard support.
- Explain accessible names, focus management, color contrast, and when ARIA is actually appropriate.
- Use the cascade, specificity, inheritance, and custom properties deliberately, and predict which properties inherit.
- Choose between normal flow, Flexbox, Grid, and positioning for a given layout, and justify the choice.
- Explain responsive units, media queries, and fluid sizing, and build layouts that adapt without breaking.
- Inspect a small HTML or CSS example, predict its behavior, and explain a practical fix.

## Topics

### 1. Meaningful and Usable Markup

- [Semantic HTML and document structure](semantic-html-and-document-structure.md)
- [Forms and native controls](forms-and-native-controls.md)
- [Accessibility and keyboard interaction](accessibility-and-keyboard-interaction.md)

### 2. CSS Foundations

- [CSS cascade, specificity, and inheritance](css-cascade-specificity-and-inheritance.md)
- [CSS custom properties](css-custom-properties.md)
- [Box model, sizing, and overflow](box-model-sizing-and-overflow.md)

### 3. Layout Systems

- [Normal flow and positioning](normal-flow-and-positioning.md)
- [Flexbox](flexbox.md)
- [Grid](grid.md)

### 4. Responsive Interface Design

- [Responsive design and media queries](responsive-design-and-media-queries.md)

### 5. Production Styling Awareness

- [CSS architecture and styling strategies](css-architecture-and-styling-strategies.md)

## Scope Boundaries

- Browser APIs, DOM events, rendering, and network requests belong in Module 4 - Browser Platform and ASP.NET Core API Integration.
- React component structure and rendering belong in Module 5 - React.
- Frontend test tooling belongs in Module 6 - Frontend Testing and Tooling.
- Advanced ARIA patterns and screen-reader-specific differences, SEO beyond semantic structure, stacking-context edge cases, and animation performance are working-awareness topics, not covered in depth here. CSS architecture choices are introduced at working-awareness level and applied in Module 5.

## Suggested Learning Sequence

1. Establish semantic document structure, then build forms using the native controls that already provide the expected browser behavior.
2. Add accessible names, keyboard interaction, focus management, and contrast checks to that markup.
3. Learn how the cascade and inheritance resolve, then use the box model to reason about an element's size and overflow.
4. Build layouts progressively: normal flow first, then positioning for overlays, Flexbox for one-dimensional relationships, and Grid for two-dimensional ones.
5. Make those layouts adapt to space, media, and user preferences with responsive units and media queries.

## Practical Deliverables

- Repair an inaccessible form without changing its visual design.
- Convert a `div`-based page into semantic landmarks, headings, navigation, and controls.
- Build a responsive two-column layout that becomes a single column without duplicating markup.
- Explain a layout bug caused by specificity, an unexpected containing block, or the flex-item minimum-size trap.
- Implement a dark-mode toggle using CSS custom properties and `prefers-color-scheme`, with no per-element duplication.
- Predict the result of a short markup or CSS snippet and describe the cheapest browser check that would confirm your answer.
- Diagnose a broken responsive layout in browser DevTools, naming the computed style or box-model constraint that causes the visible symptom.
- Rebuild a compact interface from a reference image using semantic HTML and a deliberate layout system, then justify the CSS approach.

## Interview Coverage

Each topic includes basic, intermediate, and advanced/follow-up questions plus a code-prediction prompt. For every topic, practice four passes: **explain** the foundation question in plain language, **implement** or correct the smallest useful HTML/CSS example, **debug** by naming the visible symptom, a falsifiable hypothesis, and the browser tool or check that would confirm it, and **defend** the trade-off and how the decision would affect a React component built on top of it.

## References

- [MDN: HTML](https://developer.mozilla.org/docs/Web/HTML)
- [MDN: CSS](https://developer.mozilla.org/docs/Web/CSS)
- [MDN: Accessibility](https://developer.mozilla.org/docs/Web/Accessibility)
