---
name: finish
description: Use whenever a sub-issue is closed — checks whether every sibling sub-issue under its parent epic is now closed, and if so closes the parent with a rollup summary linking the merged PRs. Also usable directly to check/close any epic on demand. Triggers whenever a sub-issue is closed (e.g. at the end of /build), or on "/finish".
---

# Finish

Closes the loop `/setup` opened. A sub-issue closing doesn't mean the
project is done — the parent epic (`/setup`'s one big issue) only closes
once every sub-issue `/plan-work` created under it is closed too.

## When invoked

- **Automatically, whenever a sub-issue is closed** — most often right
  after `/build` merges and closes a ticket. This skill can invoke itself
  on its own judgment for this case, no explicit `/finish` required.
- Directly via `/finish`, to check or close any epic on demand. If
  there's no clear parent/child context already in the conversation, ask
  for the repo and either the parent issue number or a child issue
  number to work from.

## Process

1. **Identify the parent.** If invoked right after a sub-issue closes,
   the parent is already known from that context (the epic `/plan-work`
   linked it to). Otherwise, find it: a sub-issue knows its parent via
   the same sub-issues relationship, queryable from the parent's side.

2. **Check every sibling's state:**
   ```
   gh api repos/<owner>/<repo>/issues/<parent_number>/sub_issues --jq '.[] | {number, title, state}'
   ```

3. **If any sibling is still open**, this isn't the last one — do
   nothing to the parent. Report how many are done (e.g. "2 of 3 closed")
   and stop. No comment, no partial close, no nagging.

4. **If every sibling is closed**, close the parent with a rollup
   summary: each sub-issue's title/number and a link to the PR that
   closed it.
   ```
   gh issue close <parent_number> --repo <owner>/<repo> \
     --comment "<rollup: one line per sub-issue with its PR link>"
   ```

5. **Report back** the parent issue URL and whether it closed, or how
   many sub-issues remain if it didn't.

## Tone

Terse, no padding. Only acts when genuinely everything under the epic is
done — never a partial close, never a comment just to say "not yet."
