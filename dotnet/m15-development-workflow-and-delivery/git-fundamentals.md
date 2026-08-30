# Git Fundamentals

## Definition

Git is a distributed version control system tracking changes to a codebase as a graph of **commits**, each a snapshot pointing to its parent(s). Understanding it as a graph — not a linear history, and not "a fancy folder backup" — is what makes branching, merging, and history rewriting make sense rather than feeling like arbitrary magic.

```bash
git init
git add file.cs
git commit -m "Add OrderService"
git log --oneline --graph  # visualizes the actual commit graph, not just a flat list
```

## Alternatives & Trade-offs

Older centralized version control (a single central server holding the only full history) required network access for most operations and made branching expensive. Git's distributed model — every clone has the full history — makes branching and local commits cheap and offline-capable, at the cost of a genuinely more complex mental model (a graph, not a line) that trips up developers who've only ever memorized a handful of commands without understanding what's actually happening underneath.

## How It Works

### The three areas — working directory, staging area, and repository

```bash
# Working directory: your actual files, as you're editing them
git status              # shows what's changed relative to the staging area and last commit

# Staging area (the "index"): what WILL be in the next commit
git add file.cs          # moves a change from working directory into the staging area

# Repository: the committed history
git commit -m "..."      # takes what's staged and creates a new commit from it
```

This three-area model explains a common point of confusion: `git add` doesn't commit anything — it stages a specific version of a file to be included in whatever the *next* commit turns out to be, which is exactly what makes partial commits (staging only some changes) possible.

### Commits as a graph, not a line

```
A -- B -- C  (main)
      \
       D -- E  (feature-branch)
```

Each commit points to its parent (or parents, for a merge commit) — a branch is just a movable label pointing at a specific commit, not a separate copy of the history. This is why creating a branch is instant and cheap: `git branch feature-branch` just creates a new label at the current commit, copying nothing.

### HEAD — where you currently are in the graph

```bash
git checkout feature-branch  # moves HEAD (and updates the working directory) to point at that branch's commit
```

`HEAD` is a pointer to "whatever commit your working directory currently reflects" — usually the tip of whichever branch you have checked out, but it can also point directly at a specific commit ("detached HEAD"), which is a common source of confusion when it happens accidentally.

### Remotes — synchronizing with another copy of the same repository

```bash
git remote add origin https://github.com/example/repo.git
git push origin main    # sends local commits to the remote
git pull origin main     # fetches and merges the remote's commits into the local branch
```

## Application

Understand every day-to-day Git operation (branching, merging, rebasing, resolving conflicts) as a specific operation on this commit graph, not as an isolated command to memorize — this is what makes unfamiliar situations (a confusing merge, a detached HEAD) debuggable rather than something to just `git add . && git commit` your way out of blindly.

## Common Mistakes

- Treating Git commands as a memorized checklist without understanding the underlying commit-graph model, making unfamiliar situations feel unrecoverable instead of just another graph state to reason about.
- Confusing the working directory, staging area, and repository — for example, expecting `git add` to have committed something.
- Not realizing a branch is just a movable pointer, leading to confusion about why deleting a branch doesn't delete its commits if another branch (or a detached reference) still points to them.
- Panicking at "detached HEAD" instead of recognizing it as simply being checked out at a specific commit rather than the tip of a named branch.

## Common Interview Questions

### Basic
- What are the three areas in Git's basic model (working directory, staging area, repository)?
- What is `HEAD`?

### Intermediate
- What does `git add` actually do, and why doesn't it commit anything?
- Why is creating a new branch in Git nearly instantaneous, regardless of repository size?

### Advanced
- How would you explain "detached HEAD" to someone who's confused by encountering it, in terms of the underlying commit graph?
- How does understanding commits as a graph (rather than a line) change how you'd approach recovering from a confusing merge or rebase?

### Follow-up Questions
- Does deleting a branch delete its commits?
- Can two branches point at the exact same commit?

### Code Prediction
Given `git branch feature-branch` followed immediately by `git log --oneline` on both `main` and `feature-branch`, what would the two logs show, assuming no commits have been made on either branch since the branch was created?

## Practical Tasks

- Stage only some changes from a modified file using `git add -p` and commit them separately from the rest.
- Deliberately enter and then recover from a detached HEAD state, explaining what happened at each step.
- Draw the commit graph for a small sequence of commits and branch operations, then verify it against `git log --graph`.

## Readiness Criteria

Explain Git's three-area model and commit-graph structure precisely, and reason about unfamiliar Git situations (detached HEAD, confusing history) in terms of that underlying model rather than memorized commands.

## References

### Other

- [Pro Git book (free, official)](https://git-scm.com/book/en/v2)
