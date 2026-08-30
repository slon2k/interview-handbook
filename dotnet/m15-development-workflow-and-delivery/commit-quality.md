# Commit Quality

## Definition

A good commit is a small, focused, self-contained change with a clear message explaining *why* it was made — not just *what* changed (the diff already shows that). Commit quality matters because the commit history is a genuine debugging and archaeology tool: `git bisect`, `git blame`, and code review all depend on commits being small and well-explained to be useful.

```bash
# Poor: vague, doesn't explain why, bundles unrelated changes
git commit -m "fixes"

# Good: specific, explains the reasoning, scoped to one change
git commit -m "Fix discount calculation rounding for orders under $1

Previously used integer division, which truncated fractional cents for
small orders. Switched to decimal division with explicit rounding."
```

## Alternatives & Trade-offs

Committing infrequently, in large batches, is less disciplined to maintain in the moment, but produces a history where `git blame` on any given line points to a huge, unrelated commit, and `git bisect` (binary-searching commit history to find which one introduced a bug) becomes far less precise, since one commit might contain a bug alongside dozens of unrelated changes. Committing small and often costs a bit more discipline per commit but keeps the history genuinely useful as a debugging tool, not just a backup log.

## How It Works

### Why message quality matters beyond documentation

```bash
git blame OrderService.cs -L 42,42
# shows WHICH commit last changed this line, and that commit's message is the only
# context available for WHY, unless the code itself is somehow self-explanatory
```

A vague message ("fix bug") gives a future developer (often the same person, months later) nothing beyond what the diff already shows — a good message answers the question the diff alone can't: *why* was this change necessary?

### `git bisect` and why small, focused commits matter

```bash
git bisect start
git bisect bad HEAD           # the bug is present now
git bisect good v1.2.0        # the bug wasn't present at this earlier known-good point
# git bisect automatically checks out commits between the two, narrowing down which ONE
# commit introduced the bug, via binary search
```

If each bisected commit bundles many unrelated changes, finding which commit introduced a bug doesn't tell you *what part* of that commit is actually responsible — small, focused commits make bisect's answer immediately actionable instead of just a starting point for further digging.

### Imperative mood and a conventional structure — a common, useful convention

```
Subject line: imperative mood, under ~50 characters ("Fix X", not "Fixed X" or "Fixes X")
(blank line)
Body: explains WHY, wrapped at a reasonable width, as much detail as the change warrants
```

```bash
git commit -m "Add retry policy to PaymentGatewayClient

The payment gateway occasionally returns transient 503s under load.
Adds exponential backoff retry (Module 13) to handle these without
surfacing an error to the end user for what's usually a momentary blip."
```

### Atomic commits — one logical change per commit

```
A commit that both fixes a bug AND reformats unrelated code makes the diff harder to review
and the eventual git blame/bisect less precise — reformatting belongs in its own commit,
separate from the actual behavioral fix.
```

## Application

Write commit messages that explain *why*, not just restate *what* the diff shows. Keep each commit to one logical, atomic change, separating unrelated concerns (a bug fix vs. a reformat) into distinct commits even within the same PR/branch.

## Common Mistakes

- Writing vague, uninformative commit messages ("fix", "wip", "updates") that provide no context beyond what the diff already shows.
- Bundling unrelated changes (a bug fix and an unrelated refactor) into one commit, making both `git blame` and `git bisect` less precise.
- Committing very large, infrequent batches of work instead of smaller, more frequent, individually-reviewable commits.
- Not explaining *why* a non-obvious change was made, leaving future readers (including the original author, later) to reconstruct the reasoning from scratch.

## Common Interview Questions

### Basic
- What makes a commit message good, beyond just describing the change?
- What is `git bisect`, and why does commit quality affect how useful it is?

### Intermediate
- Why should a bug fix and an unrelated code reformat generally be separate commits?
- What does "atomic commit" mean, and why does it matter for `git blame`?

### Advanced
- How would you use `git bisect` to find which commit introduced a regression, and how does poor commit granularity undermine that process?
- How would you explain, to a team with a habit of large infrequent commits, the concrete cost this creates for future debugging?

### Follow-up Questions
- Should every commit message have a body, or is a good subject line sometimes sufficient?
- Does commit message quality matter more or less once a branch is squash-merged?

### Code Prediction
A commit titled "various fixes" bundles a rounding bug fix, an unrelated whitespace reformat, and a new logging statement. Six months later, `git bisect` identifies this commit as introducing a subtle regression. How much closer does this get a developer to the actual root cause, compared to if these three changes had been three separate, well-described commits?

## Practical Tasks

- Write a well-structured commit message (subject + body explaining why) for a hypothetical bug fix.
- Split a bundled, multi-concern change into separate atomic commits.
- Use `git bisect` on a small sample repository to find which of several commits introduced a deliberately planted bug.

## Readiness Criteria

Write commit messages that explain reasoning, not just restate the diff, keep commits atomic and focused, and use `git bisect`/`git blame` effectively as debugging tools that depend on that discipline.

## References

### Other

- [How to Write a Git Commit Message (Chris Beams)](https://cbea.ms/git-commit/)
