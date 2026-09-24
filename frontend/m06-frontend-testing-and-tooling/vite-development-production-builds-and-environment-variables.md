# Vite Development, Production Builds, and Environment Variables

## Definition

Vite provides a development server with fast module updates and a production build that transforms, bundles, and emits deployable assets. Its client environment variables are substituted at build time and become visible to anyone who can load the frontend bundle.

## Alternatives & Trade-offs

Vite is a practical default for modern React projects. Other build systems can be appropriate for framework or legacy constraints. Regardless of tool, client configuration is public configuration, not a secret store; a frontend cannot safely contain a private key.

## How It Works

Vite exposes only variables with the configured public prefix, commonly `VITE_`. Development values may come from local `.env` files; production values must be supplied during the build or deployment process. Source maps aid diagnosis but need a deliberate publication policy because they can expose source structure.

## Application

Use environment variables for public API base URLs, build labels, and feature configuration that is safe to disclose. Validate required configuration early, document local setup, and use a backend or secure runtime boundary for secrets.

## Common Mistakes

- Prefixing a secret with `VITE_` and assuming it remains private.
- Confusing a dev-server proxy with a production API routing solution.
- Publishing source maps without considering access and error-reporting needs.
- Letting local environment files silently replace required deployment configuration.

## Common Interview Questions

- What differs between Vite development and production modes?
- Why cannot a frontend environment variable be a secret?
- What are source maps for, and what is their trade-off?

## Practical Tasks

- Configure a public API URL for local and production builds and verify the emitted bundle behavior.
- Identify a secret that was incorrectly placed in client configuration and move its use behind a server boundary.

## Readiness Criteria

You can distinguish dev-server behavior from a production build, configure public values safely, and explain why secrets stay server-side.

## References

- [Vite env and mode](https://vite.dev/guide/env-and-mode)
- [Vite build](https://vite.dev/guide/build)
