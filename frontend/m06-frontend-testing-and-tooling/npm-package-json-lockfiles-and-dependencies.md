# npm, `package.json`, Lockfiles, and Dependencies

## Definition

`package.json` describes a JavaScript project's scripts, package metadata, and dependency ranges. A lockfile records the resolved dependency graph so installs are reproducible. Dependencies ship or run with the application; development dependencies support building, testing, linting, and tooling.

## Alternatives & Trade-offs

Semver ranges allow compatible updates but can introduce changed transitive code over time. Lockfiles make CI and local installs agree, but must be reviewed as supply-chain changes rather than generated noise. `npm ci` favors reproducibility in CI; `npm install` updates the lockfile when intentionally changing dependencies.

## How It Works

Use scripts as named, reviewable project entry points such as `test`, `typecheck`, `lint`, and `build`. Place runtime packages in `dependencies`, tooling in `devDependencies`, and peer dependencies in libraries that require the host application to provide a compatible package.

## Application

Commit one package-manager lockfile, pin the Node and package-manager expectations where the repository requires it, and update packages deliberately. Inspect why a package exists and its bundle impact before adding it.

## Common Mistakes

- Deleting a lockfile to repair an install without understanding the graph change.
- Using a dev dependency for code required at runtime.
- Ignoring peer-dependency warnings from a library with a framework compatibility constraint.
- Treating a large lockfile diff as unreviewable.

## Common Interview Questions

- What is the purpose of a lockfile?
- When should CI use `npm ci`?
- How do dependencies, dev dependencies, and peer dependencies differ?

## Practical Tasks

- Add a package through the package manager and explain each changed manifest/lockfile entry.
- Compare a clean reproducible install with an install that updates dependency resolutions.

## Readiness Criteria

You can explain package manifests and lockfiles, choose the correct dependency category, and maintain reproducible installs.

## References

- [npm package.json](https://docs.npmjs.com/cli/v11/configuring-npm/package-json)
- [npm ci](https://docs.npmjs.com/cli/v11/commands/npm-ci)
