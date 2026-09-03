# Frontend Foundations

This track prepares a .NET-focused full-stack candidate to build, test, and discuss a modern TypeScript and React user interface backed by an ASP.NET Core API.

**Short on time?** Start with JavaScript, TypeScript, browser/API integration, and React. Web-platform and test/tooling topics provide the practical foundation around them.

## Scope

The core path targets junior-to-mid-level ASP.NET Core full-stack roles over a two-to-three-week preparation window. It assumes React is the primary UI framework and TypeScript is the default language.

Legacy ASP.NET MVC needs only working awareness of server-rendered HTML, forms, DOM events, and progressive JavaScript enhancement. Razor Pages and Blazor are ecosystem awareness topics, not core preparation. Angular belongs in a separate optional module when target roles require it.

## Learning Path

1. [Module 1: Web platform: HTML, CSS, and accessibility](m01-web-platform-foundations/README.md)
2. Module 2: JavaScript language and runtime
3. Module 3: TypeScript
4. Module 4: Browser platform and ASP.NET Core API integration
5. Module 5: React
6. Module 6: Frontend testing and tooling

## Learning Outcomes

By the end of this track, you should be able to:

- Build accessible, responsive UI using semantic HTML and modern CSS layout.
- Explain JavaScript values, scope, closures, async behavior, promises, and the browser event loop.
- Model UI state and API contracts with TypeScript while recognising its runtime limits.
- Diagnose common browser/API integration failures involving CORS, authentication, CSRF, caching, and cancelled or stale requests.
- Build React applications with functional components, hooks, React Router, typed data fetching, and clear loading, error, and empty states.
- Explain where local UI, shared, server, and URL state belong; describe TanStack Query or an equivalent server-state library at awareness level.
- Write user-focused component tests and explain the frontend test pyramid and common tooling choices.

## Modules

### Module 1 - Web Platform Foundations

**Priority:** High  
**Prerequisites:** None

See the [Web Platform Foundations module](m01-web-platform-foundations/README.md).

### Module 2 - JavaScript Language and Runtime

**Priority:** Critical  
**Prerequisites:** Module 1

- Primitive and reference values; objects, arrays, equality, and mutation
- Type coercion, `==` versus `===`, truthy/falsy values, `null` versus `undefined`
- Scope, `var`, `let`, `const`, hoisting, and the temporal dead zone
- Functions, arrow functions, `this`, and closures
- Destructuring, spread/rest syntax, optional chaining, nullish coalescing, modules, and array methods
- Errors, promises, `async`/`await`, event loop, microtasks, and macrotasks
- Prototype inheritance, classes, iterables/generators, and garbage collection at awareness level

### Module 3 - TypeScript

**Priority:** Critical  
**Prerequisites:** Module 2

- Type inference, annotations, interfaces, type aliases, and structural typing
- Unions, literal types, narrowing, discriminated unions, and optional properties
- Function types, generics, constraints, `keyof`, `typeof`, and indexed-access types
- Utility types, `unknown`, `any`, `never`, assertions, and strict compiler settings
- Typing UI state, forms, and API DTOs; null and optional-property semantics across TypeScript and C#
- Runtime validation of untrusted API data
- Mapped types, conditional types, overloads, declaration files, enums, and variance at awareness level

> TypeScript types do not normally exist at runtime and do not validate external data.

### Module 4 - Browser Platform and ASP.NET Core API Integration

**Priority:** High  
**Prerequisites:** Modules 2 and 3

- DOM events, bubbling, capturing, delegation, and form events
- Browser rendering, reflow, repaint, and browser developer tools
- `fetch`, `AbortController`, request cancellation, and stale-response handling
- Same-origin policy, CORS, environment variables, cookies, browser storage, HTTP caching, XSS, and CSRF
- Client-side routing, URLs, history, and URL state
- API contracts: DTOs, pagination, error payloads including `ProblemDetails`, and validation errors
- Authentication consequences for cookie and bearer-token approaches; handling `401`, `403`, and login redirects
- OpenAPI/Swagger-generated TypeScript clients versus hand-written API wrappers
- React dev-server proxying, environment-specific API URLs, and frontend environment-variable safety
- WebSockets and Web Workers at awareness level

**Scope boundaries:** HTTP semantics belong in [Module 7](../dotnet/m07-http-rest-api-design/README.md), application-security theory in [Module 12](../dotnet/m12-application-security/README.md), and general performance/observability in [Module 13](../dotnet/m13-performance-diagnostics-observability/README.md). This module covers their browser-facing effects.

### Module 5 - React

**Priority:** Critical  
**Prerequisites:** Modules 1-4

#### Core Rendering Model

- Functional components, JSX, props, state, events, conditional rendering, lists, and keys
- Component identity, composition, lifting state, controlled and uncontrolled components
- Derived state versus stored state
- What causes rerenders, reconciliation, referential equality, and `React.memo`
- Route-based code splitting with `React.lazy` and `Suspense`; awareness of bundle-size and loading trade-offs

#### Hooks

- Rules of Hooks, `useState`, functional updates, `useEffect`, dependencies, cleanup, and stale closures
- Synchronising with external systems versus event handling
- `useRef`, `useReducer`, `useContext`, and custom hooks
- `useMemo` and `useCallback` as targeted optimisation tools rather than defaults

#### Application Structure and Data

- React Router, nested routes, route parameters, and URL state
- Typed data fetching; loading, error, empty, and retry states
- Request cancellation and effect race conditions
- Local UI, shared, server, and URL state
- Forms: controlled and uncontrolled components, form libraries such as React Hook Form or Formik, validation, error display, and submission handling
- Styling approaches: CSS Modules, utility-first CSS such as Tailwind, and CSS-in-JS awareness; choosing and justifying a strategy
- Client-state libraries: Redux, Zustand, and Jotai awareness; local UI versus shared client state versus server state
- Error boundaries, Suspense, and feature-oriented component structure
- Server-state libraries such as TanStack Query at awareness level: caching, invalidation, and why raw `useEffect` fetching becomes limiting

### Module 6 - Frontend Testing and Tooling

**Priority:** High  
**Prerequisites:** Modules 3 and 5

#### Testing React Components and Applications

- React Testing Library: queries, user-oriented interaction tests, and testing asynchronous UI states
- Mocking API requests with Mock Service Worker (MSW)
- Component integration tests and the limitations of hook-only tests
- Unit, component, integration, and end-to-end tests; the frontend test pyramid
- Vitest or Jest, Playwright or Cypress, accessibility-oriented queries, and test reliability
- Snapshot testing trade-offs and avoiding implementation-detail tests

#### Tooling and Delivery

- npm, `package.json`, lock files, semantic versioning, and dependency categories
- Vite, build versus development modes, environment variables, source maps, and browser developer tools
- TypeScript compiler, ESLint, Prettier, bundling, tree shaking, lazy loading, and code splitting
- Bundle size, image optimisation, Core Web Vitals, network waterfalls, render profiling, browser caching, and frontend CI basics

## Scope Boundaries

- React is the core framework; Angular is a future optional module for roles that explicitly require it.
- ASP.NET MVC, Razor Pages, and Blazor are not core modules. Know their role in the .NET ecosystem, but prepare React first.
- Backend API, authentication, security, testing, performance, and delivery concepts should link to the .NET track instead of being repeated here.

## Suggested Learning Sequence

1. Complete the essential HTML, CSS, and accessibility topics in Module 1.
2. Focus on Module 2 async behavior, closures, values/references, modules, and array transformations.
3. Complete Module 3 through typed API contracts and runtime-validation limits.
4. Study Module 4 browser/API integration alongside the linked .NET HTTP and security modules.
5. Spend the largest share of time on Module 5: rendering, hooks, Router, API states, and state boundaries.
6. Finish with Module 6 testing and tooling, focusing on Vitest, React Testing Library, MSW, and Playwright awareness.

## Practical Deliverables

Use these as explanation, code-reading, debugging, or small implementation exercises; a separate capstone is not required.

- Diagnose an inaccessible form and describe the semantic HTML, label, focus, and validation fixes.
- Explain a JavaScript event-loop output and repair a mutation or closure bug.
- Type a paginated ASP.NET Core response and explain why its TypeScript type is not runtime validation.
- Diagnose a CORS, cookie/CSRF, `401`, `403`, or `ProblemDetails` handling issue from the browser's perspective.
- Review a React component for incorrect effect dependencies, stale requests, derived state, or unstable list keys.
- Explain a route and URL-state design for a searchable, paginated API resource.
- Write or review a React Testing Library test covering loading, error, and successful data states.

## Interview Coverage

Each topic should include:

- Foundation questions for terminology and core behavior.
- Intermediate questions based on everyday implementation and trade-offs.
- Advanced follow-ups that require debugging, design reasoning, or performance judgment.
- Code-prediction questions involving JavaScript async behavior, TypeScript narrowing, browser requests, and React rendering.

## Ecosystem Awareness

- **ASP.NET MVC and Razor Pages:** Server-rendered HTML with forms and progressive JavaScript enhancement.
- **Blazor:** Component-based .NET UI; useful to recognise but niche relative to React for this target.
- **Angular:** A distinct TypeScript framework path involving templates, dependency injection, RxJS, signals, change detection, routing, and Angular-specific testing.
