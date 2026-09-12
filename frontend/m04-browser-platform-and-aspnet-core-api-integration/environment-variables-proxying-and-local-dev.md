# Environment Variables, API Proxying, and Local Development Configuration

## Definition

Client applications often need different values in local development, testing, and production. Environment variables and proxy configuration let the browser call the correct API without hard-coding remote endpoints into the frontend.

## How It Works

- Browser code runs in a web context and cannot safely access arbitrary server secrets.
- Environment-specific URLs are usually configured at build time or via a dev server proxy.
- A dev server proxy can route a frontend request to a backend during local development without CORS issues.
- Production configuration must avoid leaking internal endpoints or credentials to end users.
- The same code path can behave differently when local origin, mode, or environment variables change.

## Application

Keep API URLs and configuration explicit and environment-aware. Centralize the client configuration so it is obvious which endpoint is used in each environment and how local development differs from deployed environments.

## Common Mistakes

- Hard-coding a production API URL into client code and then forgetting how it behaves in dev or staging.
- Assuming a browser can call a backend directly without considering CORS or same-origin rules.
- Exposing secrets in client-side configuration.
- Using a proxy for production and accidentally creating a fragile deployment topology.

## Common Interview Questions

### Foundation

- Why should a frontend not store secrets in browser code?
- What is a dev-server proxy used for?

### Intermediate

- How does local development differ from production configuration?
- Why does a browser app sometimes need an environment variable for its API base URL?

### Advanced and Follow-up

- How would you debug a frontend that works locally via proxy but fails in production with a wrong origin or wrong base URL?
- How would you keep local and production values consistent while avoiding accidental credential leakage?

### Code Prediction

Given a frontend app running on `localhost:3000` calling a backend at `https://api.internal`, explain why a direct browser request may fail and why a local proxy can solve it without changing the app's code path.

## Practical Tasks

- Define local and production API configuration for a frontend app.
- Explain why a reverse proxy or dev-server proxy sometimes solves a CORS issue in local development while not being a production-safe solution by itself.

## Readiness Criteria

You can explain environment-driven API configuration, dev-server proxies, and the practical limits of browser-side configuration in real deployment scenarios.

## References

- [MDN: Environment variables in browser apps](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/await)
- [Vite: Environment variables](https://vitejs.dev/guide/env-and-mode.html)
- [Create React App: Proxying API requests in development](https://create-react-app.dev/docs/proxying-api-requests-in-development/)
