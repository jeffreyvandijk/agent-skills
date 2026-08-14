---
name: plan-work
description: Use after /setup has filed the one big tracking issue, to break it into sub-issues — one per scope checklist item — linked via GitHub's native sub-issue relationship, with the original issue staying open as the parent epic. Triggers on "/plan-work", "break this issue apart", "split into sub-issues".
disable-model-invocation: true
---

# Plan Work

Break the one big tracking issue (from `/setup`) into individual,
actionable sub-issues — one per scope checklist item — linked to it as
real GitHub sub-issues, not just a task list in the description.

## When invoked

Requires the parent issue to work from. If it's not already clear from
the conversation, ask for the repo and issue number (or the issue URL).
Fetch it directly rather than relying on memory of what `/setup` wrote:

```
gh issue view <number> --repo <owner>/<repo> --json title,body,number
```

## Process

1. **Extract the scope checklist.** Pull the in-scope items from the
   parent issue's body (the checklist `/setup` wrote from the brief).
   Each item becomes one candidate sub-issue.

2. **Propose the breakdown before creating anything.** Show the user the
   proposed list — one line per sub-issue (title + one-line description
   derived from the checklist item). Let them edit, merge, split, or drop
   items. Do not call `gh issue create` until they've confirmed the list.
   This mirrors a scope checklist item 1:1 — don't invent extra tasks the
   brief didn't call for.

3. **Create each sub-issue:**
   ```
   gh issue create --repo <owner>/<repo> \
     --title "<item title>" \
     --body "<item description, with a back-reference to the parent issue, e.g. 'Part of #<parent_number>'>"
   ```

4. **Link each as a real sub-issue of the parent.** `gh` (as of 2.76) has
   no `--parent` flag for `issue create`, so link via the REST API
   directly. Sub-issues are addressed by numeric database ID, not issue
   number:
   ```
   # Get the parent's database id (once)
   gh api repos/<owner>/<repo>/issues/<parent_number> --jq .id

   # Get each new sub-issue's database id
   gh api repos/<owner>/<repo>/issues/<child_number> --jq .id

   # Attach it as a sub-issue of the parent — use -F (typed), not -f
   # (string): the API rejects sub_issue_id sent as a string.
   gh api repos/<owner>/<repo>/issues/<parent_number>/sub_issues \
     -X POST -F sub_issue_id=<child_database_id> \
     -H "Accept: application/vnd.github+json" \
     -H "X-GitHub-Api-Version: 2022-11-28"
   ```

5. **Leave the parent issue open.** It stays open as the epic — GitHub
   renders a live progress checklist of its sub-issues automatically.
   Don't close or edit its body to duplicate what the sub-issues now
   track.

6. **Verify and report.** `gh issue view <parent_number> --repo
   <owner>/<repo>` to confirm the sub-issues are attached, then report
   the parent issue URL and the list of new sub-issue URLs back to the
   user.

## Tone

Match `/start` and `/setup`: show the proposed breakdown plainly, ask for
confirmation in one round rather than piecemeal, and don't pad the
per-item descriptions beyond what the checklist item already said.
