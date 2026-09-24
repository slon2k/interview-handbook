# TypeScript, ESLint, Prettier, and Automated Quality Checks

## Definition

TypeScript checks type consistency, ESLint identifies configurable code-quality problems, and Prettier formats source consistently. They solve different problems and work best as repeatable commands run locally and in CI.

## Alternatives & Trade-offs

A formatter reduces review noise but cannot establish correctness. Lint rules can prevent common bugs but should not become an unexplained obstacle. TypeScript catches many invalid states but cannot validate runtime API data. Use each tool for its intended layer.

## How It Works

Run `tsc --noEmit` or the repository's typecheck command separately from bundling where practical. Configure ESLint with framework-aware rules such as React hooks rules. Let Prettier own formatting and avoid overlapping style rules that cause formatter-linter churn.

## Application

Expose `typecheck`, `lint`, `format:check`, and `test` scripts. Use editor integration for rapid feedback and CI for enforcement. Document exceptions beside the rule disablement and remove them when the reason no longer applies.

## Common Mistakes

- Treating a successful bundle as a complete typecheck.
- Disabling a hooks dependency warning instead of reasoning about the synchronization model.
- Expecting lint or formatting to validate runtime data.
- Enforcing checks only through a developer's editor.

## Common Interview Questions

- What different failure classes do TypeScript, ESLint, and Prettier detect?
- Why should CI run checks even when editors do?
- When is a lint suppression justified?

## Practical Tasks

- Add separate typecheck, lint, and format-check scripts.
- Repair a hooks lint warning by changing the design rather than suppressing it.

## Readiness Criteria

You can assign quality tools to their proper role, run them consistently, and justify an exception rather than silencing it by default.

## References

- [TypeScript compiler options](https://www.typescriptlang.org/tsconfig/)
- [ESLint documentation](https://eslint.org/docs/latest/)
- [Prettier documentation](https://prettier.io/docs/)
