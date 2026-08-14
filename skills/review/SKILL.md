---
name: review
description: Use before marking any issue, ticket, or piece of work as complete or closing it — runs a two-axis code review (correctness bugs + simplification/reuse/efficiency) via parallel subagents against the diff, posts findings as a PR comment, and blocks closing on any confirmed finding. Also use directly to review any diff/PR/branch. Triggers whenever an issue/ticket is about to be closed or marked done, or on "/review".
---

# Review

A two-axis code review of a diff, run as two independent subagents in
parallel, posted as a PR comment, and treated as a real gate — not a
report that gets ignored.

## When invoked

- **Automatically, whenever an issue/ticket is about to be marked
  complete or closed** — same trigger point as the `test` skill. This
  skill can invoke itself on its own judgment for this case, no explicit
  `/review` required.
- Directly via `/review`, to review any diff/PR/branch on demand.
- Inside `/build`'s implementation step, after the `test` skill's TDD
  gate passes, before deciding whether to close the issue.

## Process

1. **Identify the diff.** Normally the PR `/build` just opened for the
   issue being completed. If invoked directly, use whatever PR/branch/diff
   the user pointed at.

2. **Launch two subagents in parallel**, each given the diff and the
   issue's acceptance criteria as context, each with a narrow mandate:

   - **Correctness subagent** — hunts for logic errors, mishandled edge
     cases, crashes, and behavior that doesn't match what the issue
     actually asked for.
   - **Simplification/efficiency subagent** — hunts for duplication,
     unnecessary complexity, unused abstractions, places that should
     reuse an existing function/utility instead of a new one, and
     inefficient patterns.

   Both subagents must verify each candidate finding against the actual
   code before reporting it — no speculative "this might be an issue."
   An empty list is a valid, expected result when the diff is clean.

3. **Each subagent returns its findings** ranked most-severe first: file,
   line, one-sentence summary, and the concrete failure scenario or cost
   it causes. No padding, no restating the diff.

4. **Merge the two lists** into one review. Dedupe anything both
   subagents flagged, keep the severity ranking, and keep it terse.

5. **Post the merged review as a single PR comment:**
   ```
   gh pr comment <number> --repo <owner>/<repo> --body "<merged findings, or a short note that both axes came back clean>"
   ```
   Post the comment even when there's nothing to flag — a silent pass
   isn't the same as a documented one.

6. **Gate.** Any confirmed finding from either axis blocks the issue from
   closing. Leave it open, and make sure the PR comment makes clear
   what needs fixing before it can close. If both subagents come back
   clean, this axis clears — closing can proceed, still subject to the
   `test` skill's TDD gate passing too.

## Tone

Subagents verify before reporting — a finding that doesn't survive a
second look against the actual code doesn't get posted. No restating
what the diff already shows; state the problem and its consequence,
nothing else.
