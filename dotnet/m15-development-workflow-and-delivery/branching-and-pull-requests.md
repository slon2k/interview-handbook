# Branching and Pull Requests

## Definition

A branch is an isolated line of development; a pull request (PR) — also called a merge request on some platforms — is a proposal to merge one branch's changes into another, serving as the point where code review (next topic), automated checks (CI, Module 11), and discussion happen before the change becomes part of the shared history.

```bash
git checkout -b feature/add-discount-calculation
# ... make commits ...
git push origin feature/add-discount-calculation
# then open a PR on GitHub/GitLab/Azure DevOps proposing to merge this branch into main
```

## Alternatives & Trade-offs

Committing directly to a shared branch (like `main`) with no PR step is faster for a single developer working alone, but loses the review, discussion, and automated-check gate a PR provides — risky for anything beyond a personal project. A PR-based workflow adds a step (and some waiting for review/CI) before a change lands, in exchange for a second set of eyes and automated verification catching problems before they reach the shared branch that everyone else builds on.

## How It Works

### Common branching models — trunk-based vs. long-lived feature branches

```
Trunk-based:        short-lived branches (hours to a couple of days), merged frequently,
                     feature flags (a later topic) used to hide incomplete work rather than
                     keeping a branch open for weeks.
Long-lived feature branches: a branch stays open for the duration of a larger feature,
                     risking large, hard-to-review PRs and painful merge conflicts (next topic)
                     the longer it diverges from the branch it'll eventually merge into.
```

Trunk-based development with small, frequent PRs is generally preferred in modern teams specifically because it minimizes the divergence-related pain of both code review and merge conflicts.

### A PR as a review and verification gate

```
Opening a PR typically triggers:
  - CI running the test suite (Module 11's tests-in-ci.md) and any static analysis (a later topic)
  - Human code review (next topic)
  - Often a required minimum number of approvals before merging is even possible
```

The PR is the natural point where all these checks converge before a change reaches the shared branch — this is why "just push to main directly" bypasses far more than just "asking someone to look at it."

### Merge strategies — squash, merge commit, or rebase

```
Merge commit: preserves the full branch history, including every individual commit, plus an
              explicit merge commit tying them together.
Squash merge: collapses all of a branch's commits into ONE commit on the target branch —
              keeps main's history clean, at the cost of losing the branch's internal commit
              history (which may or may not have been valuable to preserve).
Rebase merge: replays the branch's commits on top of the target branch individually, without
              a merge commit — a linear history, but rewrites commit hashes.
```

Teams often standardize on one strategy (squash merge is common for keeping `main` readable) rather than mixing all three inconsistently.

### Keeping a PR small and reviewable

```
A 2,000-line PR touching twelve unrelated concerns is much harder to review meaningfully than
five separate 200-400 line PRs, each addressing one focused change — smaller PRs also reduce
the chance of merge conflicts accumulating while the PR waits for review.
```

## Application

Use short-lived branches and small, focused PRs as the default working style, especially in a trunk-based workflow. Rely on the PR as the natural gate for both automated checks and human review before a change reaches a shared branch. Pick one merge strategy per repository and apply it consistently.

## Common Mistakes

- Keeping a feature branch open for weeks, accumulating a large, hard-to-review diff and a growing risk of painful merge conflicts.
- Bundling many unrelated changes into one large PR, making it hard for a reviewer to actually evaluate any single change carefully.
- Bypassing the PR process for "quick fixes" pushed directly to a shared branch, skipping both review and automated checks.
- Mixing merge strategies inconsistently across a repository, producing a history that's neither fully linear nor fully preserved.

## Common Interview Questions

### Basic
- What is a pull request, and what purpose does it serve beyond just "proposing a merge"?
- What's the difference between squash merge, merge commit, and rebase merge?

### Intermediate
- Why is trunk-based development with short-lived branches generally preferred over long-lived feature branches?
- What checks typically run automatically when a PR is opened?

### Advanced
- How would you convince a team using long-lived feature branches to move toward smaller, more frequent PRs, given the pain points each approach has?
- How does PR size affect both review quality and merge-conflict risk, and how would you keep PRs appropriately small in practice?

### Follow-up Questions
- Does squash merging lose information that might matter later (e.g., for bisecting a bug)?
- Should every single commit require its own PR, or is that too granular?

### Code Prediction
A feature branch stays open for three weeks while `main` receives twenty unrelated commits during that time. What's the likely effect on the size and difficulty of the eventual merge, compared to if the same feature had been broken into several PRs merged every day or two?

## Practical Tasks

- Create a short-lived feature branch, make a small focused change, and open a PR describing it clearly.
- Compare the resulting `main` history after squash merge versus merge commit for the same branch.
- Break a hypothetical large, multi-concern change into a sequence of smaller, independently reviewable PRs.

## Readiness Criteria

Use short-lived branches and appropriately-sized PRs as a default working style, choose and apply merge strategies consistently, and explain why trunk-based development reduces both review and merge-conflict pain.

## References

### Other

- [GitHub Flow](https://docs.github.com/en/get-started/using-github/github-flow)
- [Trunk-based development](https://trunkbaseddevelopment.com/)
