# Merge Conflicts

## Definition

A merge conflict occurs when Git can't automatically reconcile changes to the same part of a file made independently on two branches — it needs a human to decide the correct final result. Understanding conflicts as "Git correctly detecting it can't safely guess" rather than "Git being broken" is the key mental shift that makes resolving them calmly straightforward rather than stressful.

```
<<<<<<< HEAD
    var total = subtotal * 0.9m; // this branch's version
=======
    var total = subtotal - discount; // the other branch's version
>>>>>>> feature/flat-discount
```

## Alternatives & Trade-offs

Avoiding conflicts entirely (by never having more than one person work on related code) isn't realistic for any real team — conflicts are an inherent, expected consequence of parallel development, not a sign something went wrong. The real trade-off is in *how often* and *how severely* they occur: short-lived branches and frequent integration (the branching topic's trunk-based approach) keep conflicts small and easy to resolve; long-lived, widely-diverging branches produce large, difficult conflicts precisely because more independent change has accumulated before reconciliation.

## How It Works

### What the conflict markers actually mean

```
<<<<<<< HEAD              <- start of YOUR branch's version (the one you're merging INTO)
    (your branch's code)
=======                    <- divider
    (the other branch's code)
>>>>>>> feature-branch     <- end of the OTHER branch's version, labeled with its name
```

Resolving a conflict means editing this section to reflect the *actual intended* final result — which might be one side, the other, or a combination — then removing the marker lines entirely and completing the merge.

```bash
git add ResolvedFile.cs   # marks the conflict as resolved for this file
git commit                # completes the merge, using a merge commit
```

### Understanding *why* a conflict occurred, not just mechanically resolving it

```
Both branches modified the SAME LINE of the discount calculation, but for different reasons —
one implemented a percentage discount, the other a flat-amount discount. The conflict isn't
just "text that doesn't match" — it's a signal that both branches made a real, potentially
incompatible business-logic decision about the same thing, which a human needs to actually
reconcile, not just mechanically pick one side of.
```

Resolving a conflict by blindly picking "my version" or "their version" without understanding *why* both sides changed that code risks silently discarding a real, intentional change from the other branch.

### Rebase vs. merge — conflicts can appear differently depending on the operation

```bash
git merge feature-branch    # a single conflict-resolution point, if any
git rebase main             # can require resolving conflicts commit-by-commit, as each of the
                              # branch's commits is individually replayed onto the new base
```

A rebase can surface the same overall set of conflicts as a merge would, but potentially spread across several individual resolution steps (one per replayed commit) rather than one combined resolution — worth knowing so it doesn't feel like rebase is "creating more conflicts" than merge would.

### Preventing severe conflicts before they happen

```bash
git pull origin main   # regularly incorporating main's latest changes into a feature branch
                          # keeps divergence small, so any eventual conflict stays small too
```

## Application

Treat a merge conflict as Git correctly flagging that a human decision is needed, not as a failure state. Resolve conflicts by understanding what each side was actually trying to accomplish, not by mechanically picking one side. Reduce the frequency and severity of conflicts by keeping branches short-lived and regularly pulling in the target branch's latest changes.

## Common Mistakes

- Resolving a conflict by blindly accepting one side without understanding why both sides changed the same code, risking silently discarding an intentional change.
- Treating a merge conflict as a sign something went wrong, rather than an expected, normal part of parallel development that Git is correctly surfacing.
- Letting a branch diverge for a long time without regularly pulling in the target branch's changes, accumulating a much larger and harder-to-resolve conflict later.
- Not testing the code again after resolving a conflict, assuming the mechanical text resolution alone guarantees the merged result actually still works correctly.

## Common Interview Questions

### Basic
- What causes a merge conflict?
- What do the `<<<<<<<`, `=======`, and `>>>>>>>` markers represent?

### Intermediate
- Why should a conflict be resolved by understanding both sides' intent, not just picking one?
- How does keeping branches short-lived reduce the frequency and severity of conflicts?

### Advanced
- How might resolving a merge conflict differ, mechanically, between a `git merge` and a `git rebase` of the same two branches?
- How would you verify that a conflict resolution is actually correct, beyond just resolving the textual markers?

### Follow-up Questions
- Does every conflict indicate that both sides made a real, meaningful change to the same logic, or can conflicts be purely mechanical (e.g., whitespace)?
- Should a merge conflict resolution always be re-tested before completing the merge?

### Code Prediction
Given the discount-calculation conflict example above, if a developer resolves it by simply keeping their own branch's version (`subtotal * 0.9m`) without looking at what the other branch intended, what business logic could be silently lost from the final merged result?

## Practical Tasks

- Deliberately create a merge conflict between two branches modifying the same line, resolve it thoughtfully, and verify the result is correct.
- Compare how the same underlying conflict is surfaced during a `git merge` versus a `git rebase`.
- Practice reducing conflict severity by regularly pulling a target branch's changes into a long-diverged feature branch before finally merging.

## Readiness Criteria

Resolve merge conflicts by understanding both sides' intent rather than mechanically picking one, explain conflict markers precisely, and reduce conflict frequency/severity through short-lived branches and frequent integration.

## References

### Other

- [Git documentation: Basic Branching and Merging](https://git-scm.com/book/en/v2/Git-Branching-Basic-Branching-and-Merging)
