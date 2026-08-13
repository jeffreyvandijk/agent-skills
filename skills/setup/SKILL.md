---
name: setup
description: Use after /start has produced a project brief, to turn it into a real GitHub repo — asks about repo naming convention and visibility, scaffolds a README and .gitignore from the brief, creates and pushes the repo, then files one issue containing the full brief. Triggers on "/setup", "create the repo for this", "set up the github repo".
---

# Setup

Take a project brief and turn it into an actual GitHub repo: named and
visible the way the user wants, seeded with a README that reflects the
brief, pushed, and tracked by one issue containing the full brief.

## When invoked

Requires a project brief to work from. If the conversation already has one
(typically produced by `/start`), use it. If not, ask the user to either
run `/start` first or give a short summary of the idea now — don't
fabricate a brief from nothing.

## Process

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
   - `README.md` generated from the brief — problem, target user, scope,
     constraints, success criteria.
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

   Don't split this into multiple issues — one issue is the point.

7. **Verify and report.** Confirm the repo with `git remote -v` and
   `gh repo view`, confirm the issue with `gh issue view`, then report
   both the repo URL and the issue URL back to the user.

## Tone

Match `/start`: short, direct questions via AskUserQuestion for discrete
choices (name, casing, visibility), plain conversation for anything
open-ended (e.g. clarifying the stack for `.gitignore`). Don't re-ask
anything already settled in the brief.
