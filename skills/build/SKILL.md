---
name: build
description: Use to implement issues created by /plan-work — either one specific ticket (/build #12) or the whole open backlog, one ticket per isolated subagent, every ticket built test-first via /test and closed only after passing /review. Triggers on "/build", "/build #<number>", "build the issues", "implement this ticket".
disable-model-invocation: true
---

# Build

Turns a GitHub issue into real code: a branch, an implementation, a PR,
and a closed issue. Runs either against one ticket or the whole open
backlog.

## Two modes

- **`/build #<number>`** — implement exactly that issue, in the current
  conversation.
- **`/build`** (no number) — implement every open, buildable issue in the
  repo, one at a time, each in its own fresh subagent so tickets never
  share context with each other.

If a number is given, skip straight to **Single-ticket build**. If not,
this is build-all — run the **context guard** first.

## Context guard (build-all only)

Build-all only runs from an effectively fresh session. Look at the
conversation so far: if it already contains substantial history unrelated
to this `/build` invocation (earlier unrelated tasks, long exploration,
other skills already run this session), **refuse** — tell the user the
session is too far along and ask them to invoke `/build` in a new
session/window instead. Don't try to estimate an exact token count; judge
by the shape of the conversation. A `/build` that's one of the first
messages in a session passes; a `/build` tacked onto a long-running one
does not, regardless of how much room might technically be left.

Only proceed to **Build-all** if this check passes.

## Discovering buildable issues

Buildable = open, and not a parent/epic. Skip any issue that itself has
sub-issues attached (those are `/setup`'s tracking issues, not
implementable tickets):

```
gh issue list --repo <owner>/<repo> --state open --json number,title,body

# for each issue, skip it if this is nonzero:
gh api repos/<owner>/<repo>/issues/<number>/sub_issues --jq 'length'
```

## Single-ticket build

1. Fetch the issue: `gh issue view <number> --repo <owner>/<repo> --json title,body,number`
2. Create a branch named after it, e.g. `issue-<number>-<kebab-slug-of-title>`
3. Implement what the issue describes by following the **`test`**
   skill's process: derive the behaviors to build from the issue's
   acceptance criteria/scope items (including edge cases and error
   paths), then work through them red → green → refactor, one at a
   time. Never write implementation code that doesn't have a preceding
   failing test behind it.
4. Commit with a message referencing the issue, e.g. `Fixes #<number>: <summary>`
5. Push the branch and open a PR:
   ```
   gh pr create --repo <owner>/<repo> --title "<summary>" \
     --body "Closes #<number>

   <what changed and why>"
   ```
6. **Gate 1 — TDD.** Close only if the `test` skill's "done" bar is met:
   every behavior has a test that was red before it was green, and the
   full suite passes (not just the new tests). If that bar isn't met,
   **do not close the issue** — leave the PR open (or don't open one at
   all if nothing is green yet), comment on the issue with what's
   failing and why, report that back, and stop here for this ticket.
7. **Gate 2 — review.** Once Gate 1 passes, run the `review` skill
   against the PR: two subagents (correctness, simplification/efficiency)
   review the diff in parallel, post their merged findings as a PR
   comment. If either subagent has a confirmed finding, **do not close
   the issue** — leave it open, the PR comment already states what needs
   fixing, report that back, and stop here for this ticket.
8. **Merge and close.** Only once both gates pass — an issue closed
   against an unmerged PR isn't actually done, the code still isn't on
   `main`:
   ```
   gh pr merge <pr-number> --repo <owner>/<repo> --merge --delete-branch
   gh issue close <number> --repo <owner>/<repo> \
     --comment "<what was implemented, link to the PR, confirm tests pass and review is clean>"
   ```
9. Report back the PR URL and whether it was merged + issue closed, or
   left open pending fixes (and which gate blocked it, if any).

## Build-all

1. Run the context guard above — stop here if it fails.
2. List buildable issues (above). If there are none, say so and stop.
3. Show the user the full list of issues about to be built and confirm
   before starting — this can run long and touches multiple
   branches/PRs, so don't launch it silently.
4. For each issue, **in order**, launch a fresh subagent (Agent/Task)
   whose entire job is the **Single-ticket build** process above for
   that one issue — hand it the repo and issue number, and the exact
   single-ticket steps to follow. Nothing about one ticket's
   implementation should carry into the next; the only thing that
   flows back to this orchestrating session is a short result (issue
   number, PR URL, pass/fail, one-line summary).
5. After each subagent finishes, record its result and move to the next
   issue. A ticket left open because it didn't clear Gate 1 (`test`) or
   Gate 2 (`review`) is an expected outcome, not a crash — note it and
   continue. Only stop the whole run early if the failure looks systemic
   (e.g. the repo doesn't build at all regardless of ticket).
6. When the run ends (all issues attempted, or stopped early), report a
   summary table: issue number → PR URL + closed, or left open with the
   reason (which gate blocked it), for every ticket attempted.

## Tone

Same as the rest of this skill set: confirm before anything irreversible
(build-all's issue list, in particular), don't pad reports, state results
plainly.
