---
name: build
description: Use to implement issues created by /plan-work — either one specific ticket (/build #12) or the whole open backlog, one ticket per isolated subagent. Triggers on "/build", "/build #<number>", "build the issues", "implement this ticket".
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
3. Implement what the issue describes, following the repo's existing
   patterns and conventions. Run whatever tests/build/lint the project
   has before considering it done — don't report success without
   verifying it.
4. Commit with a message referencing the issue, e.g. `Fixes #<number>: <summary>`
5. Push the branch and open a PR:
   ```
   gh pr create --repo <owner>/<repo> --title "<summary>" \
     --body "Closes #<number>

   <what changed and why>"
   ```
6. Close the issue with a summary comment:
   ```
   gh issue close <number> --repo <owner>/<repo> \
     --comment "<what was implemented, link to the PR>"
   ```
7. Report back the PR URL and confirm the issue is closed.

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
   issue. If a ticket fails, note it and continue to the next rather
   than aborting the whole run — unless the failure looks systemic (e.g.
   the repo doesn't build at all), in which case stop and report
   immediately.
6. When the run ends (all issues attempted, or stopped early), report a
   summary table: issue number → PR URL or failure reason, for every
   ticket attempted.

## Tone

Same as the rest of this skill set: confirm before anything irreversible
(build-all's issue list, in particular), don't pad reports, state results
plainly.
