# Module 1 - Web Platform Foundations

**Status:** Complete  
**Priority:** High  
**Prerequisites:** None

## Scope

This module establishes the browser-facing foundations needed to build, review, and debug usable, accessible, and responsive interfaces. It focuses on the decisions interviewers commonly probe rather than attempting to be a complete HTML or CSS reference.

## Why This Matters in Interviews

React does not replace the web platform. A strong frontend candidate can choose the right HTML primitive, explain how the browser interprets it, build a responsive layout, and diagnose accessibility or CSS bugs before reaching for a framework-specific abstraction.

Interviewers are usually testing whether you can:

- Choose semantic elements and explain their built-in behaviour.
- Make forms and interactive controls usable with a keyboard and assistive technology.
- Predict the cascade, box model, and layout behaviour from a small example.
- Select an appropriate layout technique and explain its responsive trade-offs.
- Inspect and repair a real UI rather than reciting an element or property catalogue.

## Learning Outcomes

By the end of this module, you should be able to:

- Structure a page with semantic HTML and choose elements by meaning rather than appearance.
- Build forms with correctly associated labels, native validation, useful error messages, and keyboard support.
- Explain accessible names, focus management, colour contrast, and when ARIA is appropriate.
- Use the cascade, specificity, inheritance, and custom properties deliberately.
- Choose between normal flow, Flexbox, Grid, and positioning for a responsive layout.
- Explain responsive units, media queries, stacking contexts, and common CSS maintenance trade-offs.
- Inspect a small HTML or CSS example, predict its behaviour, and explain a practical fix.

## Topics and Priorities

### Must Know

1. [Semantic HTML and document structure](semantic-html-and-document-structure.md)
2. [Forms and native controls](forms-and-native-controls.md)
3. [Accessibility and keyboard interaction](accessibility-and-keyboard-interaction.md)
4. [CSS cascade and box model](css-cascade-and-box-model.md)
5. [Layout and responsive design](layout-and-responsive-design.md)

### Working Awareness

- Advanced ARIA patterns and screen-reader differences.
- SEO details beyond semantic structure.
- Stacking-context edge cases and animation performance.
- CSS architecture options such as CSS Modules, utility classes, and layered global styles.

## Interview Practice

For every topic, practise four passes:

1. **Explain:** answer the foundation question in plain language.
2. **Implement:** write or correct the smallest useful HTML or CSS example.
3. **Debug:** identify the visible symptom, form a hypothesis, and name the browser tool or test that would confirm it.
4. **Defend:** explain the trade-off and how the decision affects a React component.

## Scope Boundaries

- Browser APIs, DOM events, rendering, and network requests belong in Module 4 - Browser Platform and ASP.NET Core API Integration.
- React component structure and rendering belong in Module 5 - React.
- Frontend test tooling belongs in Module 6 - Frontend Testing and Tooling.

## Suggested Learning Sequence

1. Start with semantic HTML, forms, and native controls.
2. Add keyboard interaction, accessible names, focus management, and contrast checks.
3. Practise the cascade, specificity, inheritance, and box model.
4. Build a responsive page using normal flow, Flexbox, and Grid.
5. Rehearse debugging and explain each choice as if reviewing a pull request.

## Practical Deliverables

- Repair an inaccessible form without changing its visual design.
- Convert a `div`-based page into semantic landmarks, headings, navigation, and controls.
- Build a responsive two-column layout that becomes a single column without duplicating markup.
- Explain a layout bug caused by specificity, an unexpected containing block, or a stacking context.
- Predict the result of a short markup or CSS snippet and describe the cheapest check that would confirm your answer.

## Readiness Checklist

- [ ] I can explain why a native HTML element is preferable to a custom control in a given scenario.
- [ ] I can make a form's labels, validation errors, focus order, and submission state understandable without relying on colour.
- [ ] I can explain when ARIA is appropriate and identify incorrect or redundant ARIA.
- [ ] I can predict common cascade, box-model, Flexbox, Grid, overflow, and stacking-context behaviour.
- [ ] I can build and test a responsive layout at narrow and wide viewport sizes.
- [ ] I can inspect a UI, state a falsifiable hypothesis, and choose a focused browser or accessibility check.
