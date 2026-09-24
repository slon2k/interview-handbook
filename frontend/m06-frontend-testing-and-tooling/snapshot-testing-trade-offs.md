# Snapshot Testing Trade-offs

## Definition

A snapshot test records a serialized rendering or value and fails when a later run differs. It can detect unexpected structural change, but it does not explain whether a change is correct, accessible, or meaningful to a user.

## Alternatives & Trade-offs

Snapshots are cheap to add and useful for small, stable output such as a formatter, a compact configuration object, or deliberately versioned markup. Large component snapshots are noisy: developers can update them mechanically, and the review diff rarely identifies the behavior that changed. Prefer direct assertions for user-facing behavior.

## How It Works

The test framework compares the current serialized output with a committed baseline. A changed snapshot is a prompt to inspect the diff, not evidence that the code is wrong. The review question is: which behavior is protected, and would a direct assertion express it better?

## Application

Use a narrow snapshot only when it has a stable, readable contract and reviewing changes is meaningful. For components, test roles, names, state messages, values, and interactions. Use visual regression tooling when layout is the contract, while retaining semantic and interaction tests for usability.

## Common Mistakes

- Snapshotting a whole application tree and approving every change without review.
- Treating a snapshot as proof that an accessible control works.
- Using snapshots instead of testing loading, error, and interaction states.
- Allowing generated snapshots to hide a meaningful source-code change.

## Common Interview Questions

### Foundation

- What does a snapshot test assert?
- Why can large snapshots become low value?

### Intermediate

- When would a snapshot be a reasonable choice?
- What direct assertion would be better for a submit button?

### Advanced and Follow-up

- How would you review a snapshot change in a pull request?

### Code Prediction

If a component's class names change but its roles, names, and interactions do not, should a behavior-focused test fail? Should a broad markup snapshot?

## Practical Tasks

- Replace one large UI snapshot with focused behavior assertions.
- Identify one small, stable value where a snapshot remains useful and explain why.

## Readiness Criteria

You can explain the signal and noise of snapshots, review them critically, and choose direct user-focused assertions by default.

## References

- [Jest snapshot testing](https://jestjs.io/docs/snapshot-testing)
- [Testing Library guiding principles](https://testing-library.com/docs/guiding-principles/)
