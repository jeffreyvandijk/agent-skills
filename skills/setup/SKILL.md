---
name: setup
description: Use after /start has produced a project brief, to turn it into a real GitHub repo — asks about repo naming convention and visibility, scaffolds a short README and .gitignore, creates and pushes the repo, then files one issue containing the full brief and links the README to it. If the repo already exists (a later phase's brief, e.g. v2 after /start ran again), skips repo creation and just files a new tracking issue for that phase. Triggers on "/setup", "create the repo for this", "set up the github repo".
disable-model-invocation: true
---

# Setup

Take a project brief and turn it into an actual GitHub repo: named and
visible the way the user wants, seeded with a short README, pushed, and
tracked by one issue containing the full brief. The brief lives in one
place — the issue — not duplicated into the README.

## When invoked

Requires a project brief to work from. If the conversation already has one
(typically produced by `/start`), use it. If not, ask the user to either
run `/start` first or give a short summary of the idea now — don't
fabricate a brief from nothing.

**Check whether the repo already exists first.** Run `git rev-parse
--is-inside-work-tree` and `git remote -v` in the current directory. If
this is already a git repo with a GitHub `origin` remote, this is a later
phase of a project `/setup` already created (e.g. a v2 brief from running
`/start` again after a v1 shipped) — jump straight to **Process B**. Don't
re-ask about repo name, visibility, or local directory; that was already
decided and answering it again just wastes the user's time. Only run
**Process A** when there's no existing repo to work with.

## Process A — new project (no repo yet)

1. **Propose a repo name.** Derive a kebab-case name from the project idea
   in the brief. Confirm it with the user via AskUserQuestion — kebab-case
   is the recommended default, but let them override the name or the
   casing convention if they prefer something else.

2. **Ask visibility.** Public or private — don't default to either.
   Different projects call for different visibility; never assume public
   just because a past project was.

3. **Pick a local directory.** Default to `~/projects/<repo-name>`. State
   the path and confirm before creating anything. If that path already
   exists, stop and ask how to proceed — never overwrite a directory that
   might hold existing work.

4. **Scaffold the repo contents:**
   - `README.md` — **short**, not a restatement of the whole brief: the
     project name, a one-or-two-sentence description (from the brief's
     problem statement), and a bulleted scope summary (in-scope items
     only, no constraints/success-criteria/risks detail — that stays in
     the issue). End with a placeholder line, e.g. `Full project brief
     and progress: tracked in the repo's issues.` — this gets a real
     issue link added in step 6, once the issue exists.
   - `.gitignore` appropriate to the project's stack, inferred from the
     brief. Ask if the stack isn't clear from the brief.
   - No `LICENSE` file, and don't ask about licensing — out of scope for
     this skill.
   - Always created under the user's personal GitHub account — don't ask
     about orgs.

5. **Init, commit, create, push:**
   ```
   git init
   git add <scaffolded files>
   git commit -m "<clear, descriptive message>"
   gh repo create <name> --public|--private --source=. --remote=origin --push
   ```
   This mirrors how `agent-skills` itself was bootstrapped.

6. **File one big tracking issue.** Once the repo is pushed, create a
   single GitHub issue with `gh issue create`:
   - **Title** — the project name
   - **Body** — the full Project Brief from `/start`, unabridged: problem,
     target user, scope (with in-scope items as a markdown checklist),
     constraints, success criteria, and open risks

   Don't split this into multiple issues — one issue is the point. This
   is the single source of truth for the full brief; the README doesn't
   repeat it.

7. **Link the README to the issue.** Now that the issue URL is known,
   replace the placeholder line from step 4 with a real link using the
   **full issue URL**, not `#<issue-number>` — GitHub only auto-links the
   `#` shorthand inside issues/PRs/commits, not inside a rendered
   README.md, so the shorthand would show up as plain unlinked text (e.g.
   `Full project brief and progress:
   https://github.com/<owner>/<repo>/issues/<issue-number>`), commit, and
   push:
   ```
   git add README.md
   git commit -m "docs: link README to tracking issue #<issue-number>"
   git push
   ```

8. **Verify and report.** Confirm the repo with `git remote -v` and
   `gh repo view`, confirm the issue with `gh issue view`, then report
   both the repo URL and the issue URL back to the user.

## Process B — later phase (repo already exists)

The repo, README, and `.gitignore` already exist. All that's needed is
tracking the new phase's brief.

1. **File one big tracking issue for this phase**, same rules as Process A
   step 6:
   - **Title** — the project name plus the phase, e.g. `<project> v2` (ask
     the user what to call the phase only if it isn't obvious from the
     brief — don't guess something arbitrary)
   - **Body** — the full Project Brief from `/start` for this phase,
     unabridged

   Don't fold this into the original v1 issue and don't split it further —
   one issue per phase, same as one issue per project in Process A.

2. **Leave the README alone unless it needs updating.** If it already
   points at the repo's issues in general (not hardcoded to a single
   issue number), nothing to do. If it hardcodes a link to the v1 issue
   specifically, ask the user whether they want it updated to also
   reference the new phase issue, or left as historical — don't decide
   silently either way.

3. **Verify and report.** Confirm the issue with `gh issue view`, then
   report the issue URL back to the user.

## Tone

Match `/start`: short, direct questions via AskUserQuestion for discrete
choices (name, casing, visibility), plain conversation for anything
open-ended (e.g. clarifying the stack for `.gitignore`). Don't re-ask
anything already settled in the brief.
