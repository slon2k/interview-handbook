# Styling and State-Library Choices

## Definition

React applications can style components with ordinary CSS, CSS Modules, utility classes, CSS-in-JS, or a design system. They can also manage shared client state with libraries such as Redux, Zustand, or Jotai. These are architectural choices, not mandatory parts of React itself.

## How It Works

- CSS Modules scope class names at build time while preserving CSS as the styling language.
- Utility-first approaches compose predefined classes directly in markup and often emphasize a design-token system.
- CSS-in-JS colocates style definitions with components but may introduce runtime, tooling, or debugging trade-offs depending on the library.
- Redux emphasizes explicit actions, reducers, and predictable state transitions; Zustand and Jotai offer lighter-weight alternatives with different ownership models.
- A library should solve a demonstrated coordination problem rather than compensate for unclear component or state boundaries.

## Application

Choose styling based on team conventions, accessibility, design-system integration, build/runtime cost, and maintainability. Choose a client-state library only after classifying the state and confirming local state, context, or a server-state tool is insufficient.

The styling mechanism does not replace the platform rules in [Module 1](../m01-web-platform-foundations/README.md). CSS Modules, utility classes, and CSS-in-JS still need semantic HTML, visible focus states, sufficient contrast, responsive layout, and deliberate cascade control. Reuse custom properties or design tokens for shared values rather than embedding one-off colors and spacing in every component.

## Common Mistakes

- Choosing a styling library by popularity without considering the existing design system or build pipeline.
- Using global CSS names that collide across features.
- Putting server data into a client-state store and reimplementing caching and invalidation.
- Adding a global store to avoid lifting a small amount of state.
- Treating a library's API as the architecture instead of defining ownership first.
- Replacing a native control with a styled generic element and losing its keyboard and accessibility behavior.

## Common Interview Questions

### Foundation

- What styling approaches can a React application use?
- When would a client-state library be justified?

### Intermediate

- What are the trade-offs between CSS Modules and utility classes?
- How does Redux differ conceptually from a lightweight store?

### Advanced and Follow-up

- How would you choose a styling strategy for a multi-team application with a shared design system?
- Why should server state usually not be treated as ordinary client state?

### Code Prediction

Given a component that reads remote data from a global store, identify which cache invalidation and loading concerns the store now owns and what a server-state library might provide instead.

## Practical Tasks

- Compare two styling approaches for a feature and justify the choice using maintainability, accessibility, and team constraints.
- Review a proposed global store and identify which values should remain local or move to a server-state tool.

## Readiness Criteria

You can compare styling and client-state approaches and justify a choice based on ownership, maintainability, runtime cost, team conventions, and server-state needs.

## References

- [React: Thinking in React](https://react.dev/learn/thinking-in-react)
- [Redux documentation](https://redux.js.org/)
- [Zustand documentation](https://zustand.docs.pmnd.rs/)
- [Jotai documentation](https://jotai.org/)
