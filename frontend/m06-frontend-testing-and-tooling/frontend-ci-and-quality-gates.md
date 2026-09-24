# Frontend CI and Quality Gates

## Definition

Frontend continuous integration (CI) runs reproducible checks against a proposed change before it merges. Quality gates commonly include dependency installation, type checking, linting, formatting checks, unit/component tests, a production build, and selected browser tests.

## Alternatives & Trade-offs

Running every E2E and cross-browser check on every change maximizes immediate confidence but can slow feedback and consume resources. A proportionate pipeline keeps fast correctness checks on pull requests and schedules broader browser, visual, or external-integration checks based on risk.

## How It Works

Start from a clean lockfile-based install, run named scripts, and publish useful failure artifacts such as test results, Playwright traces, screenshots, and build reports. Branch protections should require checks that are stable and meaningful; a permanently ignored failing check is worse than no gate.

## Application

Order checks to fail quickly: install, typecheck/lint/format, component tests, build, then browser tests. Cache package downloads when it is safe, not build outputs that could hide a broken configuration. Keep credentials scoped and never expose secrets to untrusted pull-request code.

## Common Mistakes

- Running a different command in CI than developers run locally.
- Treating a successful build as proof that tests and types passed.
- Keeping flaky required checks without ownership or diagnostics.
- Storing real production credentials in frontend test configuration.

## Common Interview Questions

- Which frontend checks would you run for every pull request?
- Why should CI install from the lockfile?
- What failure artifacts should browser tests retain?

## Practical Tasks

- Define a pull-request pipeline for a React app using the module's scripts and targeted Playwright coverage.
- Diagnose a CI-only browser failure using a trace or screenshot artifact.

## Readiness Criteria

You can design a proportionate frontend CI pipeline, preserve actionable artifacts, and keep local and CI quality checks aligned.

## References

- [GitHub Actions documentation](https://docs.github.com/actions)
- [Playwright CI](https://playwright.dev/docs/ci)
