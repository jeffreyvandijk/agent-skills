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

5. **Show the finished product.** Now that the epic just closed, launch
   the app so the user (and anyone else on the network) can see it live —
   skip this quietly if the project isn't a running app (a library, a
   one-shot script, etc.):
   - Check for a project-specific launch skill first; otherwise fall back
     to the built-in `run` skill's pattern for this project's type
     (server/CLI/TUI/etc.), including its background-launch and
     readiness-check approach.
   - Bind the listener to all interfaces — `0.0.0.0` or the framework's
     equivalent (`--host 0.0.0.0`, `HOST=0.0.0.0`, etc.) — instead of
     localhost-only, so other clients on the LAN can reach it too. This
     opens the port to everyone on the local network, which is fine for a
     demo on a trusted LAN but is a real exposure — mention it when you
     report the URL, and don't do this on a machine reachable beyond a
     trusted network.
   - Get this machine's LAN IP (`hostname -I | awk '{print $1}'` on
     Linux, `ipconfig getifaddr en0` on macOS) and report the reachable
     URL, e.g. `http://<lan-ip>:<port>`.
   - Leave it running in the background after verifying it's up — the
     point is for the user to view it, not to tear it down immediately.

6. **Report back** the parent issue URL and whether it closed, or how
   many sub-issues remain if it didn't. If it closed, include the live
   URL from step 5.

## Tone

Terse, no padding. Only acts when genuinely everything under the epic is
done — never a partial close, never a comment just to say "not yet."
