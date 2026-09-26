# Module 5 - React

**Status:** Complete  
**Priority:** Critical  
**Prerequisites:** [Module 1 - Web Platform Foundations](../m01-web-platform-foundations/README.md), [Module 2 - JavaScript Language and Runtime](../m02-javascript-language-and-runtime/README.md), [Module 3 - TypeScript](../m03-typescript/README.md), and [Module 4 - Browser Platform and ASP.NET Core API Integration](../m04-browser-platform-and-aspnet-core-api-integration/README.md)

## Scope

This module covers React as the component and application layer on top of the JavaScript, TypeScript, browser, and API foundations established earlier in the track. It focuses on functional components, JSX, rendering, component identity, hooks, state boundaries, routing, forms, typed data fetching, and practical application structure.

The emphasis is on understanding React's programming model rather than memorising library APIs. A strong React developer can explain why a component rerenders, when an effect is appropriate, where state belongs, how a request race happens, and how to keep UI states honest and maintainable.

## Why This Matters in Interviews

React interviews commonly probe component identity, list keys, derived state, effect dependencies, stale closures, controlled forms, state ownership, and data-fetching failure modes. Interviewers are looking for candidates who can reason about rendering and synchronization instead of adding effects or memoization until a symptom disappears.

## Learning Outcomes

By the end of this module, you should be able to:

- Build functional components with typed props, JSX, events, conditional rendering, lists, and stable keys.
- Type component props, children, DOM events, render callbacks, and generic component contracts without `any`.
- Explain component identity, composition, lifting state, controlled and uncontrolled inputs, and derived state.
- Explain rerenders, reconciliation, referential equality, and when `React.memo` changes behavior.
- Use `useState`, functional updates, `useEffect`, cleanup, `useRef`, `useReducer`, `useContext`, and custom hooks appropriately.
- Distinguish synchronizing with an external system from handling a user event, and avoid stale closures and incorrect dependencies.
- Build routed React features with typed URL state, forms, validation, and clear loading, error, empty, and retry states.
- Decide where local UI, shared client, server, and URL state should live.
- Explain when memoization, code splitting, TanStack Query, or a client-state library is justified.

## Topics

### 1. Rendering Fundamentals

- [Components, JSX, props, and rendering](components-jsx-props-and-rendering.md)
- [TypeScript components, props, events, and generics](typescript-components-props-events-and-generics.md)
- [Component identity, composition, and state](component-identity-composition-and-state.md)
- [Reconciliation, rerenders, and keys](reconciliation-rerenders-and-keys.md)

### 2. Hooks and Synchronization

- [Hooks, `useState`, and functional updates](hooks-rules-usestate-and-functional-updates.md)
- [`useEffect`, dependencies, cleanup, and synchronization](useeffect-dependencies-cleanup-and-synchronization.md)
- [`useRef`, `useReducer`, `useContext`, and custom hooks](useref-usereducer-usecontext-and-custom-hooks.md)
- [`useMemo`, `useCallback`, and rendering performance](usememo-usecallback-and-rendering-performance.md)

### 3. Routing, State, and Data

- [React Router, routes, and URL state](react-router-routes-and-url-state.md)
- [Typed data fetching and async UI states](typed-data-fetching-and-async-ui-states.md)
- [Forms, validation, and submission](forms-validation-and-submission.md)
- [State categories and application architecture](state-categories-and-application-architecture.md)
- [Guided capstone: searchable catalog](capstone-searchable-catalog.md)

### 4. Boundaries and Ecosystem Awareness

- [Error boundaries, Suspense, and code splitting](error-boundaries-suspense-and-code-splitting.md)
- [Styling and state-library choices](styling-and-state-library-choices.md)
- [TanStack Query and server state](tanstack-query-and-server-state.md)
- [Modern React 19 awareness](modern-react-19-awareness.md)

## Scope Boundaries

- JavaScript closures, promises, and event-loop behavior belong in [Module 2](../m02-javascript-language-and-runtime/README.md).
- TypeScript modeling, narrowing, generics, and runtime-validation limits belong in [Module 3](../m03-typescript/README.md).
- DOM events, browser rendering, `fetch`, CORS, authentication, URL mechanics, and API contracts belong in [Module 4](../m04-browser-platform-and-aspnet-core-api-integration/README.md); this module explains how React uses them.
- React Testing Library, MSW, Vitest or Jest, Playwright or Cypress, bundling, and frontend delivery belong in [Module 6](../m06-frontend-testing-and-tooling/README.md).
- HTTP semantics, backend API design, and security architecture remain in the related .NET modules.

## Suggested Learning Sequence

1. Learn components, JSX, props, lists, keys, and conditional rendering.
2. Understand component identity, state ownership, composition, and rerender behavior.
3. Learn `useState` and `useEffect`, then distinguish event handling from external-system synchronization.
4. Add refs, reducers, context, custom hooks, and targeted performance tools.
5. Apply the model to routing, URL state, typed data fetching, async UI states, and forms.
6. Finish with component architecture, boundaries, code splitting, styling, server-state libraries, and modern React awareness.
7. Complete the guided capstone by applying the preceding modules to one API-backed feature.

## Practical Deliverables

- Review a component for incorrect keys, derived-state drift, unnecessary effects, and unstable callback assumptions.
- Build a typed list feature with route parameters, loading/error/empty states, request cancellation, and retry behavior.
- Model a controlled form with field-level validation and server-side validation errors.
- Refactor a component with prop drilling into a deliberate composition, reducer, or context design.
- Explain whether a feature needs local state, shared state, URL state, or a server-state library.
- Complete a searchable, paginated feature with typed URL state, cancellation, field-level server validation, and deliberate auth/conflict behavior.

## Cross-Module Connections

- [Module 1: Semantic HTML](../m01-web-platform-foundations/semantic-html-and-document-structure.md) informs the HTML emitted by every React component.
- [Module 2: Closures](../m02-javascript-language-and-runtime/closures.md) explains stale values in callbacks and effects; [promises and async/await](../m02-javascript-language-and-runtime/promises-and-async-await.md) informs data-loading behavior.
- [Module 3: UI state](../m03-typescript/typing-ui-state-forms-and-async-results.md) and [type-level reuse](../m03-typescript/keyof-typeof-indexed-access-and-utility-types.md) underpin typed components, forms, and async branches.
- [Module 4: Fetch and cancellation](../m04-browser-platform-and-aspnet-core-api-integration/fetch-requests-cancellation-and-stale-responses.md), [API contracts](../m04-browser-platform-and-aspnet-core-api-integration/api-dtos-pagination-and-validation-errors.md), and [URL state](../m04-browser-platform-and-aspnet-core-api-integration/client-side-routing-urls-and-history-state.md) describe the browser behavior that React features coordinate.

## Interview Coverage

Each topic includes basic, intermediate, and advanced/follow-up questions plus a code-prediction or debugging prompt. Practice explaining what React knows during rendering, what an effect synchronizes, and which state transition or identity rule causes the observed behavior.

## References

- [React documentation](https://react.dev/learn)
- [React TypeScript Cheatsheet](https://react-typescript-cheatsheet.netlify.app/)
- [React Router documentation](https://reactrouter.com/home)
- [TanStack Query documentation](https://tanstack.com/query/latest)
